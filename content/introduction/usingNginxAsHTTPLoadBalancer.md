+++
title = "使用 Nginx 作为 HTTP 负载均衡器"
date = 2023-08-14T17:37:07+08:00
weight = 151
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Using nginx as HTTP load balancer - 使用 Nginx 作为 HTTP 负载均衡器

https://nginx.org/en/docs/http/load_balancing.html

## 简介 - Introduction

Load balancing across multiple application instances is a commonly used technique for optimizing resource utilization, maximizing throughput, reducing latency, and ensuring fault-tolerant configurations.

​	在多个应用实例之间进行负载均衡是一种常用的技术，用于优化资源利用、最大化吞吐量、减少延迟，并确保容错配置。

It is possible to use nginx as a very efficient HTTP load balancer to distribute traffic to several application servers and to improve performance, scalability and reliability of web applications with nginx.

​	可以使用 nginx 作为高效的 HTTP 负载均衡器，将流量分发到多个应用服务器，并通过 nginx 提升 Web 应用的性能、可扩展性和可靠性。



## 负载均衡方法 - Load balancing methods

The following load balancing mechanisms (or methods) are supported in nginx:

​	nginx 支持以下负载均衡机制（或方法）：

- round-robin — requests to the application servers are distributed in a round-robin fashion,
- round-robin（轮询）— 将请求以轮询方式分发给应用服务器。
- least-connected — next request is assigned to the server with the least number of active connections,
- least-connected（最少连接）— 下一个请求分配给具有最少活动连接数的服务器。
- ip-hash — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).
- ip-hash（IP 哈希）— 使用哈希函数确定下一个请求应选择哪个服务器（基于客户端的 IP 地址）。





## 默认负载均衡配置 - Default load balancing configuration

The simplest configuration for load balancing with nginx may look like the following:

​	使用 nginx 进行负载均衡的最简配置可能如下所示：

```
http {
 upstream myapp1 {
     server srv1.example.com;
     server srv2.example.com;
     server srv3.example.com;
 }

 server {
     listen 80;

     location / {
         proxy_pass http://myapp1;
     }
 }
}
```



In the example above, there are 3 instances of the same application running on srv1-srv3. When the load balancing method is not specifically configured, it defaults to round-robin. All requests are [proxied]({{< ref "/mod_ref/ngx_http_proxy_module#proxy_pass">}}) to the server group myapp1, and nginx applies HTTP load balancing to distribute the requests.

​	在上面的示例中，有三个相同应用的实例运行在 srv1-srv3 上。当没有特定配置负载均衡方法时，默认为 round-robin。所有请求都被代理到服务器组 myapp1，nginx 应用 HTTP 负载均衡以分发请求。

Reverse proxy implementation in nginx includes load balancing for HTTP, HTTPS, FastCGI, uwsgi, SCGI, memcached, and gRPC.

​	nginx 的反向代理实现包括 HTTP、HTTPS、FastCGI、uwsgi、SCGI、memcached 和 gRPC 的负载均衡。

To configure load balancing for HTTPS instead of HTTP, just use “https” as the protocol.

​	要为 HTTPS 而不是 HTTP 配置负载均衡，只需将协议设置为“https”。

When setting up load balancing for FastCGI, uwsgi, SCGI, memcached, or gRPC, use [fastcgi_pass]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass">}}), [uwsgi_pass]({{< ref "/mod_ref/ngx_http_uwsgi_module#uwsgi_pass">}}), [scgi_pass]({{< ref "/mod_ref/ngx_http_scgi_module#scgi_pass">}}), [memcached_pass]({{< ref "/mod_ref/ngx_http_memcached_module#memcached_pass">}}), and [grpc_pass]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_pass">}}) directives respectively.

​	在为 FastCGI、uwsgi、SCGI、memcached 或 gRPC 配置负载均衡时，分别使用 [fastcgi_pass]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass">}})、[uwsgi_pass]({{< ref "/mod_ref/ngx_http_uwsgi_module#uwsgi_pass">}})、[scgi_pass]({{< ref "/mod_ref/ngx_http_scgi_module#scgi_pass">}})、[memcached_pass]({{< ref "/mod_ref/ngx_http_memcached_module#memcached_pass">}}) 和 [grpc_pass]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_pass">}}) 指令。



