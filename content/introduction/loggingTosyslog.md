+++
title = "日志记录到 syslog"
date = 2023-08-14T16:52:07+08:00
weight = 90
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Logging to syslog - 日志记录到 syslog

https://nginx.org/en/docs/syslog.html

The [error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) and [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directives support logging to syslog. The following parameters configure logging to syslog:

​	[error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) 和 [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) 指令支持将日志记录到 syslog。以下参数用于配置日志记录到 syslog：

## `server=address`

Defines the address of a syslog server. The address can be specified as a domain name or IP address, with an optional port, or as a UNIX-domain socket path specified after the “`unix:`” prefix. If port is not specified, the UDP port 514 is used. If a domain name resolves to several IP addresses, the first resolved address is used.

​	定义 syslog 服务器的地址。地址可以指定为域名或 IP 地址，可以在可选的端口之后指定，或者在“`unix:`”前缀之后指定 UNIX 域套接字路径。如果未指定端口，将使用 UDP 端口 514。如果域名解析为多个 IP 地址，则使用第一个解析的地址。

## `facility=string`

Sets facility of syslog messages, as defined in [RFC 3164](https://datatracker.ietf.org/doc/html/rfc3164#section-4.1.1). Facility can be one of “`kern`”, “`user`”, “`mail`”, “`daemon`”, “`auth`”, “`intern`”, “`lpr`”, “`news`”, “`uucp`”, “`clock`”, “`authpriv`”, “`ftp`”, “`ntp`”, “`audit`”, “`alert`”, “`cron`”, “`local0`”..“`local7`”. Default is “`local7`”.

​	设置 syslog 消息的设施，如 [RFC 3164](https://datatracker.ietf.org/doc/html/rfc3164#section-4.1.1) 中定义。设施可以是 “`kern`”、 “`user`”、 “`mail`”、 “`daemon`”、 “`auth`”、 “`intern`”、 “`lpr`”、 “`news`”、 “`uucp`”、 “`clock`”、 “`authpriv`”、 “`ftp`”、 “`ntp`”、 “`audit`”、 “`alert`”、 “`cron`”、 “`local0`”..“`local7`”。默认为 “`local7`”。

## `severity=string`

Sets severity of syslog messages for [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log), as defined in [RFC 3164](https://datatracker.ietf.org/doc/html/rfc3164#section-4.1.1). Possible values are the same as for the second parameter (level) of the [error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive. Default is “`info`”.Severity of error messages is determined by nginx, thus the parameter is ignored in the `error_log` directive.

​	为 [access_log](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) 设置 syslog 消息的严重性，如 [RFC 3164](https://datatracker.ietf.org/doc/html/rfc3164#section-4.1.1) 中定义。可能的值与 [error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) 指令的第二个参数（级别）的可能值相同。默认为 “`info`”。错误消息的严重性由nginx确定，因此在 `error_log` 指令中此参数将被忽略。

## `tag=string`

Sets the tag of syslog messages. Default is “`nginx`”.

​	设置 syslog 消息的标签。默认为 “`nginx`”。

## `nohostname`

Disables adding the “hostname” field into the syslog message header (1.9.7).

​	禁止将 “hostname” 字段添加到 syslog 消息头中（1.9.7 版本）。



Example syslog configuration:

## 示例 syslog 配置

```
error_log syslog:server=192.168.1.1 debug;

access_log syslog:server=unix:/var/log/nginx.sock,nohostname;
access_log syslog:server=[2001:db8::1]:12345,facility=local7,tag=nginx,severity=info combined;
```

> Logging to syslog is available since version 1.7.1. As part of our [commercial subscription](http://nginx.com/products/) logging to syslog is available since version 1.5.3.
>
> ​	从版本 1.7.1 开始，支持将日志记录到 syslog。作为我们 [商业订阅](http://nginx.com/products/) 的一部分，从版本 1.5.3 开始支持将日志记录到 syslog。