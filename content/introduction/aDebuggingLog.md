+++
title = "aDebuggingLog"
date = 2023-08-14T16:51:33+08:00
weight = 80
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# A debugging log

https://nginx.org/en/docs/debugging_log.html



To enable a debugging log, nginx needs to be configured to support debugging during the build:

> ```
> ./configure --with-debug ...
> ```

Then the `debug` level should be set with the [error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive:

> ```
> error_log /path/to/log debug;
> ```

To verify that nginx is configured to support debugging, run the `nginx -V` command:

> ```
> configure arguments: --with-debug ...
> ```

Pre-built [Linux](https://nginx.org/en/linux_packages.html) packages provide out-of-the-box support for debugging log with the `nginx-debug` binary (1.9.8) which can be run using commands

> ```
> service nginx stop
> service nginx-debug start
> ```

and then set the `debug` level. The nginx binary version for Windows is always built with the debugging log support, so only setting the `debug` level will suffice.

Note that redefining the log without also specifying the `debug` level will disable the debugging log. In the example below, redefining the log on the [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) level disables the debugging log for this server:

> ```
> error_log /path/to/log debug;
> 
> http {
>     server {
>         error_log /path/to/log;
>         ...
> ```

To avoid this, either the line redefining the log should be commented out, or the `debug` level specification should also be added:

> ```
> error_log /path/to/log debug;
> 
> http {
>     server {
>         error_log /path/to/log debug;
>         ...
> ```



## Debugging log for selected clients

It is also possible to enable the debugging log for [selected client addresses](https://nginx.org/en/docs/ngx_core_module.html#debug_connection) only:

> ```
> error_log /path/to/log;
> 
> events {
>     debug_connection 192.168.1.1;
>     debug_connection 192.168.10.0/24;
> }
> ```





## Logging to a cyclic memory buffer

The debugging log can be written to a cyclic memory buffer:

> ```
> error_log memory:32m debug;
> ```

Logging to the memory buffer on the `debug` level does not have significant impact on performance even under high load. In this case, the log can be extracted using a `gdb` script like the following one:

> ```
> set $log = ngx_cycle->log
> 
> while $log->writer != ngx_log_memory_writer
>     set $log = $log->next
> end
> 
> set $buf = (ngx_log_memory_buf_t *) $log->wdata
> dump binary memory debug_log.txt $buf->start $buf->end
> ```

Or using an `lldb` script as follows:

> ```
> expr ngx_log_t *$log = ngx_cycle->log
> expr while ($log->writer != ngx_log_memory_writer) { $log = $log->next; }
> expr ngx_log_memory_buf_t *$buf = (ngx_log_memory_buf_t *) $log->wdata
> memory read --force --outfile debug_log.txt --binary $buf->start $buf->end
> ```