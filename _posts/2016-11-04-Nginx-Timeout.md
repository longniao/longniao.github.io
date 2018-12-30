---
layout: post
title: nginx中的超时设置
excerpt: nginx使用proxy模块时，默认的读取超时时间是60s。
---


## 问题：504 Gateway Time-out

常见于使用nginx作为web server的服务器的网站

我遇到这个问题是在升级discuz论坛的时候遇到的

一般看来, 这种情况可能是由于nginx默认的fastcgi进程响应的缓冲区太小造成的, 这将导致fastcgi进程被挂起, 如果你的fastcgi服务对这个挂起处理的不好, 那么最后就极有可能导致504 Gateway Time-out现在的网站, 尤其某些论坛有大量的回复和很多内容的, 一个页面甚至有几百K默认的fastcgi进程响应的缓冲区是8K, 我们可以设置大点在nginx.conf里, 加入:

fastcgi_buffers 8 128k

这表示设置fastcgi缓冲区为8×128k，当然如果您在进行某一项即时的操作, 可能需要nginx的超时参数调大点, 例如设置成3秒:

send_timeout 3;

调整了这两个参数, 结果就是没有再显示那个超时, 可以说效果不错, 但是也可能是由于其他的原因, 目前关于nginx的资料不是很多, 很多事情都需要长期的经验累计才有结果。

```
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size 10m;
client_body_buffer_size 128k;
proxy_connect_timeout 3;
proxy_send_timeout 3;
proxy_read_timeout 3;
proxy_buffer_size 4k;
proxy_buffers 32 4k;
proxy_busy_buffers_size 64k;
```



## 1. send_timeout

```
syntax: send_timeout the time
default: send_timeout 60
context: http, server, location

Directive assigns response timeout to client. Timeout is established not on entire transfer of answer, but only between two operations of reading, if after this time client will take nothing, then nginx is shutting down the connection.
```

## 2. 负载均衡配置时的2个参数：`fail_timeout` 和 `max_fails`

这2个参数一起配合，来控制nginx怎样认为upstream中的某个server是失效的当在fail_timeout的时间内，某个server连接失败了max_fails次，则nginx会认为该server不工作了。同时，在接下来的 fail_timeout时间内，nginx不再将请求分发给失效的server。

个人认为，nginx不应该把这2个时间用同一个参数fail_timeout来控制，要是能再增加一个fail_time，来控制接下来的多长时间内，不再使用down掉的server就更好了~

如果不设置这2个参数，fail_timeout默认为10s，max_fails默认为1。就是说，只要某个server失效一次，则在接下来的10s内，就不会分发请求到该server上

## 3. proxy模块的 `proxy_connect_timeout`

```
syntax: proxy_connect_timeout timeout_in_seconds

context: http, server, location

This directive assigns a timeout for the connection to the proxyserver. This is not the time until the server returns the pages, this is the proxy_read_timeout statement. If your proxyserver is up, but hanging (e.g. it does not have enough threads to process your request so it puts you in the pool of connections to deal with later), then this statement will not help as the connection to the server has been made. It is necessary to keep in mind that this time out cannot be more than 75 seconds.
```

## 4. proxy模块的 `proxy_read_timeout`

```
syntax: proxy_read_timeout the_time

default: proxy_read_timeout 60

context: http, server, location

This directive sets the read timeout for the response of the proxied server. It determines how long NGINX will wait to get the response to a request. The timeout is established not for entire response, but only between two operations of reading.

In contrast to proxy_connect_timeout, this timeout will catch a server that puts you in it's connection pool but does not respond to you with anything beyond that. Be careful though not to set this too low, as your proxy server might take a longer time to respond to requests on purpose (e.g. when serving you a report page that takes some time to compute). You are able though to have a different setting per location, which enables you to have a higher proxy_read_timeout for the report page's location.

If the proxied server nothing will communicate after this time, then nginx is shut connection.
```
