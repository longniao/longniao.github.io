---
layout:     post
title:      修复通过Homebrew安装Mysql5.7不能正常运行的问题
subtitle:   Charles 是在 Mac 下常用的网络封包截取工具，在做移动开发时，我们为了调试与服务器端的网络通讯协议，常常需要截取网络封包来分析。
date:       2022-06-07
author:     Longniao
header-img: img/post-mysql.jpeg
catalog: true
tags:
    - Homebrew
    - Mysql
    - Mac
---

通过 Homebrew 安装 Mysql 5.7:

```
brew install mysql@5.7
```

安装完后通过 `brew services list` 发现 Mysql 启动失败并不停尝试重启。

通过 `cat ~/Library/LaunchAgents/homebrew.mxcl.mysql@5.7.plist` 查看 Mysql 相关信息：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>KeepAlive</key>
	<true/>
	<key>Label</key>
	<string>homebrew.mxcl.mysql@5.7</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/local/opt/mysql@5.7/bin/mysqld_safe</string>
		<string>--datadir=/usr/local/var/mysql</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>WorkingDirectory</key>
	<string>/usr/local/var/mysql</string>
</dict>
</plist>
```

找到 Mysql 的配置文件：`/usr/local/etc/my.cnf` 进行修改：


```
注释掉 
# mysqlx-bind-address = 127.0.0.1
增加
explicit_defaults_for_timestamp=ON
```

初始化数据库文件

```
/usr/local/opt/mysql@5.7/bin/mysqld --initialize-insecure --explicit_defaults_for_timestamp --datadir=/usr/local/var/mysql
```

建立软链

```
ln -s /usr/local/opt/mysql@5.7/bin/mysql /usr/local/bin/mysql
```

现在应该可以正常启动了