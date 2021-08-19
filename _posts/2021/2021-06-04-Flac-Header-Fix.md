---
layout: post
title: 手动修复损坏的 FLAC 文件头
date: 2021-06-04 20:00:00
categories: [Operation]
author: MowChan
tags: [flac, 文件头]
nightly: true
---


最近在使用 MP3Tag 软件给音乐添加标签时会出现文件损坏的情况，使用环境如下：

> 添加或修改标签时音乐文件被 Windows Media Player 占用
>
> MP3Tag 版本：3.06a
>
> 系统环境：Windows 10 LTSC 2019 x64
>
> 损坏文件类型：flac

![](https://cdn.jsdelivr.net/gh/mowchann/blog-static@master/image/column/post/2021/20210705-101844.webp)

之后我尝试用 FFmpeg、其它主流的格式转换软件修复均告失败。用 WinHex 查看损坏的音乐文件，发现文件开始都是`00`填充。合理怀疑是 MP3Tag 在修改或添加标签时把文件头丢了。

![](https://cdn.jsdelivr.net/gh/mowchann/blog-static@master/image/column/post/2021/20210705-105248.webp)

动手修复，最简单的方法是将正常的FLAC文件文件头复制到损坏文件头位置。因为对FLAC文件头格式不了解，如果手动将FLAC文件头含有的标签信息剔除掉仍然无法修复文件，所以只能是带着其它音乐的标签信息一起粘贴过来。

![](https://cdn.jsdelivr.net/gh/mowchann/blog-static@master/image/column/post/2021/20210604-112828.webp)

保存后，可以看到资源管理器已经读出标签信息。

![](https://cdn.jsdelivr.net/gh/mowchann/blog-static@master/image/column/post/2021/20210604-112758.webp)

需要注意的是，要完整地将标签信息复制过来，如果缺少某个字节可能会使得标签文本解析错误，导致资源管理器崩溃。

![](https://cdn.jsdelivr.net/gh/mowchann/blog-static@master/image/column/post/2021/20210604-115621.webp)

最后，再用 MP3Tag 修改为正确的标签，修复完成！