## 最少连接负载均衡 - Least connected load balancing

Another load balancing discipline is least-connected. Least-connected allows controlling the load on application instances more fairly in a situation when some of the requests take longer to complete.

​	另一种负载均衡策略是最少连接。最少连接允许在某些请求需要更长时间完成的情况下，更公平地控制应用实例的负载。

With the least-connected load balancing, nginx will try not to overload a busy application server with excessive requests, distributing the new requests to a less busy server instead.

​	使用最少连接负载均衡，nginx 将尝试不过载繁忙的应用服务器，而是将新请求分配给较不繁忙的服务器。

Least-connected load balancing in nginx is activated when the [least_conn]({{< ref "/mod_ref/ngx_http_upstream_module#least_conn">}}) directive is used as part of the server group configuration:

​	在 nginx 中，当在服务器组配置中使用 [least_conn]({{< ref "/mod_ref/ngx_http_upstream_module#least_conn">}}) 指令时，将激活最少连接负载均衡：

```
 upstream myapp1 {
     least_conn;
     server srv1.example.com;
     server srv2.example.com;
     server srv3.example.com;
 }
```





## 会话持续性 - Session persistence

Please note that with round-robin or least-connected load balancing, each subsequent client’s request can be potentially distributed to a different server. There is no guarantee that the same client will be always directed to the same server.

​	请注意，在 round-robin 或 least-connected 负载均衡中，每个后续客户端请求可能会被分发到不同的服务器。不能保证相同的客户端总是被导向同一服务器。

If there is the need to tie a client to a particular application server — in other words, make the client’s session “sticky” or “persistent” in terms of always trying to select a particular server — the ip-hash load balancing mechanism can be used.

​	如果需要将客户端绑定到特定的应用服务器 — 换句话说，使客户端的会话在始终尝试选择特定服务器方面变得“粘性”或“持续”，可以使用 ip-hash 负载均衡机制。

With ip-hash, the client’s IP address is used as a hashing key to determine what server in a server group should be selected for the client’s requests. This method ensures that the requests from the same client will always be directed to the same server except when this server is unavailable.

​	使用 ip-hash，客户端的 IP 地址被用作散列键，以确定服务器组中为客户端请求选择哪个服务器。该方法确保来自同一客户端的请求将始终被导向同一服务器，除非此服务器不可用。

To configure ip-hash load balancing, just add the [ip_hash]({{< ref "/mod_ref/ngx_http_upstream_module#ip_hash">}}) directive to the server (upstream) group configuration:

​	要配置 ip-hash 负载均衡，只需将 [ip_hash]({{< ref "/mod_ref/ngx_http_upstream_module#ip_hash">}}) 指令添加到服务器（上游）组配置中：

```
upstream myapp1 {
 ip_hash;
 server srv1.example.com;
 server srv2.example.com;
 server srv3.example.com;
}
```





## 加权负载均衡 - Weighted load balancing

It is also possible to influence nginx load balancing algorithms even further by using server weights.

​	还可以通过使用服务器权重进一步影响 nginx 负载均衡算法。

In the examples above, the server weights are not configured which means that all specified servers are treated as equally qualified for a particular load balancing method.

​	在上面的示例中，服务器权重未配置，这意味着所有指定的服务器在特定负载均衡方法中都被视为同等合格。

With the round-robin in particular it also means a more or less equal distribution of requests across the servers — provided there are enough requests, and when the requests are processed in a uniform manner and completed fast enough.

​	对于 round-robin，这也意味着在服务器之间的请求分布更或多或少相等 — 前提是有足够的请求，并且请求以均匀的方式处理且足够快。

When the [weight]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) parameter is specified for a server, the weight is accounted as part of the load balancing decision.

​	当为服务器指定 [weight]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 参数时，该权重将纳入负载均衡决策。

```
 upstream myapp1 {
     server srv1.example.com weight=3;
     server srv2.example.com;
     server srv3.example.com;
 }
```



With this configuration, every 5 new requests will be distributed across the application instances as the following: 3 requests will be directed to srv1, one request will go to srv2, and another one — to srv3.

​	通过此配置，每 5 个新请求将分布到应用实例，如下所示：3 个请求将导向 srv1，一个请求将导向 srv2，另一个请求将导向 srv3。

