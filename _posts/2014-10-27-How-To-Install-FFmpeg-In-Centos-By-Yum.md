---
layout:     post
title:      使用yum在centos下安装最新版的ffmpeg
subtitle:   使用yum在centos下安装最新版的ffmpeg。
date:       2014-10-27
author:     Longniao
header-img: img/post-ffmpeg.jpg
catalog: true
tags:
    - Yum
    - Centos
    - Ffmpeg
---

首先安装编译环境,如果系统有就不用安装了。

yum install -y automake autoconf libtool gcc gcc-c++ 

yum install make

yum install svn

如果还需要其他的软件就按照下面的方式安装。

yum search **

yum install **

到此,我们就可以通过svn命令获取最新的ffmpeg了

svn checkout svn://svn.mplayerhq.hu/ffmpeg/trunk ffmpeg

你会发现在你所在的目录,自动出现一个ffmpeg的目录,就是你下载的源代码。

切换到ffmpeg目录下,执行以下命令。

./configure --prefix=/usr 

make 

make install

安装完毕,运行ffmpeg命令试一下,把mov文件转换为mp4文件,保证相应目录下有qq.mov这个文件,注意大小写。

ffmpeg -i /usr/local/movi/qq.mov -r 25 -b 3200k -vcodec mpeg4 -ab 128k -ac 2 -ar 44100  /usr/local/movi/kk.mp4
