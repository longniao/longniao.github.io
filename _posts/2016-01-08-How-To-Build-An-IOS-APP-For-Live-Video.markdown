---
layout: post
title: IOS手机直播Demo技术简介
excerpt: IOS手机直播Demo技术简介及一些技术细节要点
---

# 原文：[IOS手机直播Demo技术简介](https://www.zybuluo.com/qvbicfhdx/note/126161)

# 目录：

*   [IOS手机直播Demo技术简介](#ios手机直播demo技术简介)
    *   *   [1.测试服务器的搭建](#1测试服务器的搭建)
            *   *   *   [1.下载nginx](#1下载nginx)
                    *   [2.下载nginx-rtmp-module](#2下载nginx-rtmp-module)
                    *   [3.编译安装nginx:](#3编译安装nginx)
                    *   [4.配置nginx](#4配置nginx)
                    *   [5.启动nginx ：](#5启动nginx)
        *   [2.编译ffmpeg库](#2编译ffmpeg库)
            *   *   *   [1.下载并编译libx264](#1下载并编译libx264)
                    *   [2.下载并编译libfdk-aac](#2下载并编译libfdk-aac)
                    *   [3.下载并编译librtmp](#3下载并编译librtmp)
                    *   [4.下载并编译ffmpeg](#4下载并编译ffmpeg)
        *   [3.采集音视频](#3采集音视频)
            *   *   *   [1.ios音频采集：](#1ios音频采集)
                    *   [2.ios视频录制](#2ios视频录制)
        *   [4.编码（AAC,H264）](#4编码aach264)
            *   *   *   [AAC编码](#aac编码)
                    *   [H264编码](#h264编码)
        *   [5.RTMP推送](#5rtmp推送)
