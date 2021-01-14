---
layout: post
title: MATLAB 使用与配置问题汇总
date: 2021-01-06 23:00:00
categories: [MatLab]
author: MowChan
latex: false
incomplete: true
tags: [MatLab, 混合字体]
---



#### 代码编辑区使用其它字体（或中文乱码）

打开设置面板 HOME->Preferences->MATLAB->Fonts，设置 Desktop code font 为含中文的字体即可。如果使用混合字体（含中文的编程字体），需要注意在 Windows 安装该字体时选择“为所有用户安装”，否则是无法在 MATLAB 中找到该字体的。

https://www.jianshu.com/p/ffb9eddb8f21



#### MATLAB 精简便携版本

MATLAB 在只安装核心功能后是可以直接将程序目录打包作为便携版使用，甚至打包成镜像文件，直接启动`bin\matlab.exe`就可以正常运行。

Win8/Win10系统下 MATLAB 的用户配置文件存储在 `C:\Users\用户名\AppData\Roaming\MathWorks\MATLAB\` 。

