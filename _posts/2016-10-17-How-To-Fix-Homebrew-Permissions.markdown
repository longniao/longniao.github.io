---
layout: post
title: How to fix homebrew permissions?
excerpt: Your user does not have write permissions in /usr/local.
---

在用Homebrew安装[Kong](https://github.com/Mashape/homebrew-kong)的时候，报没有权限

```
Error: Your user does not have write permissions in /usr/local
-- you may want to run as a privileged user or use your local tree with --local.
```

解决方法：

修改安装目录的权限

```
sudo chown -R "$USER":admin /usr/local
```
以及

```
sudo chown -R "$USER":admin /Library/Caches/Homebrew
```

来源：[gitHub's homebrew issue tracker](https://github.com/mxcl/homebrew/issues/19670)