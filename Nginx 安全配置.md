title: "Nginx安全性能配置&&DDOS防范"
date: 2016-03-09 23:17:00
tags: [安全,配置]
----
<!-- more -->
最近看了一些Nginx的配置的文章主要和性能有关，包括一些安全上的配置，并不对所有设备适用，总结下来觉得有用的可以自取，另外是加深自己对服务器的理解。其中有一些有关DDOS的配置。内容参考了两篇文章和自己的一些安全理解。文章地址如下：

[Tuning NGINX for Performance](http://nginx.com/blog/tuning-nginx/)

[Mitigating DDoS Attacks with NGINX and NGINX Plus](https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/)



# Nginx  配置优化

基本配置路径一般在/etc/nginx/nginx.conf,如果站点配置文件不是nginx.conf，而是独立的站点配置文件，那就到相应的站点去修改配置。

` worker_processes  auto` 该选项控制Nginx运行时候的工作进程个数，默认值为1。调节为**auto**

`worker_connections` 表示工作进程能处理的最大连接数。默认为512，可调节为更高，我自己设置为1000。主要看服务器的硬件配置及流量特性。

`keepalive_requests` 表示客户端单个连接上最多能发送多少个请求，默认值是100。可以设置成更高的值，试情况而定。

`keepalive_timeout`指定每个连接最多保持多长的打开状态，为了防止DDOS攻击，可以改为60或者更小。视情况而定。

`server_tokens ` 该选项可以隐藏Nginx的版本号，关闭他可以防止攻击者嗅探到Nginx版本从而做相关渗透。

`worker_rlimit_nofile` 进程最大打开文件数 可以配置为一个较高的数字。避免出现『too many open files』

`proxy_hide_header X-Powered-By;` 该指令可以隐藏一些header的信息。通常**x-powered-by**会泄露网站相关信息我们需要将其隐藏。

**host header attack攻击修复**

在server模块中添加：

```
# Only requests to our Host are allowed
if ($host !~ ^($server_name)$ ) {
return 444;
}
```
# Nginx DDOS 防御配置优化

现在的DDOS基于应用层的比较多，比如CC攻击。通常有如下特点：

- 攻击的IP或IP段相对固定，每个IP都有远大于真实用户的连接数和请求数。

- 因为攻击是由木马发出且目的是使服务器超负荷，请求的频率会远远超过正常人的请求。

- User-Agent通常是一个非标准的值

- Referer有时是一个容易联想到攻击的值

根据以上的相关特征可以做以下配置来抵抗DDOS攻击

- 限制请求速度

`limit_req_zone $binary_remote_addr zone=one:10m rate=2/s;`

- 限制连接数量

`limit_conn_zone $binary_remote_addr zone=addr:10m;`

- 关闭慢连接

在server中添加 

```
server {

client_body_timeout 5s;

client_header_timeout 5s;

}

```

- 设置IP黑/白名单

- 使用缓存进行流量削峰

- 屏蔽特定请求

>1. 针对特定URL的请求
>2. 针对不是常见的User-Agent的请求
>3. 针对Referer头中包含可以联想到攻击的值的请求
>4. 针对其他请求头中包含可以联想到攻击的值的请求


比如，如果你判定攻击是针对一个特定的URL：/foo.php，攻击请求的User-Agent中包含foo或bar 我们就可以屏蔽到这个页面的请求：

``` 
location /foo.php {

deny all;

}


location /{
if ($http_user_agent ~* foo|bar) {

return 403;
}
}

```

# 总结

Nginx和Nginx Plus可以作为抵御DDOS攻击的一个有力手段，而且Nginx Plus中提供了一些附加的特性来更好的抵御DDOS攻击并且当攻击发生时及时的识别到。

