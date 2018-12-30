---
layout: post
title: 技术管理者的创业工具箱
excerpt: 作为一个技术管理者，在一个10人到几十人的初创企业，都有些什么推荐的工具和系统来组织公司内部的各种系统呢？
---

有刚创业的朋友问，作为一个技术管理者，都有些什么推荐的工具和系统来组织公司内部的各种系统？今天就简单讲讲，目标人群是技术团队在10人到几十人的初创企业，人太少没必要，人太多可能会有更好的选择。

### IT平台

建议把整个IT平台放在amazon上，或者阿里云，表面上看上去要比在本地建立各种服务多花不少钱，但间接的好处不容忽视。这个不多说。

### 身份管理

一个公司的IT系统，最基本的是身份管理（Identity Management）系统，它能识别进入这个王国（realm）的用户是否是一个授权的用户。这里面涉及到很多复杂的协议（LDAP，Kerberos，RADIUS等）和软件，一个个装配起来很麻烦。可以使用freeipa，它是一组开源软件的集合，能帮你解决涉及到身份管理的几乎所有问题。

部署好freeipa后，公司内部的大部分系统都可以与之集成，员工可以一套密码，到处使用。

### 邮箱

国内公司不用想了，QQ企业邮箱吧。

### 聊天/交流

电脑上hipchat是首选，托管免费，自己部署要买license。优先考虑托管。

如果想自己部署，但又舍不得花钱，可以看看开源的kandan。

神马，为什么不用QQ？给跪了。用用hipchat，你就知道为啥工作上的交流不该使用QQ。。。

手机上自然是微信了。

### 代码托管

代码托管可以放在github上，按月交租子就好。github解决了代码托管，知识管理（wiki），问题追踪（bug tracking），代码走读（code review）等基本需求，而且它倡导fork/branching/pull request/review/merge这样的best practice，是第一选择。

如果你对代码这样的「核心资产」托管在别人家里不太放心的话，但又非常习惯于github上的工作流程，可以尝试Atlassian的系列系统：stash/confluence/jira。这些都是需要花钱买license的，如果你不选github，这些可以优先考虑，而且10人以下的企业license基本算是白给（$10），业界良心。

不想花钱，那就gitlab + redmine，没那么好，也够用。

自己做代码托管要想清楚，因为日常备份和灾难恢复会占用开发人员（或者devops）的宝贵时间。建议备份往S3上备，无他，安全，便宜（阿里云有无类似服务我不知道）。

### CI

jenkins是第一选择，其次是travis，这两个都没得说。QuickCheck CI没用过，感兴趣的可以也试试。

### SSL VPN

如果你把各个系统部署到了amazon，很现实的一个问题就是安全性。内部系统不该有公有IP，这样，最大程度地保证了安全性。然而，没有公有IP，员工怎么访问？

我们需要部署OpenVPN，这样每个员工就可以登录到内网（比如说内部的IT系统，建立在10.0.0.0/24的网段）。OpenVPN是个开源软件，自带各个平台的客户端，员工可以安装相应的客户端。mac用户可以上tunnelblick。客户端配合google authenticator，可以实现高大上的密码+动态token级别的保护，创业公司一下子有了大公司的逼格。

出差时想远程访问公司内部系统？在家办公？OpenVPN全帮你搞定。

### 域名管理

貌似现在的选择也就是dnspod，还有其他么？

### 服务集成

hashicorp在github上open source了一个叫consul的系统，golang写的，它用DNS的方式来进行服务发现，很有意思。可以把它部署成一个中间层，屏蔽各个服务的细节，由它来进行中转（通过DNS解析）。consul不光可以对内部IT系统做服务发现，还可以对你公司的产品内部的服务部署和发现提供支持。有空的话我会讲讲consul，它是个非常有意思的服务。

### 开发/测试环境

建议公司统一有dotfiles（可以clone我的 tyrchen/dotfiles），把vim/bash/osx等的基础设置统一。编辑环境可以用osx，运行/测试环境可以放在vagrant上。公司可以提供一个配置妥当的内部的vagrant box，也可以使用标准的box，然后用ansible做provisioning。建议后者，因为反正也要做测试环境和生产环境的部署脚本，ansible的脚本可以做好几套，vagrant/testing/production，而且可以随着代码的变化（比如加了新的dependency）而不断更新。

差不多就这些。一个新员工到来后，admin给创建邮件账户，VPN账户，域账户就OK。为方便计，员工最好一水的mbp，磁盘做加密。devops最好提供给用户一套脚本，可以给新电脑同步dotfiles，安装必备软件（如homebrew，vagrant等），然后provision 最新的 vagrant box。生成完vagrant box后，新员工就可以运行所有的unit testing，TA会欣喜地看到，所有case都跑过 —— 在这个基础上，TA的生产力就可以迅速爆发出来，在新手训练营阶段，就可以hack-test-hack-test。

有什么遗漏么？或者，你有不同的practice？说来听听。:)

PS: 本文所涉概念和软件很多，就不一一提供链接。所有的名词你应该都可以google到，直接google不到，可以 google xxx github 或者 xxx opensource。至于怎么上google（科学上网），我想，这不仅是技术管理者需要掌握的，每个技术从业人员都需要知道，你说呢？
