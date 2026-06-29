+++
date = 2026-06-29T10:00:00+08:00
draft = false
title = 'Windows certutil 工具常用命令'
tags = ['Windows', 'certutil', '命令行']
categories = ['Windows']
summary = '整理 Windows 自带的 CertUtil 工具常用命令：查看文件十六进制、计算文件哈希、Base64 编码与解码。'
+++

# windows 相关

## certutil

`CertUtil` 是 **Windows 自带的证书管理工具**（Certificate Utility），随 Windows 一起安装，无需额外下载。虽然它最初是为管理证书设计的，但由于功能比较丰富，也经常被开发者当作文件分析工具使用。

* 查看文件十六进制

```shell
certutil -dump test.txt
```

* 计算文件哈希

```shell
certutil -hashfile test.txt MD5
```

支持其它算法

```shell
certutil -hashfile test.txt SHA1
certutil -hashfile test.txt SHA256
certutil -hashfile test.txt SHA512
```

* Base64编码/解码

编码

```shell
certutil -encode input.bin output.txt
```

解码

```shell
certutil -decode input.txt output.bin
```