It is similarly possible to use weights with the least-connected and ip-hash load balancing in the recent versions of nginx.

​	在 nginx 的最新版本中，最少连接和 ip-hash 负载均衡也可以使用权重。



## 健康检查 - Health checks

Reverse proxy implementation in nginx includes in-band (or passive) server health checks. If the response from a particular server fails with an error, nginx will mark this server as failed, and will try to avoid selecting this server for subsequent inbound requests for a while.

​	nginx 的反向代理实现包括带内（或被动）服务器健康检查。如果来自特定服务器的响应失败，nginx 将标记该服务器为失败，并会在一段时间内尝试避免选择此服务器进行后续入站请求。

The [max_fails]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) directive sets the number of consecutive unsuccessful attempts to communicate with the server that should happen during [fail_timeout]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}). By default, [max_fails]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) is set to 1. When it is set to 0, health checks are disabled for this server. The [fail_timeout]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) parameter also defines how long the server will be marked as failed. After [fail_timeout]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) interval following the server failure, nginx will start to gracefully probe the server with the live client’s requests. If the probes have been successful, the server is marked as a live one.

​	[max_fails]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 指令设置在 [fail_timeout]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 期间与服务器通信的连续失败尝试次数。默认情况下，[max_fails]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 设置为 1。当设置为 0 时，对于此服务器禁用健康检查。[fail_timeout]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 参数还定义了服务器标记为失败的持续时间。在服务器失败后的 [fail_timeout]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 间隔后，nginx 将开始通过实时客户端请求优雅地探测服务器。如果探测成功，该服务器被标记为活动。



## 进一步阅读 - Further reading

In addition, there are more directives and parameters that control server load balancing in nginx, e.g. [proxy_next_upstream]({{< ref "/mod_ref/ngx_http_proxy_module#proxy_next_upstream">}}), [backup]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}), [down]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}), and [keepalive]({{< ref "/mod_ref/ngx_http_upstream_module#keepalive">}}). For more information please check our [reference documentation](https://nginx.org/en/docs/).

​	此外，nginx 中还有更多指令和参数用于控制服务器负载均衡，例如 [proxy_next_upstream]({{< ref "/mod_ref/ngx_http_proxy_module#proxy_next_upstream">}})、[backup]({{< ref "/mod_ref/ngx_http_upstream_module#server">}})、[down]({{< ref "/mod_ref/ngx_http_upstream_module#server">}}) 和 [keepalive]({{< ref "/mod_ref/ngx_http_upstream_module#keepalive">}})。有关更多信息，请查阅我们的[参考文档](https://nginx.org/en/docs/)。

Last but not least, [application load balancing](https://www.nginx.com/products/application-load-balancing/), [application health checks](https://www.nginx.com/products/application-health-checks/), [activity monitoring](https://www.nginx.com/products/live-activity-monitoring/) and [on-the-fly reconfiguration](https://www.nginx.com/products/on-the-fly-reconfiguration/) of server groups are available as part of our paid NGINX Plus subscriptions.

​	最后但并非最不重要的是，作为我们付费的 NGINX Plus 订阅的一部分，我们提供[应用负载均衡](https://www.nginx.com/products/application-load-balancing/)、[应用健康检查](https://www.nginx.com/products/application-health-checks/)、[活动监控](https://www.nginx.com/products/live-activity-monitoring/) 和 [实时重新配置](https://www.nginx.com/products/on-the-fly-reconfiguration/)。

The following articles describe load balancing with NGINX Plus in more detail:

​	以下文章详细描述了在 NGINX Plus 中进行负载均衡：

- [Load Balancing with NGINX and NGINX Plus](https://www.nginx.com/blog/load-balancing-with-nginx-plus/)
- [Load Balancing with NGINX and NGINX Plus part 2](https://www.nginx.com/blog/load-balancing-with-nginx-plus-part2/)
- [使用 NGINX 和 NGINX Plus 进行负载均衡](https://www.nginx.com/blog/load-balancing-with-nginx-plus/)
- [使用 NGINX 和 NGINX Plus 进行负载均衡（第二部分）](https://www.nginx.com/blog/load-balancing-with-nginx-plus-part2/)