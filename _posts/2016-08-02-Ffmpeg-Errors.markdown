---
layout: post
title: Ffmpeg安装及错误处理
excerpt: FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。
---
FFmpeg是一个开源免费跨平台的视频和音频流方案，属于自由软件，采用LGPL或GPL许可证（依据你选择的组件）。它提供了录制、转换以及流化音视频的完整解决方案。它包含了非常先进的音频/视频编解码库libavcodec，为了保证高可移植性和编解码质量，libavcodec里很多codec都是从头开发的。

系统准备
---

`首先需要安装第三方rpmforce库`

#### 1. 安装编码和依赖库文件

```
yum -y install lame lame-devel libogg libogg-devel dirac dirac-devel libvorbis libvorbis-devel SDL SDL-devel gsm gsm-devel libvpx libvpx-devel libvpxlame-devel xvidcore xvidcore-devel faac faac-devel opencore-amr opencore-amr-devel yasm faad2 a52dec  
```
#### 2. 安装libtheora软件包

```
tar jxf libtheora-1.1.1.tar.bz2  
cd libtheora-1.1.1 
./configure --prefix=/usr/local/ffmpeg --with-ogg=/usr --with-vorbis=/usr --with-sdl-prefix=/usr
make
make install

```

#### 3. 安装x264 yum 中x264 版本有点旧，需要更高版本的x264

```
wget ftp://ftp.videolan.org/pub/x264/snapshots/last_x264.tar.bz2  
tar jxf last_x264.tar.bz2   
cd x264-snapshot-20110822-2245  
./configure --prefix=/usr --enable-shared   
make  
make install
```
#### 4. 最后安装ffmpeg，此处安装 ffmpeg 0.8.2

```
wget http://ffmpeg.org/releases/ffmpeg-0.8.2.tar.gz  
tar zxf ffmpeg-0.8.2.tar.gz  
cd ffmpeg-0.8.2   
./configure --prefix=/usr --libdir=/usr/lib64 --shlibdir=/usr/lib64 --mandir=/usr/share/man --incdir=/usr/include --disable-avisynth --disable-indev=v4l --disable-indev=v4l2 --extra-cflags='-O2 -g -pipe -m64 -fPIC' --enable-avfilter --enable-libdirac --enable-libfaac --enable-libgsm --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libx264 --enable-gpl --enable-nonfree --enable-postproc --enable-pthreads --enable-shared --enable-swscale --enable-vdpau --enable-version3 --enable-x11grab  
make install
```

#### 5. 测试ffmpeg

```
ffmpeg -i 1.avi -vframes 1 -y -f gif -pix_fmt rgb24 2.gif # 视频截图 gif  
ffmpeg -i 1.avi -vframes 1 -y -f image2 -t 0.001 -s 600x480 2.jpg # 视频截图 jpg  

```

YUM安装
---

```
yum install ffmpeg
```

错误处理
---

- ffmpeg: error while loading shared libraries: libx264.so.148: cannot open shared object file: No such file or directory

> 解决方法:

> ```
> echo 'include /usr/local/lib' >> /etc/ld.so.conf
> ldconfig
> ```

- ERROR: libfaac not found

> 增加配置 `/etc/yum.repos.d/linuxtech.repo`:

> ```
> [linuxtech]
> name=LinuxTECH
> baseurl=http://pkgrepo.linuxtech.net/el6/release/
> enabled=1
> gpgcheck=1
> gpgkey=http://pkgrepo.linuxtech.net/el6/release/RPM-GPG-KEY-LinuxTECH.NET
> ```
> 
> 安装 libfaac-devel 包
> 
> ```
> yum install libfaac-devel
> ```

- ERROR: libmp3lame >= 3.98.3 not found

> ```
> yum install libmp3lame-devel
> ```

- ERROR: libopencore_amrnb not found

> ```
> yum --enablerepo=naulinux-school install libopencore-amrnb0
> wget http://ftp.altlinux.org/pub/distributions/ALTLinux/Sisyphus/x86_64/RPMS.classic/libopencore-amrnb-devel-0.1.3-alt1.git20140714.x86_64.rpm
> rpm -ivh libopencore-amrnb-devel-0.1.3-alt1.git20140714.x86_64.rpm
> ```

- ERROR: libopencore_amrwb not found

> ```
> yum --enablerepo=naulinux-school install libopencore-amrwb
> ```





rpm包 搜索 **https://pkgs.org/search/libopencore_amrnb**