---
layout: post
title: Centos6安装STUN服务器
excerpt: 虽然免费的stun服务器也能凑合着用，但是也越来越不适用于中国的网络环境，所以你可以到大的IDC去买云主机并购买额外IP来搭建STUN服务器。
---

###下载stun

    wget http://ncu.dl.sourceforge.net/project/stun/stun/0.97/stund-0.97.tgz

###检查安装编译环境

    yum -y install gcc automake autoconf libtool make

###解压

    tar -zxvf stund-0.97.tgz

###进入目录

    cd stund

###编译

    make

###复制服务端

    cp server /usr/bin/stunserver

###创建服务脚本

    vi /etc/init.d/stund

###然后填充服务脚本

	#!/bin/bash
	# 
	#chkconfig: 2345 70 30
	#description:STUN Server
	RETVAL=0 
	start(){ #启动服务的入口函数  
	echo  "STUN Server is started..."  
	/usr/bin/stunserver -v -h 127.0.0.1  -a 192.168.65.5 -v
	#主IP 辅助IP
	}  
	   
	stop(){ #关闭服务的入口函数  
	echo  "STUN Server is stoped..."  
	}  
	   
	#使用case选择  
	case $1 in  
	start)  
	start  
	;;  
	stop)  
	stop  
	;;  
	*)  
	echo "error choice ! please input start or stop";;  
	esac  
	exit $RETVA

###按【ESC】输入:wq保存退出

###设置权限

	chmod +x /etc/init.d/stund

###添加服务

	chkconfig --add stund

###设置防火墙

	setup

###找到Firewall设置 把Enable去掉，然后Save保存退出

###为了安全 可以修改root用户名和设置denyhosts

###重启后，你就可以下载NatTypeTester测试你的服务器了。

