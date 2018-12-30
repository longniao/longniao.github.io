---
layout: post
title: 让MySQL支持Emoji表情
excerpt: 让MySQL支持Emoji表情，涉及无线相关的 MySQL 数据库建议都提前采用 utf8mb4 字符集。
---

utf8mb4和utf8到底有什么区别呢？原来以往的mysql的utf8一个字符最多3字节，而utf8mb4则扩展到一个字符最多能有4字节，所以能支持更多的字符集。

### 解决方案

将Mysql的编码从utf8转换成utf8mb4。

`需要 >= MySQL 5.5.3版本、从库也必须是5.5的了、低版本不支持这个字符集、复制报错`

#### 1. 停止MySQL Server服务

#### 2. 修改 my.cnf或者mysql.ini

```
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

#### 3. 重启 MySQL Server、检查字符集

```
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```

![](http://www.linuxidc.com/upload/2015_04/150406062777081.png)

#### 4. 修改字符集

修改数据库字符集

```
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

修改表的字符集：

```
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

修改字段的字符集：

```
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

如果只是某个字段需要 只需要修改那个字段的字符集就可以了

另外服务器连接数据库 Connector/J的连接参数中，不要加characterEncoding参数。 不加这个参数时，默认值就时autodetect。