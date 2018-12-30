---
layout: post
title: Mac OS 10.9.2安装mysql+php+apache,并且配置上Phalcon和yaf扩展
excerpt: 很早以前的教程就是让共享上开启apache就可以了，但是苹果已经取消了这个选项，现在我们怎么做呢？今天我就讲解开发环境的搭建和使用.
---

>很早以前的教程就是让共享上开启apache就可以了，但是苹果已经取消了这个选项，现在我们怎么做呢？今天我就讲解开发环境的搭建和使用

####1.apache的启用和配置

    #启动apache
    sudo apachectl start
    
    #配置文件目录
    cd /etc/apache2/
    
    #开启虚拟主机
    cd /etc/apache2 && sudo vim httpd.conf
    
    #加载php,去掉LoadModule php5_module libexec/apache2/libphp5.so前边的#
    LoadModule php5_module libexec/apache2/libphp5.so
    
    #找到如下，去掉Include前边的#
    # Virtual hosts
    Include /private/etc/apache2/extra/httpd-vhosts.conf
    
#####配置虚拟主机

>一般我们都是开发用，所以都自己配置自己简单喜欢的就行，我的配置如下，我打算安装phalcon这个扩展

    cd /etc/apache2/extra && sudo vim httpd-vhosts.conf
    
    #我的简单配置如下，DocumentRoot是你的目录，ServerName是你的域名
    <VirtualHost *:80>
        ServerAdmin webmaster@dummy-host.example.com
        DocumentRoot "/Users/widuu/wwwroot/phalcon"
        ServerName   phalcon
    </VirtualHost>
    
    #配置hosts
    
    sudo vim /etc/hosts
    
    #添加
    127.0.0.1 phalcon
    
    cd /Users/widuu/wwwroot/phalcon && vim index.php
    
    #写入
    <?php phpinfo();?>
    
    #打开浏览器，输入phalcon，这时候phpinfo的信息就全出来了
    
####2.安装phalcon和yaf扩展
    
    #打开终端先安装命令行开发者工具，如果不安装这个会出现
    
    #include "php.h"
         ^
    1 error generated.
    make: *** [phalcon.lo] Error 1 或者 make: *** [yaf.lo] Error 1的错误
    
    #输入如下命令安装命令行工具
    xcode-select --install
    
#####phalcon安装方法

    git clone --depth=1 git://github.com/phalcon/cphalcon.git
    cd cphalcon/build
    sudo ./install

    此命令默认安装master phalcon1.3.4，如果要安装2.0，请执行

    git clone http://github.com/phalcon/cphalcon
    git checkout 2.0.0
    cd ext
    sudo ./install-test

#####yaf安装方法

    wget http://pecl.php.net/get/yaf-2.3.2.tgz
    tar zxvf yaf-2.3.2.tgz && cd yaf-2.3.2
    phpize
    ./configure --with-php-config=/usr/bin/php-config
    make && sudo make install
    
>安装完成之后，都会显示一个安装的目录，譬如我的是

    Installing shared extensions:     /usr/lib/php/extensions/no-debug-non-zts-20100525/
    
>这个时候你配置php.ini 如果/etc下没有php.ini 你执行如下

 cp /etc/php.ini.default /etc/php.ini
 sudo chmod 755 /etc/php.ini
 
>配置环境

    sudo vim /etc/php.ini
    
    #添加如下
    extension=/usr/lib/php/extensions/no-debug-non-zts-20100525/yaf.so
    extension=/usr/lib/php/extensions/no-debug-non-zts-20100525/phalcon.so
    
    #重启apache
    sudo apachectl retstart

>测试安装是否成功

    <?php
    
    echo "<pre>";
    print_r(get_loaded_extensions());
    echo "</pre>";
    
>如图安装成功

![安装成功](https://camo.githubusercontent.com/e7f2c38f650d86ba6a8ab53021cf848208cea36a/687474703a2f2f77696475752e752e71696e6975646e2e636f6d2f402f696d616765732f7961662e706e67)
    
####3.mysql安装配置

 下载地址:[http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)，然后根据自己的系统下载对应的，我下载的Mac OS X 10.7 (x86, 64-bit), DMG Archive

 ![mysql下载截图](https://camo.githubusercontent.com/39b3fc7ca4fb2f9efbaa1576708492147e680c04/687474703a2f2f77696475752e752e71696e6975646e2e636f6d2f402f696d616765732f6d7973716c312e706e67)
 
 进入下载页面下载

 ![mysql下载截图](https://camo.githubusercontent.com/9671fc9d8c55bb6c854341fa4e54cbe291df2a4a/687474703a2f2f77696475752e752e71696e6975646e2e636f6d2f402f696d616765732f6d7973716c322e706e67) 
 
 双击打开下载的文件，会有三个文件
    
    mysql-5.6.17-osx10.7-x86_64.pkg
    MySQLStartupItem.pkg
    MySQL.prefPane
    
然后依次打开安装,安装好后，在macos中的系统偏好设置会有mysql的图标，这时候我们点击，然后启动mysql就可以了

![打开mysql](https://camo.githubusercontent.com/088448107b67e00ccab5ace862c19ebf1e7cb591/687474703a2f2f77696475752e752e71696e6975646e2e636f6d2f402f696d616765732f6d7973716c332e706e67)

#####配置mysql

    cd ; vim .bash_profile
    
    #点击i进入编辑模式加入下边的代码
    export PATH="/usr/local/mysql/bin:$PATH"
    
    #保存退出，执行如下命令
    source ~/.bash_profile
    
#####修改mysql密码

    mysqladmin -u root password '这里填你要设置的密码'
    
    
 
