+++
title = "ngx_http_access_module"
date = 2023-08-15T08:11:36+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

## Module ngx_http_access_module

https://nginx.org/en/docs/http/ngx_http_access_module.html



The `ngx_http_access_module` module allows limiting access to certain client addresses.

Access can also be limited by [password](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html), by the [result of subrequest](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html), or by [JWT](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html). Simultaneous limitation of access by address and by password is controlled by the [satisfy](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) directive.



## Example Configuration



> ```
> location / {
>     deny  192.168.1.1;
>     allow 192.168.1.0/24;
>     allow 10.1.1.0/16;
>     allow 2001:0db8::/32;
>     deny  all;
> }
> ```



The rules are checked in sequence until the first match is found. In this example, access is allowed only for IPv4 networks `10.1.1.0/16` and `192.168.1.0/24` excluding the address `192.168.1.1`, and for IPv6 network `2001:0db8::/32`. In case of a lot of rules, the use of the [ngx_http_geo_module](https://nginx.org/en/docs/http/ngx_http_geo_module.html) module variables is preferable.



## Directives



### allow

| Syntax:  | `allow address | CIDR | unix: | all;`        |
| :------- | -------------------------------------------- |
| Default: | —                                            |
| Context: | `http`, `server`, `location`, `limit_except` |

Allows access for the specified network or address. If the special value `unix:` is specified (1.5.1), allows access for all UNIX-domain sockets.



### deny

| Syntax:  | `deny address | CIDR | unix: | all;`         |
| :------- | -------------------------------------------- |
| Default: | —                                            |
| Context: | `http`, `server`, `location`, `limit_except` |

Denies access for the specified network or address. If the special value `unix:` is specified (1.5.1), denies access for all UNIX-domain sockets.