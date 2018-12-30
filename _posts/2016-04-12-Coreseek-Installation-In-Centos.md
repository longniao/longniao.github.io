---
layout: post
title: 在CentOS6.4下安装Coreseek/Sphinx 
excerpt: Coreseek/Sphinx的安装说明及一些错误处理方法。
---

## 1. 在CentOS6.4下安装coreseek之前需要预先安装以下软件  


1. 打开终端 输入 su 获取管理员权限

-  输入命令 
`yum install make gcc g++ gcc-c++ libtool autoconf automake imake mysql-devel libxml2-devel expat-devel`

**在使用命令的时候出现如下错误**

`Error: Cannot find a valid baseurl for repo: addons`

**解决方法**

`echo "nameserver 8.8.8.8" >> /etc/resolv.conf`

## 2. 下载安装coreseek-4.1

```
wget http://www.coreseek.cn/uploads/csft/4.0/coreseek-4.1-beta.tar.gz

tar xzvf `coreseek-4.1-beta.tar.gz

cd coreseek-4.1-beta
```

### 2.1 安装mmseg

```
cd mmseg-3.2.14

./bootstrap #输出的warning信息可以忽略，如果出现error则需要解决

./configure --prefix=/usr/local/mmseg3

make && make install

cd ..
```

### 2.2 安装coreseek

```
cd csft-4.1

sh buildconf.sh #输出的warning信息可以忽略，如果出现error则需要解决

./configure --prefix=/usr/local/coreseek --without-unixodbc --with-mmseg --with-mmseg-includes=/usr/local/mmseg3/include/mmseg/ --with-mmseg-libs=/usr/local/mmseg3/lib/ --with-mysql

make && make install

cd ..
```

### 2.3 测试mmseg分词，coreseek搜索

```
cd testpack

cat var/test/test.xml #此时应该正确显示中文 

/usr/local/mmseg3/bin/mmseg -d /usr/local/mmseg3/etc var/test/test.xml 

/usr/local/coreseek/bin/indexer -c etc/csft.conf --all 
```

#### 可能会出现的错误：

`/usr/local/coreseek/bin/indexer: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory`

#### 解决方案：

`32位系统：ln -s /usr/local/mysql/lib/libmysqlclient.so.18  /usr/lib/</span>`

`64位系统：ln -s /usr/local/mysql/lib/libmysqlclient.so.18  /usr/lib64/`

### 2.4 测试

`/usr/local/coreseek/bin/search -c etc/csft.conf 网络搜索`

** 至此全部安装成【后续需要配置】**

### 2.5 你可能会遇到的问题

#### 2.5.1 make error

```
make[2]: *** [indexer] 错误 1 

make[2]: Leaving directory `/setup/coreseek-3.2.14/csft-3.2.14/src' 

make[1]: *** [all] 错误 2 

make[1]: Leaving directory `/setup/coreseek-3.2.14/csft-3.2.14/src' 

make: *** [all-recursive] 错误 1
```

**解决方案**

编辑：`./src/MakeFile文件`

将

`LIBS = -lm -lexpat -L/usr/local/lib`

改成

`LIBS = -lm -lexpat -liconv -L/usr/local/lib`

#### 2.5.2 install error

```
error while loading shared libraries: libiconv.so.2: cannot open shared object file: No such file or directory
```

**解决方案**

在/etc/ld.so.conf中加一行/usr/local/lib，运行ldconfig

