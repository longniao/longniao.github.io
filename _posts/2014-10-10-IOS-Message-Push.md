---
layout:     post
title:      细说 iOS 消息推送
subtitle:   经常有人问，iOS上推送究竟怎么做啊，为什么我的设备总收不到推送呢，这里跟大家集中讨论一下iOS上推送的实现细节。
date:       2014-10-10
author:     Longniao
header-img: img/post-push-notification.jpg
catalog: true
tags:
    - IOS
    - 推送
    - App
---


<ul id="articleIndex"><li><a href="#articleHeader0">APNS的推送机制</a></li><li><a href="#articleHeader1">推送相关的几个概念</a></li><ul><li><a href="#articleHeader2">消息类型</a></li><li><a href="#articleHeader3">本地消息通知</a></li></ul><li><a href="#articleHeader4">代码里面如何实现推送</a></li><ul><li><a href="#articleHeader5">首先，我们要获取DeviceToken。</a></li><li><a href="#articleHeader6">其次，我们要处理收到消息之后的回调</a></li></ul><li><a href="#articleHeader7">常见问题FAQ</a></li><ul><li><a href="#articleHeader8">我能推送长消息吗</a></li><li><a href="#articleHeader9">推送怎么加声音提醒</a></li><li><a href="#articleHeader10">推送的Badge是怎么回事</a></li><li><a href="#articleHeader11">AVOS Cloud平台发出去的通知格式究竟是什么样子的</a></li><li><a href="#articleHeader12">如何显示本地化的消息</a></li><li><a href="#articleHeader13">应用该怎么响应推送消息</a></li></ul></ul>

经常有同学问，iOS上推送究竟怎么做啊，为什么我的设备总收不到推送呢，这里跟大家集中讨论一下iOS上推送的实现细节。

# APNS的推送机制

