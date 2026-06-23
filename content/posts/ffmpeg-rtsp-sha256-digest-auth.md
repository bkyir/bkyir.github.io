+++
date = 2026-06-23T10:00:00+08:00
draft = false
title = 'FFmpeg RTSP Digest 认证添加 SHA-256 算法支持'
tags = ['FFmpeg', 'C', 'RTSP', '认证', '网络']
categories = ['音视频']
summary = 'FFmpeg 的 RTSP Digest 认证原本只支持 MD5，部分新版安防设备（如海康威视）要求 SHA-256 导致认证失败。在 httpauth.c 中增加 SHA-256 分支，兼容 RFC 7616，且完全向后兼容。'
+++

## 一、背景

FFmpeg 的 RTSP 协议复用了 HTTP 的认证模块（`libavformat/httpauth.c`），支持 Basic 和 Digest 两种认证方式。

在 Digest 认证中，FFmpeg 原本只支持 **MD5** 和 **MD5-sess** 两种摘要算法。然而根据 RFC 7616（HTTP Digest Access Authentication），Digest 认证还支持 **SHA-256** 等更安全的哈希算法。部分较新的 RTSP 服务端（如海康威视等安防设备）会优先要求使用 SHA-256 进行认证，此时 FFmpeg 会因算法不支持而认证失败。

## 二、Digest 认证原理

Digest 认证的核心是计算一个 `response` 值，过程如下：

**第一步：计算 HA1（凭证摘要）**

```
HA1 = H(username : realm : password)
```

其中 `H()` 为哈希函数（MD5 或 SHA-256），`realm` 为服务端返回的认证域。

**第二步：计算 HA2（请求摘要）**

```
HA2 = H(method : uri)
```

**第三步：计算 response（最终响应）**

- 无 qop 时：
  ```
  response = H(HA1 : nonce : HA2)
  ```
- 有 qop（如 `auth`）时：
  ```
  response = H(HA1 : nonce : nc : cnonce : qop : HA2)
  ```

其中 `nonce` 为服务端随机数，`nc` 为请求计数，`cnonce` 为客户端随机数。

MD5 输出 16 字节（32 个十六进制字符），SHA-256 输出 32 字节（64 个十六进制字符），这是两者在实现上的主要区别。

## 三、代码修改

修改文件：`libavformat/httpauth.c`

### 1. 添加头文件

```c
#include "libavutil/sha.h"
```

### 2. 扩大缓冲区

SHA-256 输出为 32 字节，十六进制表示为 64 字符，因此需要将哈希缓冲区从 33 扩展到 65：

```c
// 修改前（仅支持 MD5）
char A1hash[33], A2hash[33], response[33];
uint8_t hash[16];

// 修改后（兼容 MD5 和 SHA-256）
int is_sha256 = (!strcmp(digest->algorithm, "SHA-256"));
char A1hash[65], A2hash[65], response[65];
uint8_t hash[32];
```

### 3. 添加 SHA-256 计算分支

根据 `algorithm` 字段判断是否使用 SHA-256，分别走不同的计算路径：

```c
if (is_sha256) {
    struct AVSHA *sha = av_sha_alloc();
    if (!sha)
        return NULL;

    // HA1 = SHA-256(username : realm : password)
    av_sha_init(sha, 256);
    av_sha_update(sha, username, strlen(username));
    av_sha_update(sha, ":", 1);
    av_sha_update(sha, state->realm, strlen(state->realm));
    av_sha_update(sha, ":", 1);
    av_sha_update(sha, password, strlen(password));
    av_sha_final(sha, hash);
    ff_data_to_hex(A1hash, hash, 32, 1);

    // HA2 = SHA-256(method : uri)
    av_sha_init(sha, 256);
    av_sha_update(sha, method, strlen(method));
    av_sha_update(sha, ":", 1);
    av_sha_update(sha, uri, strlen(uri));
    av_sha_final(sha, hash);
    ff_data_to_hex(A2hash, hash, 32, 1);

    // response = SHA-256(HA1 : nonce : [nc : cnonce : qop] : HA2)
    av_sha_init(sha, 256);
    av_sha_update(sha, A1hash, 64);
    av_sha_update(sha, ":", 1);
    av_sha_update(sha, digest->nonce, strlen(digest->nonce));
    if (!strcmp(digest->qop, "auth") || !strcmp(digest->qop, "auth-int")) {
        av_sha_update(sha, ":", 1);
        av_sha_update(sha, nc, strlen(nc));
        av_sha_update(sha, ":", 1);
        av_sha_update(sha, cnonce, strlen(cnonce));
        av_sha_update(sha, ":", 1);
        av_sha_update(sha, digest->qop, strlen(digest->qop));
    }
    av_sha_update(sha, ":", 1);
    av_sha_update(sha, A2hash, 64);
    av_sha_final(sha, hash);
    ff_data_to_hex(response, hash, 32, 1);

    av_free(sha);
} else {
    // 原有 MD5 / MD5-sess 逻辑保持不变
    ...
}
```

### 4. 保持向后兼容

MD5 和 MD5-sess 的原有逻辑被完整保留在 `else` 分支中，不影响已有功能。

## 四、注意事项

1. **头文件无需修改** — `httpauth.h` 中的 `algorithm[10]` 字段足以容纳 `"SHA-256"`（7字符 + null = 8字节）。但如果将来需要支持 `"SHA-256-sess"`（12字符），则需要将该字段扩大。

2. **不支持 SHA-256-sess** — 当前实现仅支持无 session 的 SHA-256，session 模式（`SHA-256-sess`）需要额外的 nonce/cnonce 计算，暂未实现。

3. **仅需替换 avformat 相关库** — 该修改涉及 `libavformat/httpauth.c`，编译后只需替换 `avformat-61.dll`（动态库）或 `libavformat.a`（静态库），头文件无需更新。

## 五、验证

使用海康威视摄像头进行 RTSP 拉流测试：

```bash
ffmpeg -i "rtsp://admin:password@172.16.25.130:554/Streaming/Channels/101" \
       -t 3 -c copy -y test.mp4
```

测试结果：

```
Input #0, rtsp, from 'rtsp://...':
  Stream #0:0: Video: hevc (Main), 1920x1080, 30 fps
  Stream #0:1: Audio: aac (Main), 16000 Hz, mono
frame=   89 fps= 41 Lsize=    1064KiB time=00:00:03.03
```

拉流成功，SHA-256 认证工作正常。

## 六、总结

本次修改在 FFmpeg 的 Digest 认证模块中添加了 SHA-256 算法支持，使其兼容 RFC 7616 标准，解决了部分新版 RTSP 服务端要求 SHA-256 认证导致连接失败的问题。改动集中在一个文件，风险可控，且完全向后兼容。
