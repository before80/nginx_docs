+++
title = "ngx_http_access_module"
date = 2023-08-15T08:11:36+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Module ngx_http_access_module

https://nginx.org/en/docs/http/ngx_http_access_module.html



The `ngx_http_access_module` module allows limiting access to certain client addresses.

​	`ngx_http_access_module` 模块允许限制对特定客户端地址的访问。

Access can also be limited by [password](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html), by the [result of subrequest](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html), or by [JWT](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html). Simultaneous limitation of access by address and by password is controlled by the [satisfy]({{< ref "/mod_ref/ngx_http_core_module#satisfy">}}) directive.

​	还可以通过[密码](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)、[子请求的结果](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html)或者[JWT](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html)来限制访问。通过地址和密码同时限制访问的情况由 [satisfy]({{< ref "/mod_ref/ngx_http_core_module#satisfy">}}) 指令控制。



## 示例配置 Example Configuration



```
location / {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
}
```

The rules are checked in sequence until the first match is found. In this example, access is allowed only for IPv4 networks `10.1.1.0/16` and `192.168.1.0/24` excluding the address `192.168.1.1`, and for IPv6 network `2001:0db8::/32`. In case of a lot of rules, the use of the [ngx_http_geo_module](../ngx_http_geo_module) module variables is preferable.

​	规则会按顺序检查，直到找到第一个匹配项。在这个示例中，只允许 IPv4 网络 `10.1.1.0/16` 和 `192.168.1.0/24` 访问，排除地址 `192.168.1.1`，以及 IPv6 网络 `2001:0db8::/32`。如果有很多规则，最好使用 [ngx_http_geo_module](../ngx_http_geo_module) 模块变量。

## 指令 Directives



### allow

  Syntax:  `allow address | CIDR | unix: | all;`

  Default: —

  Context: `http`, `server`, `location`, `limit_except`


Allows access for the specified network or address. If the special value `unix:` is specified (1.5.1), allows access for all UNIX-domain sockets.

​	允许指定的网络或地址访问。如果指定了特殊值 `unix:`（1.5.1），允许访问所有 UNIX 域套接字。

### deny

  Syntax:  `deny address | CIDR | unix: | all;`

  Default: —

  Context: `http`, `server`, `location`, `limit_except`

Denies access for the specified network or address. If the special value `unix:` is specified (1.5.1), denies access for all UNIX-domain sockets.

​	拒绝指定的网络或地址访问。如果指定了特殊值 `unix:`（1.5.1），拒绝访问所有 UNIX 域套接字。