与Android上我们自己实现的推送服务不一样，Apple对设备的控制非常严格，消息推送的流程必须要经过APNs：
[![remote_notif_simple_2x](https://blog.avoscloud.com/wp-content/uploads/2014/05/remote_notif_simple_2x.png)](https://blog.avoscloud.com/wp-content/uploads/2014/05/remote_notif_simple_2x.png)

这里 Provider 是指某个应用的Developer，当然如果开发者使用AVOS Cloud的服务，把发送消息的请求委托给我们，那么这里的Provider就是AVOS Cloud的推送服务程序了。上图可以分为三步：

第一步：AVOS Cloud推送服务程序把要发送的消息、目的设备的唯一标识打包，发给APNs。

第二步：APNs在自身的已注册Push服务的应用列表中，查找有相应标识的设备，并把消息发送到设备。

第三步：iOS系统把发来的消息传递给相应的应用程序，并且按照设定弹出Push通知

<!--more-->

为了实现消息推送，有两点非常重要：

1，**App的推送证书**

要能够完整实现一条消息推送，需要我们在App ID中打开Push Notifications，需要我们准备好Provisioning Profile和SSL证书，并且一定要注意Development和Distribution环境是需要分开的。最后，把SSL证书导入到AVOS Cloud平台，就可以尝试远程消息推送了。具体的操作流程可以参考我们的使用指南：[iOS推送证书设置指南](https://cn.avoscloud.com/docs/ios_push_cert.html "iOS推送证书设置指南")。

2，**设备标识DeviceToken**

知道了谁要推送，或者说要推送给哪个App之后，APNs还需要知道推到哪台设备上，这就是设备标识的作用。获取设备标识的流程如下：

[![](https://blog.avoscloud.com/wp-content/uploads/2014/05/registration_sequence_2x.png)](https://blog.avoscloud.com/wp-content/uploads/2014/05/registration_sequence_2x.png)

第一步：App打开推送开关，用户要确认TA希望获得该App的推送消息

第二步：App获得一个DeviceToken

第三步：App将DeviceToken保存起来，这里就是通过`[AVInstallation saveInBackground]`将DeviceToken保存到AVOS Cloud

第四步：当某些特定事件发生，开发者委托AVOS Cloud来发送推送消息，这时候AVOS Cloud的推送服务器就会给APNs发送一则推送消息，APNs最后消息送到用户设备

# 推送相关的几个概念

## 消息类型

一条消息推送过来，可以有如下几种表现形式：

<li>显示一个alert或者banner，展现具体内容</li>

<li>在应用icon上提示一个新到消息数</li>

<li>播放一段声音</li>

开发者可以在每次推送的时候设置，在推送达到用户设备时开发者也可以选择不同的提示方式。

## 本地消息通知

iOS上有两种消息通知，一种是本地消息（Local Notification），一种是远程消息(Push Notification，也叫Remote Notification)，设计这两种通知的目的都是为了提醒用户，现在有些什么新鲜的事情发生了，吸引用户重新打开应用。

本地消息什么时候有用呢？譬如你正在做一个To-do的工具类应用，对于用户加入的每一个事项，都会有一个完成的时间点，用户可以要求这个To-do应用在事项过期之前的某一个时间点提醒一下TA。为了达到这一目的，App就可以调度一个本地通知，在时间点到了之后发出一个Alert消息或者其他提示。

我们在处理推送消息的时候，也可以综合运用这两种方式。

# 代码里面如何实现推送

## 首先，我们要获取DeviceToken。

App需要每次启动的时候都去注册远程通知——通过调用UIApplication的`registerForRemoteNotificationTypes:`方法，传递给它你希望支持的消息类型参数即可，例如：

<pre lang="objc" class=" hljs objectivec"><span class="widget-clipboard"></span>- (<span class="hljs-built_in">BOOL</span>)application:(<span class="hljs-built_in">UIApplication</span> *)application didFinishLaunchingWithOptions:(<span class="hljs-built_in">NSDictionary</span> *)launchOptions
{
    <span class="hljs-comment">// do some initiale working</span>
    ...

    [application registerForRemoteNotificationTypes:UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound];
    <span class="hljs-keyword">return</span> <span class="hljs-literal">YES</span>;
}
</pre>

如果注册成功，APNs会返回给你设备的token，iOS系统会把它传递给app delegate代理——`application:didRegisterForRemoteNotificationsWithDeviceToken:`方法，你应该在这个方法里面把token保存到AVOS Cloud后台，例如：

<pre lang="objc" class=" hljs objectivec"><span class="widget-clipboard"></span>- (<span class="hljs-keyword">void</span>)application:(<span class="hljs-built_in">UIApplication</span> *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    <span class="hljs-built_in">NSLog</span>(<span class="hljs-string">@"Receive DeviceToken: %@"</span>, deviceToken);
    AVInstallation *currentInstallation = [AVInstallation currentInstallation];
    [currentInstallation setDeviceTokenFromData:deviceToken];
    [currentInstallation saveInBackground];
}
</pre>

如果注册失败，`application:didFailToRegisterForRemoteNotificationsWithError:`方法会被调用，通过NSError参数你可以看到具体的出错信息，例如：

<pre lang="objc" class=" hljs objectivec"><span class="widget-clipboard"></span>- (<span class="hljs-keyword">void</span>)application:(<span class="hljs-built_in">UIApplication</span> *)application didFailToRegisterForRemoteNotificationsWithError:(<span class="hljs-built_in">NSError</span> *)error {
    <span class="hljs-built_in">NSLog</span>(<span class="hljs-string">@"注册失败，无法获取设备ID, 具体错误: %@"</span>, error);
}
</pre>

请注意，注册流程需要在app每次启动时调用，这并不不会带来额外的负担，因为iOS操作系统在第一次获得了有效的device token之后，会本地缓存起来，以后app再调用registerForRemoteNotificationTypes:的时候会立刻返回，并不会再进行网络请求。另外，app层面不应该对device token进行缓存，因为device token也有可能变化——如果用户重装了操作系统，那么APNs再次给出的device token就会和之前的不一样，又或者是，用户restore了原来的backup到新的设备上，那么原来的device token也会失效。

## 其次，我们要处理收到消息之后的回调

我们可以设想一下消息通知的几种使用场景：

1，在app没有被启动的时候，接收到了消息通知。这时候操作系统会按照默认的方式来展现一个alert消息，在app icon上标记一个数字，甚至播放一段声音。

2，用户看到消息之后，点击了一下action按钮或者点击了应用图标

如果action按钮被点击了，系统会通过调用`application:didFinishLaunchingWithOptions:`这个代理方法来启动应用，并且会把notification的payload数据传递进去。

如果应用图标被点击了，系统也一样会调用`application:didFinishLaunchingWithOptions:`这个代理方法来启动应用，唯一不同的是这时候启动参数里面不会有任何notification的信息。

示例代码如下：

<pre lang="objc" class=" hljs objectivec"><span class="widget-clipboard"></span>- (<span class="hljs-built_in">BOOL</span>)application:(<span class="hljs-built_in">UIApplication</span> *)application didFinishLaunchingWithOptions:(<span class="hljs-built_in">NSDictionary</span> *)launchOptions
{
    <span class="hljs-comment">// do initializing works</span>
    ...

    <span class="hljs-keyword">if</span> (launchOptions) {
        <span class="hljs-comment">// do something else</span>
        ...

        [AVAnalytics trackAppOpenedWithLaunchOptions:launchOptions];
    }

    [application registerForRemoteNotificationTypes:UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound];

    <span class="hljs-keyword">return</span> <span class="hljs-literal">YES</span>;
}
</pre>

3，如果远程消息发送过来的时候，app正在运行，这时候会发生什么呢？

app代理的`application:didReceiveRemoteNotification:`方法会被调用，同时远程消息中的payload数据会作为参数传递进去。

示例代码如下：

<pre lang="objc" class=" hljs objectivec"><span class="widget-clipboard"></span>- (<span class="hljs-keyword">void</span>)application:(<span class="hljs-built_in">UIApplication</span> *)application didReceiveRemoteNotification:(<span class="hljs-built_in">NSDictionary</span> *)userInfo {
    <span class="hljs-keyword">if</span> (application<span class="hljs-variable">.applicationState</span> == UIApplicationStateActive) {
        <span class="hljs-comment">// 转换成一个本地通知，显示到通知栏，你也可以直接显示出一个alertView，只是那样稍显aggressive：）</span>
        <span class="hljs-built_in">UILocalNotification</span> *localNotification = [[<span class="hljs-built_in">UILocalNotification</span> alloc] init];
        localNotification<span class="hljs-variable">.userInfo</span> = userInfo;
        localNotification<span class="hljs-variable">.soundName</span> = UILocalNotificationDefaultSoundName;
        localNotification<span class="hljs-variable">.alertBody</span> = [[userInfo objectForKey:<span class="hljs-string">@"aps"</span>] objectForKey:<span class="hljs-string">@"alert"</span>];
        localNotification<span class="hljs-variable">.fireDate</span> = [<span class="hljs-built_in">NSDate</span> date];
        [[<span class="hljs-built_in">UIApplication</span> sharedApplication] scheduleLocalNotification:localNotification];
    } <span class="hljs-keyword">else</span> {
        [AVAnalytics trackAppOpenedWithRemoteNotificationPayload:userInfo];
    }
}
</pre>

# 常见问题FAQ

## 我能推送长消息吗

不能，APNs限制了每个notification的payload最大长度是256字节，超长的消息是不能发送的。

## 推送怎么加声音提醒

消息推送是可以指定声音的。譬如你可以对正面的反馈使用欢快的声音，对负面的反馈使用低沉一点的声音，都可以达到别出心裁让人眼前一亮的目的。

你需要先放一些aiff、wav或者caf音频文件到app的资源文件中，然后在推送的时候指定不同的音频文件名就可以了。
![](https://blog.avoscloud.com/wp-content/uploads/2014/05/sendWithSound.png)

## 推送的Badge是怎么回事

推送并不一定会导致应用图标上红色数字增加，是否显示这一数字，显示成多少，都取决于开发者自己。

在发送推送消息的时候，我们可以选择是否递增这一数字；如果不选择这一项，那么消息推送并不会导致应用图标上红色数字的出现。
![](https://blog.avoscloud.com/wp-content/uploads/2014/05/sendWithBadge.png)

好，现在问题来了，这个数字如果搞出来了，怎么让它消失掉呢？

其实我们只需要在任何时候设置 UIApplication.applicationIconBadgeNumber 属性为0，就可以让这个数字消失掉。

一般我们会选择在应用启动的时候（application:didFinishLaunchingWithOptions:方法中），或者干脆一点，在应用每次被切换到前台的时候（applicationWillEnterForeground:方法中），调用这一行代码，即可立刻清除掉Badge数字了。

## AVOS Cloud平台发出去的通知格式究竟是什么样子的

对于每一条推送消息，都包含一个payload，通常是组成了一个JSON的Dictionary，这其中必不可少的是aps属性，它对应的value也是一个Dictionary，包含下面一些内容：

<li>alert消息（文本或Dictionary）</li>

<li>应用图标上的红色数字</li>

<li>播放的声音文件名</li>

在由推送激活的app打开事件中，`application:didFinishLaunchingWithOptions:`的options参数就是这个大的Dictionary对象。

<pre class=" hljs cs"><span class="widget-clipboard"></span>{
    aps =     {
        alert = <span class="hljs-string">"hello, everyone"</span>;
        badge = <span class="hljs-number">2</span>;
        sound = <span class="hljs-keyword">default</span>;
    };
}
</pre>

这里要注意的时alert部分，它的值可以是一个String（文本消息），也可以是一个JSON的Dictionary。当它是文本消息的时候，系统就会把这些文字显示到一个alertview中；如果它也是由一个JSON Dictionary组成的话，其格式如下：

<li>body</li>

<li>action-loc-key</li>

<li>loc-key</li>

<li>loc-args</li>

<li>launch-image</li>

body部分就是alertView中将要展现出来的文本消息，_loc-_属性主要是用来实现本地化消息，launch-image只是app主bundle里的一个图片文件的名称，一般来说我们不指定这一属性。

## 如何显示本地化的消息

有两种办法可以实现推送消息的本地化：

1，在推送的payload中使用loc-key和loc-args来指定进行本地化，这样Provider方只需要按照统一的格式来发送即可，消息的解析和组装则由客户端来完成。

2，如果推送的payload里面不包含loc-key/loc-args信息，那么Provider方就需要自己做本地化处理，然后给不同的device发送不同的消息，为了做到这一点，还需要app在上传device token的时候也把用户的语言设置信息传回来。

目前，因为AVOS Cloud主要就是瞄准中国大陆市场和海外中文用户，所以我们在推送上还不提供多语言支持。

## 应用该怎么响应推送消息

上面说的处理流程，只能简单展示一下远程消息，激活用户让他们重新回到app中来。但是有时候，我们希望带给用户更好的使用体验，譬如如果我们告诉用户：张三刚刚评论了你的照片。这时候用户如果点击action按钮进入app，我们是展示具体的评论页面为好，还是展示通常的启动页面然后让用户自己去找张三的评论好？我想负责任的开发者都会选择前者：）

要做到灵活响应不同类型的通知消息，我们需要在通知的payload中增加更多信息，而不能仅仅只有alert出来的文字信息。对于AVOS Cloud消息推送平台来讲，就需要开发者使用更高级功能的JSON格式。譬如我们发送这样的json字符串
`{"action":{"type":4},"alert":"hello, everyone”} `最终在app内会收到这样的UserInfo Dictionary：

<pre class=" hljs bash"><span class="widget-clipboard"></span>{
    action =     {
        <span class="hljs-built_in">type</span> = <span class="hljs-number">4</span>;
    };
    aps =     {
        alert = <span class="hljs-string">"hello, everyone"</span>;
        badge = <span class="hljs-number">4</span>;
    };
}
</pre>

“hello, everyone”会显示到alertView中，但是整个Dictionary会通过launchOptions传递给application: didFinishLaunchingWithOptions: 方法，这样我们在程序里面就可以对不同的消息实现不同的跳转了。
