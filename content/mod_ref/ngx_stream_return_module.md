+++
title = "ngx_stream_return_module"
date = 2023-08-15T08:24:08+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_return_module

https://nginx.org/en/docs/stream/ngx_stream_return_module.html



The `ngx_stream_return_module` module (1.11.2) allows sending a specified value to the client and then closing the connection.



## Example Configuration



> ```
> server {
>     listen 12345;
>     return $time_iso8601;
> }
> ```





## Directives



### return

| Syntax:  | `return value;` |
| :------- | --------------- |
| Default: | —               |
| Context: | `server`        |

Specifies a `value` to send to the client. The value can contain text, variables, and their combination.