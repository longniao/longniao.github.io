---
layout: post
title: MySql ERROR! The server quit without updating PID file
excerpt: Mysql Error 及解决办法。
---

重启电脑发现Mysql 启动失败，报错信息如下：

```
➜  support-files git:(master) sudo ./mysql.server start
Password:
Starting MySQL
. ERROR! The server quit without updating PID file (/usr/local/var/mysql/XianRoudeMBP.pid).
```

直觉告诉我磁盘满了，df查了下，果然是下载数据把磁盘搞满了，删除了些东西后，重动数据库还是出错。

原来是权限错误了，不知道为何磁盘满了后权限就变更了。

```
sudo chown -R mysql:mysql /usr/local/var/mysql
```

修改后正常启动。

Google了下 ，问题可能的原因有多种，整理出来：

**1. 可能是/usr/local/mysql/data/rekfan.pid文件没有写的权限**

解决方法 ：给予权限，执行 “chown -R mysql:mysql /var/data” “chmod -R 755 /usr/local/mysql/data”  然后重新启动mysqld！

**2. 可能进程里已经存在mysql进程**

解决方法：用命令“ps -ef|grep mysqld”查看是否有mysqld进程，如果有使用“kill -9  进程号”杀死，然后重新启动mysqld！

**3. 可能是第二次在机器上安装mysql，有残余数据影响了服务的启动。**

解决方法：去mysql的数据目录/data看看，如果存在mysql-bin.index，就赶快把它删除掉吧，它就是罪魁祸首了。本人就是使用第三条方法解决的 ！http://blog.rekfan.com/?p=186

**4. mysql在启动时没有指定配置文件时会使用/etc/my.cnf配置文件，请打开这个文件查看在[mysqld]节下有没有指定数据目录(datadir)。**

解决方法：请在[mysqld]下设置这一行：datadir = /usr/local/mysql/data

**5. skip-federated字段问题**

解决方法：检查一下/etc/my.cnf文件中有没有没被注释掉的skip-federated字段，如果有就立即注释掉吧。

**6. 错误日志目录不存在**

解决方法：使用“chown” “chmod”命令赋予mysql所有者及权限

**7. selinux惹的祸，如果是centos系统，默认会开启selinux**

解决方法：关闭它，打开/etc/selinux/config，把SELINUX=enforcing改为SELINUX=disabled后存盘退出重启机器试试。
