---
layout: post
title: MQTT协议及拓展
excerpt: MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是IBM开发的一个即时通讯协议。
---

## 1. 什么是MQTT协议

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是IBM开发的一个即时通讯协议。

MQTT协议是为大量计算能力有限，且工作在低带宽、不可靠的网络的远程传感器和控制设备通讯而设计的协议，它具有以下主要的几项特性：

1. 使用发布(Publish)/订阅/(Subscribe)消息模式，提供一对多的消息发布，解除应用程序耦合

2. 对负载内容屏蔽的消息传输

3. 使用_TCP/IP_提供网络连接

4. 有三种消息发布服务质量

5. 小型传输，开销很小（固定长度的头部是 2 字节），协议交换最小化，以降低网络流量

6. 使用 Last Will 和 Testament 特性通知有关各方客户端异常中断的机制



> **“至多一次”**，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。
> 
> **“至少一次”**，确保消息到达，但消息重复可能会发生。
> 
> **“只有一次”**，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。

#### 参考百度百科MQTT

- 简单的说MQTT协议是一个轻量级的即时通讯协议
- 因为它被运用在一些硬件和网络不太好的环境，所以它对设备的要求不会太高，以适应艰难的环境。
- 同时要保证信息传递的质量，所以有三种发布质量模式（QoS）
- 它原生条件下是基于TCP/IP的的应用层协议
- 屏蔽了消息传输的具体数据交互格式。也就是不用关心底层是怎么传输的，数据用的是什么格式来传输的
- 在物联网领域（IoT,Internet of Things）未来会有大发展

## 2. MQTT协议的实现

MQTT协议的实现，自然要分为客户端（Client）和服务端（Broker）。服务端也被称为代理，Broker。

在[MQTT的官网](http://mqtt.org/)也给出了一些[客户端的编程库](http://mqtt.org/wiki/doku.php/libraries)和[服务端的实例](http://mqtt.org/wiki/doku.php/brokers)。

服务端，或者叫代理(Broker),有很多，见下图：

![Broker](http://westionblog.qiniudn.com/brokers.png)

这些服务端有重有轻，有些是开源的，有些是企业级公司的收费产品。

像有些是IBM的一些产品框架，是要收费的。

当然也有一些开源的轻量级Broker，还有Apache的一些开源产品。

## 3. MQTT over WebSocket

“XXX over yyy” 的意思是在yyy的基础上实现XXX。

比如`MQTT over WebSocket`是指在WebSocket的基础上实现MQTT协议。

前面在说特点的时候，原生的MQTT协议应该是在运输层 TCP/IP协议的基础上的应用层协议。单独来看，WebSocket也是应用层的一个协议，是在TCP的基础上的一个全双工通道通信。 但是为了能在浏览器（浏览器只支持WebSocket这种全双工应用层协议，客户端用js实现，现在大部分浏览器都支持）上实现MQTT协议，就把WebSocket协议当做传输层的TCP/IP的角色，让MQTT跑在WebSocket协议上。

**虽然MQTT基因层面选择了TCP作为通信通道，但我们添加个编解码方式，MQTT Over Websocket也可以的**

参考[MQTT协议笔记之mqtt.io项目Websocket协议支持](http://www.blogjava.net/yongboy/archive/2014/05/26/414130.html)

类似的跑在别的协议之上的协议的实现还有很多，尽管两者协议在性质上是平级的，都是应用层的协议

当然Broker也要能支持这种模式。有些Broker是可以接受跑在WebSocket上的MQTT协议的实现，有些则不支持，只支持原生在TCP协议基础上的实现。

所以这种的服务端就不能被JS实现的客户端连接。 服务端支不支持WebSocket的这种模式，可以看Broker的介绍里有没有说。

客户端（Client）除了JS的实现，其他的基本都是在TCP/IP协议基础上的，JS的实现是应该在浏览器上的，所以要在浏览器已实现的协议上实现MQTT协议，而WebSocket是最符合的。

这个模块先讲到这，下面内容还会回来用到。下面先讲个Broker的实例

## 4. 一个Broker实例

说了这么多，看个实例 这里有个叫 [Mosquitto](http://mosquitto.org/)（这不是蚊子吗？蚊子是mosquito）

![Mosquitto](http://westionblog.qiniudn.com/mosquitto.png)

是一个开源的MQTT v3.1 Broker 这个Broker只有几百k,貌似是MQTT协议的创始人用C写的，非常精简,功能也非常强大。网上有朋友测过，可以承受20000人同时在线，自己还没测过，有空可以测一下。并且这个服务端有提供网桥功能。大概的意思是可以把服务器连在一起吧

Mosquitto 启动的时候有一个配置文件，在这个配置文件里可以设置一些参数，通过这些参数设定连接的一些条件和机制，比如timeout相关的和安全相关的认证机制。启动之后，服务器就可以开着，然后就通过客户端实现消息的发布（Publish）和订阅（Subscribe）

Eclipse有一个开源项目叫 [Paho](http://www.eclipse.org/paho/)，就是一个实现MQTT协议客户端的项目，它实现了好多语言的客户端的封装（甚至有嵌入式），还有一些工具（utilities）只要按照它的实例开发，很方便，并且文档很全。

![paho](http://westionblog.qiniudn.com/paho.png)

再次感谢开源项目，OpenSource精神

所以Android要实现推送，这里有现成的客户端。服务器就用Mosquitto,基本就差不多了。不过我还没亲测大用户量的情况。

关于Mosquito的使用，比如参数配置，还有客户端的实例编程，先不具体放代码，换一篇单独详细介绍。

## 5. WebSocket–TCP proxy





- [Icon and Image Sizes](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/IconMatrix.html#//apple_ref/doc/uid/TP40006556-CH27)

- [Launch Images](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/LaunchImages.html#//apple_ref/doc/uid/TP40006556-CH22-SW1)
