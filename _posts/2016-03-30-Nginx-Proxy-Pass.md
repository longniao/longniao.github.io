---
layout: post
title: nginx proxy_pass location 笔记
excerpt: 在nginx中配置proxy_pass时，一些规则整理。
---

在nginx中配置proxy_pass时，如果是按照^~匹配路径时

要注意proxy_pass后的url最后的/

当加上了/，相当于是绝对根路径，则nginx不会把location中匹配的路径部分代理走

如果没有/，则会把匹配的路径部分也给代理走

```
location ^~ /static_js/
{
	proxy_cache js_cache; 
	proxy_set_header Host js.test.com; 
	proxy_pass http://js.test.com/; 
}
```

H如上面的配置，如果请求的url是http://servername/static_js/test.html

会被代理成http://js.test.com/test.html

而如果这么配置:

```
location ^~ /static_js/
{ 
	proxy_cache js_cache; 
	proxy_set_header Host js.test.com; 
	proxy_pass  http://js.test.com; 
}

```

则会被代理到http://js.test.com/static_js/test.htm

当然，我们可以用如下的rewrite来实现/的功能

```
location ^~ /static_js/
{ 
	proxy_cache js_cache; 
	proxy_set_header Host js.test.com; 
	rewrite /static_js/(.+)$ /$1 break; 
	proxy_pass http://js.test.com; 
}

```

见配置，摘自nginx.conf 里的server 段：


```
server {
listen 80;
server_name abc.163.com ;
location / {
	proxy_pass http://ent.163.com/ ;
}
location /star/ {
	proxy_pass http://ent.163.com ;
}
}

```

里面有两个location，我先说第一个，/ 。其实这里有两种写法，分别是：

```
location / {
	proxy_pass http://ent.163.com/ ;
}
 
location / {
	proxy_pass http://ent.163.com ;
}

```

出来的效果都一样的。

第二个location，/star/。同样两种写法都有，都出来的结果，就不一样了。


```
location /star/ {
	proxy_pass http://ent.163.com ;
}

```

当访问 http://abc.163.com/star/ 的时候，nginx 会代理访问到 http://ent.163.com/star/ ，并返回给我们。

```
location /star/ {
	proxy_pass http://ent.163.com/ ;
}
```

当访问 http://abc.163.com/star/ 的时候，nginx 会代理访问到 http://ent.163.com/ ，并返回给我们。

这两段配置，分别在于， proxy_pass http://ent.163.com/ ; 这个”/”，令到出来的结果完全不同。

前者，相当于告诉nginx，我这个location，是代理访问到http://ent.163.com 这个server的，我的location是什么，nginx 就把location 加在proxy_pass 的 server 后面，这里是/star/，所以就相当于 http://ent.163.com/star/。如果是location /blog/ ，就是代理访问到 http://ent.163.com/blog/。

后者，相当于告诉nginx，我这个location，是代理访问到http://ent.163.com/的，http://abc.163.com/star/ == http://ent.163.com/ ，可以这样理解。改变location，并不能改变返回的内容，返回的内容始终是http://ent.163.com/ 。 如果是location /blog/ ，那就是 http://abc.163.com/blog/ == http://ent.163.com/ 。

这样，也可以解释了上面那个location / 的例子，/ 嘛，加在server 的后面，仍然是 / ，所以，两种写法出来的结果是一样的。

PS: 如果是 location ~* ^/start/(.*)\.html 这种正则的location，是不能写”/”上去的，nginx -t 也会报错的了。因为，路径都需要正则匹配了嘛，并不是一个相对固定的locatin了，必然要代理到一个server。

文章出处：[nginx proxy_pass 里的”/”](http://blog.helosa.org/2010/02/10/nginx-proxy_pass.html)
