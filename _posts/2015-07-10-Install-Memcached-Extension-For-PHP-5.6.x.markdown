---
layout: post
title: php5.6.x下安装memcached
excerpt: Memcache笔记-php5.6.x下安装memcached
---

###1、先安装libevent

    yum install gcc libevent libevent-devel

###2、安装libmemcached

    https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz

    #./configure --prefix=/usr/local/libmemcached --with-memcached      ……OK

    #make       ……error

    解决：安装gcc44 gcc44-c++
    yum  install  gcc*

    然后
    export CC="gcc44"
    export CXX="g++44"

###3.重新安装

    # ./configure  ……OK
    #make     ……OK

###4、安装memcached

    http://pecl.php.net/package/memcached

    /home/service/php-5.6.4/bin/phpize

    ./configure --with-php-config=/home/service/php-5.6.4/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached

    make && make install

###5、添加扩展

    extension = /dir/memcached.so
