---
layout: post
title: "下载 YouTube 视频片段"
subtitle: "使用 youtube-dl 配合 FFmpeg 修正下载中可能出现的错误"
date: 2020-01-03 11:29:00
categories: note
tags: [youtube, youtube-dl, ffmpeg]
---

# 下载YouTube视频片段

### 需求

下载YouTube长视频的前30分钟内容。由于视频时长超过3个小时，如果采用y2mate等网站缓存、合并音视频再下载整个视频成功率会很低，而且造成巨大的流量浪费。

### 工具

系统的环境：Python 3.x

软件/工具：youtube-dl、FFmpeg

### 步骤

首先用youtube-dl获得视频的各个质量（格式）列表。

```bash
[root@vps]# youtube-dl -F https://www.youtube.com/watch?v=hx-iVLcZO8U
[youtube] hx-iVLcZO8U: Downloading webpage
[youtube] hx-iVLcZO8U: Downloading video info webpage
WARNING: Unable to extract video title
[info] Available formats for hx-iVLcZO8U:
format code  extension  resolution note
249          webm       audio only DASH audio   73k , opus @ 50k, 67.02MiB
250          webm       audio only DASH audio   90k , opus @ 70k, 87.16MiB
140          m4a        audio only DASH audio  142k , m4a_dash container, mp4a.40.2@128k, 191.28MiB
251          webm       audio only DASH audio  158k , opus @160k, 165.29MiB
278          webm       256x144    144p  107k , webm container, vp9, 30fps, video only, 119.96MiB
160          mp4        256x144    144p  112k , avc1.4d400c, 30fps, video only, 103.25MiB
242          webm       426x240    240p  237k , vp9, 30fps, video only, 246.33MiB
133          mp4        426x240    240p  246k , avc1.4d4015, 30fps, video only, 204.72MiB
243          webm       640x360    360p  498k , vp9, 30fps, video only, 456.21MiB
134          mp4        640x360    360p  634k , avc1.4d401e, 30fps, video only, 521.65MiB
244          webm       854x480    480p  795k , vp9, 30fps, video only, 823.99MiB
135          mp4        854x480    480p 1352k , avc1.4d401f, 30fps, video only, 1011.91MiB
247          webm       1280x720   720p 1564k , vp9, 30fps, video only, 1.62GiB
136          mp4        1280x720   720p 2726k , avc1.4d401f, 30fps, video only, 1.86GiB
248          webm       1920x1080  1080p 2728k , vp9, 30fps, video only, 2.86GiB
137          mp4        1920x1080  1080p 5076k , avc1.640028, 30fps, video only, 3.21GiB
43           webm       640x360    medium , vp8.0, vorbis@128k, 1.06GiB
18           mp4        640x360    medium  565k , avc1.42001E, mp4a.40.2@ 96k (44100Hz), 835.24MiB
22           mp4        1280x720   hd720 1417k , avc1.64001F, mp4a.40.2@192k (44100Hz) (best)
```

注意，YouTube对于720p以上的视频存储方式是音视频分离，即1080p质量的视频是由编号为137的视频（无声）文件和编号为140的音频文件合成的。因此需要将这两个文件都下载下来。

```bash
[root@la default] youtube-dl -f 137 https://www.youtube.com/watch?v=hx-iVLcZO8U 
[youtube] hx-iVLcZO8U: Downloading webpage
[youtube] hx-iVLcZO8U: Downloading video info webpage
WARNING: Unable to extract video title
[youtube] hx-iVLcZO8U: Downloading js player vfl22ubNH
[youtube] hx-iVLcZO8U: Downloading js player vfl22ubNH
WARNING: Requested formats are incompatible for merge and will be merged into mkv.
[download] Destination: _-hx-iVLcZO8U.f137.mp4

[download]   0.0% of 4.32GiB at Unknown speed ETA Unknown ETA
[download]   0.0% of 4.32GiB at Unknown speed ETA Unknown ETA
[download]   0.0% of 4.32GiB at Unknown speed ETA Unknown ETA
[download]   0.0% of 4.32GiB at Unknown speed ETA Unknown ETA
[download]   0.0% of 4.32GiB at Unknown speed ETA Unknown ETA
[download]   0.0% of 4.32GiB at Unknown speed ETA Unknown ETA
[download]   0.0% of 4.32GiB at 62.78MiB/s ETA 01:10
[download]   0.0% of 4.32GiB at 76.68MiB/s ETA 00:57
...
[download]  15.6% of 4.32GiB at 101.55MiB/s ETA 00:36^C
ERROR: Interrupted by user
```

大致的计算可求得30分钟的内容占全视频的15%左右，因此在视频下载完成15%时手动打断。

```bash
[root@vps] youtube-dl -f 140 https://www.youtube.com/watch?v=hx-iVLcZO8U
[youtube] hx-iVLcZO8U: Downloading webpage
[youtube] hx-iVLcZO8U: Downloading video info webpage
WARNING: Unable to extract video title
[download] Destination: _-hx-iVLcZO8U.m4a
...
[download] 100% of 191.28MiB in 00:04
[ffmpeg] Correcting container in "_-hx-iVLcZO8U.m4a"
```

之后使用FFmpeg对音视频文件进行精确的时间裁剪（`-accurate_seek`参数可以实现精确到帧的裁剪，避免音视频不同步的问题），保存前30分钟的内容。

```bash
> ffmpeg -ss 0:0:0 -t 0:30:00 -accurate_seek -i .\_-hx-iVLcZO8U.f137.mp4.part -codec copy mix_v.mp4
> ffmpeg -ss 0:0:0 -t 0:30:00 -accurate_seek -i .\_-hx-iVLcZO8U.m4a -codec copy mix_a.m4a
```

合并音视频文件。

```bash
> ffmpeg -i .\mix_v.mp4 -i .\mix_a.m4a -c:v copy mix.mp4
```

这时候打开`mix.mp4`文件，即可看到精确剪裁后合并的文件。



