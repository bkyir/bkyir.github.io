+++
date = 2026-06-22T09:40:00+08:00
draft = false
title = '记一次 av_seek_frame 阻塞问题排查：HLS 点播场景下的编码切换坑'
tags = ['FFmpeg', 'C++', '音视频', 'HLS']
categories = ['音视频']
summary = 'HLS 点播中，服务端录制的 ts 文件混入了不同编码格式，导致 av_seek_frame 永久卡死。用 interrupt_callback 强制打断 seek 解决。'
+++

## 背景

最近在做 HLS 点播相关的工作，播放端用 FFmpeg 解复用。遇到一个诡异的问题：**某些录制文件在 seek（拖动进度条）时会直接卡死，整个播放线程 hang 住不动**。

排查下来，根因是服务端录制逻辑的一个历史遗留问题。这篇记录一下定位过程和解决方案。

## 现象

播放 HLS 点播流时，`av_seek_frame` 调用后**永久不返回**，播放线程彻底卡死。没有报错、没有异常、没有日志，就是一直阻塞在那儿。如果不做任何处理，线程会一直挂在那里，直到超时（如果有超时的话）或者进程被杀。

典型的卡死调用栈会停在 `av_seek_frame` 内部的 demuxer 读取循环里。

## 根因

和服务端同事一起查了录制文件，发现一个关键问题：**服务端录制视频时，并没有对编码参数的切换做处理**。

具体来说：

- 正常的 HLS 录制：一次直播会话对应一组固定编码参数（编码格式、分辨率、帧率等），录制出来的 ts 文件编码格式从头到尾一致。
- 出问题的录制：直播过程中**源流切换了编码格式**（比如推流端重连、转码链路变化，把 H.264 换成了 H.265，或反之），**但服务端没有在编码格式切换时重置录制上下文**，直接把**不同编码格式**的数据拼到了**同一个 ts 文件**里。

这就导致一个 ts 文件内部存在**编码格式（codec）不一致**的情况——比如前半段是 H.264，后半段突然变成 H.265。

> 注意：这里说的「编码格式」指的是 codec 级别（H.264 / H.265 等）。单纯的分辨率切换（同样是 H.264，只是 1080p → 720p）、码率变化、帧率变化**不会**导致这个问题，因为 codec 没变，SPS/PPS 更新后解码器能正常重建上下文。真正的杀手是 **codec 本身发生了切换**。

`av_seek_frame` 在 seek 时，需要定位到目标位置并读取附近的数据来重建解码上下文。当它 seek 到 codec 切换边界附近时，demuxer/解析逻辑会陷入混乱：它按 A 编码格式打开的上下文，读到的却是 B 编码格式的数据，参数对不上，于是不停往后读、试图找到一个「合理」的位置，结果陷入死循环般地一直读不到头——**最终表现为永久阻塞**。

用 ffprobe / ffplay 打开这些出问题的文件，也能看到一些异常警告，但 FFmpeg 的命令行工具有自己的容错和超时逻辑，不会像我们直接调用 API 那样裸奔地卡死。

## 解决方案

最终的解决方案是：**用 `interrupt_callback` 给 seek 操作加上强制打断机制**。

### 思路

`AVFormatContext` 提供了一个 `interrupt_callback` 回调。FFmpeg 在执行 IO 密集型操作（包括 read、seek 等）时，会周期性地调用这个回调，**如果回调返回 1，FFmpeg 就会立即中断当前操作并返回**。

这正是我们需要的：

- 进入 `av_seek_frame` 前记录开始时间
- 设置一个超时阈值（比如 3 秒）
- 在 `interrupt_callback` 里判断：如果当前时间已经超过阈值，就返回 1 强制打断

这样即使 seek 卡死，最多等 3 秒就会被打断，播放线程不会永久 hang 住。被打断后根据业务需要处理——可以回退到上一个可用位置、直接顺序播放、或者提示用户 seek 失败。

### 关键代码骨架

```cpp
// 全局/上下文：记录 seek 开始时间和超时阈值
struct SeekTimeoutCtx {
    int64_t start_ms;   // seek 开始时间戳
    int64_t timeout_ms; // 允许的最大耗时
};

// interrupt_callback：超时则返回 1，FFmpeg 立即中断当前阻塞操作
static int seek_interrupt_cb(void* opaque) {
    auto* ctx = static_cast<SeekTimeoutCtx*>(opaque);
    int64_t now = current_ms();  // 取系统时间
    return (now - ctx->start_ms) >= ctx->timeout_ms ? 1 : 0;
}

// 使用
SeekTimeoutCtx timeout_ctx{ .start_ms = current_ms(), .timeout_ms = 3000 };
fmt_ctx->interrupt_callback.cb     = seek_interrupt_cb;
fmt_ctx->interrupt_callback.opaque = &timeout_ctx;

int ret = av_seek_frame(fmt_ctx, stream_index, target_ts, flags);
// ret < 0 时，可能就是被打断了，走 seek 失败的处理逻辑
```

核心就是 `interrupt_callback` 这一层兜底。它不是去「修复」那个混乱的 ts 文件，而是在 seek 异常时及时止损，把控制权还给播放线程。

## 一些思考

### 为什么不直接修服务端录制？

理想情况下，**根治的办法当然是修服务端录制逻辑**：在检测到编码格式（codec）切换时，重置录制上下文，要么切到新的 ts 文件，要么至少保证单个 ts 内编码格式一致。

但现实中：

1. 录制端是另一个团队维护的服务，改动排期不可控
2. 线上已经存在大量历史问题录制文件，播放端不兼容就等于这些文件废了
3. 播放端的健壮性本来就该做——你没法保证所有源都「规范」

所以播放端加 `interrupt_callback` 兜底，是性价比最高、见效最快的方案。服务端那边作为一个长期优化项跟进。

### 为什么 FFmpeg 不自己处理？

FFmpeg 的 API 设计哲学是**把控制权交给调用方**。它提供了 `interrupt_callback` 这个机制，就是默认你会自己设置超时策略。如果你不设，它就认为你愿意无限等下去——这是「契约」，不是 bug。

所以用 FFmpeg API 做 IO 相关操作时，**永远记得设 `interrupt_callback`**，这是基本的健壮性要求。不只是 seek，`avformat_open_input`、`av_read_frame` 等阻塞调用同理。

### 教训

- **不要假设上游数据是规范的**：播放端/消费端永远要做最坏的打算，尤其是音视频这种数据流复杂的场景。
- **遇到「永久阻塞」类问题，第一时间找有没有 callback / timeout 机制**：FFmpeg 有 `interrupt_callback`，很多阻塞型 API 也有类似的兜底设计。
- **根因和缓解要分开看**：根因（服务端录制）要推动修，缓解（播放端打断）要马上做，两条线并行。

## 参考

- [FFmpeg 文档：AVFormatContext.interrupt_callback](https://ffmpeg.org/doxygen/trunk/structAVFormatContext.html#a5b55ac669ae280c9ef1be9fea6c78a5e)
- [FFmpeg 文档：av_seek_frame](https://ffmpeg.org/doxygen/trunk/group__lavf__seeking.html#gaa72f3cd61ad30c46dd52f5ea5f0a7b7e)
