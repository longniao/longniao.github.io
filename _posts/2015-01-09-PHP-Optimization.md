---
layout:     post
title:      PHP优化杂烩
subtitle:   讲 PHP 优化的文章往往都是教大家如何编写高效的代码，本文打算从另一个角度来讨论问题，教大家如何配置高效的环境，如此同样能够达到优化的目的。
date:       2015-01-09
author:     Longniao
header-img: img/post-php.jpg
catalog: true
tags:
    - PHP
    - 优化
---
讲 PHP 优化的文章往往都是教大家如何编写高效的代码，本文打算从另一个角度来讨论问题，教大家如何配置高效的环境，如此同样能够达到优化的目的。

## pool

一个让人沮丧的消息是绝大多数 PHP 程序员都忽视了池的价值。这里所说的池可不是指数据库连接池之类的东西，而是指进程池，PHP 允许同时启动多个池，每个池使用不同的配置，各个池之间尊重彼此的主权领土完整，互不干涉内政。

![pool](http://huoding.com/wp-content/uploads/2014/12/pool.png)

有什么好处呢？默认情况下，PHP 只启用了一个池，所有请求均在这个池中执行。一旦某些请求出现拥堵之类的情况，那么很可能会连累整个池出现火烧赤壁的结局；如果启用多个池，那么可以把请求分门别类放到不同的池中执行，此时如果某些请求出现拥堵之类的情况，那么只会影响自己所在的池，从而控制故障的波及范围。

## listen

虽然 Nginx 和 PHP 可以部署在不同的服务器上，但是实际应用中，多数人都习惯把它们部署在同一台服务器上，如此就有两个选择：一个是 TCP，另一个是 Unix Socket。

![listen](http://huoding.com/wp-content/uploads/2014/12/listen.jpg)

和 TCP 比较，Unix Socket 省略了一些诸如 TCP 三次握手之类的环节，所以相对更高效，不过需要注意的是，在使用 Unix Socket 时，因为没有 TCP 对应的可靠性保证机制，所以最好把 backlog 和 somaxconn 设置大些，否则面对高并发时会不稳定。

## pm

进程管理有动态和静态之分。动态模式一般先启动少量进程，再按照请求数的多少实时调整进程数。如此的优点很明显：节省资源；当然它的缺点也很明显：一旦出现高并发请求，系统将不得不忙着 FORK 新进程，必然会影响性能。相对应的，静态模式一次性 FORK 足量的进程，之后不管请求量如何均保持不变。和动态模式相比，静态模式虽然消耗了更多的资源，但是面对高并发请求，它不需要执行高昂的 FORK。

![pm](http://huoding.com/wp-content/uploads/2014/12/pm.png)

对大流量网站而言，除非服务器资源紧张，否则静态模式无疑是最佳选择。

## pm.max_children

启动多少个 PHP 进程合适？在你给出自己的答案之前，不妨看看下面的文章：

*   [php-fpm的max_chindren的一些误区](http://www.guangla.com/post/2014-03-14/40061238121)
*   [Should PHP Workers Always Equal Number Of CPUs](http://forum.nginx.org/read.php?3,222702)

一个 CPU 在某一个时刻只能处理一个请求。当请求数大于 CPU 个数时，CPU 会划分时间片，轮流执行各个请求，既然涉及多个任务的调度，那么上下文切换必然会消耗一部分性能，从这个意义上讲，进程数应该等于 CPU 个数，如此一来每个进程都对应一个专属的 CPU，可以把上下文切换损失的效率降到最低。不过这个结论仅在请求是&nbsp;CPU&nbsp;密集型时才是正确的，而对于一般的 Web 请求而言，多半是 IO 密集型的，此时这个结论就值得商榷了，因为数据库查询等 IO 的存在，必然会导致 CPU 有相当一部分时间处于 WAIT 状态，也就是被浪费的状态。此时如果进程数多于 CPU 个数的话，那么当发生 IO 时，CPU 就有机会切换到别的请求继续执行，虽然这会带来一定上下文切换的开销，但是总比卡在 WAIT 状态好多了。

那多少合适呢？要理清这个问题，我们除了要关注 CPU 之外，还要关注内存情况：

![PHP Memory](http://huoding.com/wp-content/uploads/2014/12/php_memory.png)

如上所示 top 命令的结果中和内存相关的列分别是 VIRT，RES，SHR。其中 VIRT 表示的是内存占用的理论值，通常不用在意它，RES 表示的是内存占用的实际值，虽然 RES 看上去很大，但是包含着共享内存，也就是 SHR 显示的值，所以单个 PHP 进程实际独立占用的内存大小等于「RES – SHR」，一般就是&nbsp;10M&nbsp;上下。以此推算，理论上 1G 内存能支撑大概一百个 PHP 进程，10G 内存能大概支撑一千个 PHP 进程。当然并不能粗暴认为越多越好，最好结合 PHP 的 [status](http://php.net/manual/en/install.fpm.configuration.php#pm.status-path) 接口，通过监控活跃连接数的数量来调整。

说明：关于 Web 并发模型方面的知识建议参考[范凯](http://robbinfan.com/)的「[Web并发模型粗浅探讨](http://robbin.iteye.com/blog/1744725)」。

