+++
title = "nginx 如何处理 TCP/UDP 会话"
date = 2023-08-14T16:56:39+08:00
weight = 180
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# How nginx processes a TCP/UDP session - nginx 如何处理 TCP/UDP 会话

https://nginx.org/en/docs/stream/stream_processing.html

A TCP/UDP session from a client is processed in successive steps called **phases**:

​	客户端的 TCP/UDP 会话在称为**阶段**的连续步骤中进行处理：

## `Post-accept`

The first phase after accepting a client connection. The [ngx_stream_realip_module]({{< ref "/mod_ref/ngx_stream_realip_module" >}}) module is invoked at this phase.

​	在接受客户端连接后的第一个阶段。此阶段调用 [ngx_stream_realip_module]({{< ref "/mod_ref/ngx_stream_realip_module" >}}) 模块。

## `Pre-access`

  Preliminary check for access. The [ngx_stream_limit_conn_module]({{< ref "/mod_ref/ngx_stream_limit_conn_module" >}}) and [ngx_stream_set_module]({{< ref "/mod_ref/ngx_stream_set_module" >}}) modules are invoked at this phase.

​	用于预先检查访问权限。在此阶段，调用 [ngx_stream_limit_conn_module]({{< ref "/mod_ref/ngx_stream_limit_conn_module" >}}) 和 [ngx_stream_set_module]({{< ref "/mod_ref/ngx_stream_set_module" >}}) 模块。

## `Access`

  Client access limitation before actual data processing. At this phase, the [ngx_stream_access_module]({{< ref "/mod_ref/ngx_stream_access_module" >}}) module is invoked, for [njs](https://nginx.org/en/docs/njs/index.html), the [js_access](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_access) directive is invoked.

​	在实际数据处理之前对客户端访问进行限制。在此阶段，调用 [ngx_stream_access_module]({{< ref "/mod_ref/ngx_stream_access_module" >}}) 模块；对于 [njs](https://nginx.org/en/docs/njs/index.html)，调用 [js_access](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_access) 指令。

## `SSL`

  TLS/SSL termination. The [ngx_stream_ssl_module]({{< ref "/mod_ref/ngx_stream_ssl_module" >}}) module is invoked at this phase.

​	TLS/SSL 终止。在此阶段，调用 [ngx_stream_ssl_module]({{< ref "/mod_ref/ngx_stream_ssl_module" >}}) 模块。

## `Preread`

  Reading initial bytes of data into the [preread buffer](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#preread_buffer_size) to allow modules such as [ngx_stream_ssl_preread_module]({{< ref "/mod_ref/ngx_stream_ssl_preread_module" >}}) analyze the data before its processing. For [njs](https://nginx.org/en/docs/njs/index.html), the [js_preread](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_preread) directive is invoked at this phase.

​	将初始数据的前几个字节读入 [预读缓冲区](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#preread_buffer_size)，以便让模块（例如 [ngx_stream_ssl_preread_module]({{< ref "/mod_ref/ngx_stream_ssl_preread_module" >}})）在处理数据之前分析数据。对于 [njs](https://nginx.org/en/docs/njs/index.html)，在此阶段调用 [js_preread](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_preread) 指令。

## `Content`

  Mandatory phase where data is actually processed, usually [proxied](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html) to [upstream](https://nginx.org/en/docs/stream/ngx_stream_upstream_module.html) servers, or a specified value is [returned](https://nginx.org/en/docs/stream/ngx_stream_return_module.html) to a client. For [njs](https://nginx.org/en/docs/njs/index.html), the [js_filter](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_filter) directive is invoked at this phase.

​	数据实际处理的必要阶段，通常将数据 [代理](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html) 到 [上游](https://nginx.org/en/docs/stream/ngx_stream_upstream_module.html) 服务器，或者将指定的值 [返回](https://nginx.org/en/docs/stream/ngx_stream_return_module.html) 给客户端。对于 [njs](https://nginx.org/en/docs/njs/index.html)，在此阶段调用 [js_filter](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_filter) 指令。

## `Log`

  The final phase where the result of a client session processing is recorded. The [ngx_stream_log_module]({{< ref "/mod_ref/ngx_stream_log_module" >}}) module is invoked at this phase.

​	客户端会话处理结果记录的最终阶段。在此阶段调用 [ngx_stream_log_module]({{< ref "/mod_ref/ngx_stream_log_module" >}}) 模块。