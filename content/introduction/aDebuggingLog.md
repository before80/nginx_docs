+++
title = "调试日志"
date = 2023-08-14T16:51:33+08:00
weight = 80
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# A debugging log - 调试日志

https://nginx.org/en/docs/debugging_log.html



To enable a debugging log, nginx needs to be configured to support debugging during the build:

​	要启用调试日志，需要在构建过程中配置nginx以支持调试：

```
./configure --with-debug ...
```

Then the `debug` level should be set with the [error_log]({{< ref "/mod_ref/ngx_core_module#error_log">}}) directive:

​	然后，应使用 [error_log]({{< ref "/mod_ref/ngx_core_module#error_log">}}) 指令设置 `debug` 级别：

```
error_log /path/to/log debug;
```

To verify that nginx is configured to support debugging, run the `nginx -V` command:

​	为了验证nginx是否配置为支持调试，请运行 `nginx -V` 命令：

```
configure arguments: --with-debug ...
```

Pre-built [Linux](https://nginx.org/en/linux_packages.html) packages provide out-of-the-box support for debugging log with the `nginx-debug` binary (1.9.8) which can be run using commands

​	预构建的 [Linux](https://nginx.org/en/linux_packages.html) 软件包提供了对调试日志的开箱即用支持，使用 `nginx-debug` 二进制文件（1.9.8）可以运行以下命令：

```
service nginx stop
service nginx-debug start
```

and then set the `debug` level. The nginx binary version for Windows is always built with the debugging log support, so only setting the `debug` level will suffice.

然后设置 `debug` 级别。Windows 版本的nginx二进制文件始终构建有调试日志支持，因此只需设置 `debug` 级别即可。

Note that redefining the log without also specifying the `debug` level will disable the debugging log. In the example below, redefining the log on the [server]({{< ref "/mod_ref/ngx_http_core_module#server">}}) level disables the debugging log for this server:

​	请注意，重新定义日志而没有同时指定 `debug` 级别会禁用调试日志。在下面的示例中，重新定义[服务器]({{< ref "/mod_ref/ngx_http_core_module#server">}})级别的日志会禁用此服务器的调试日志：

```
error_log /path/to/log debug;

http {
 server {
     error_log /path/to/log;
     ...
```

To avoid this, either the line redefining the log should be commented out, or the `debug` level specification should also be added:

​	为了避免这种情况，要么注释掉重新定义日志的行，要么同时添加 `debug` 级别的指定：

```
error_log /path/to/log debug;

http {
 server {
     error_log /path/to/log debug;
     ...
```



## 为特定客户端启用调试日志 - Debugging log for selected clients

It is also possible to enable the debugging log for [selected client addresses]({{< ref "/mod_ref/ngx_core_module#debug_connection">}}) only:

​	也可以仅为 [特定客户端地址]({{< ref "/mod_ref/ngx_core_module#debug_connection">}}) 启用调试日志：

```
error_log /path/to/log;

events {
 debug_connection 192.168.1.1;
 debug_connection 192.168.10.0/24;
}
```





## 记录到循环内存缓冲区 - Logging to a cyclic memory buffer

The debugging log can be written to a cyclic memory buffer:

​	调试日志可以写入循环内存缓冲区：

```
error_log memory:32m debug;
```

Logging to the memory buffer on the `debug` level does not have significant impact on performance even under high load. In this case, the log can be extracted using a `gdb` script like the following one:

​	在 `debug` 级别下将日志记录到内存缓冲区不会在高负载下对性能产生显著影响。在这种情况下，可以使用像下面这个 `gdb` 脚本一样的脚本提取日志：

```
set $log = ngx_cycle->log

while $log->writer != ngx_log_memory_writer
 set $log = $log->next
end

set $buf = (ngx_log_memory_buf_t *) $log->wdata
dump binary memory debug_log.txt $buf->start $buf->end
```

Or using an `lldb` script as follows:

​	或者使用 `lldb` 脚本，如下所示：

```
expr ngx_log_t *$log = ngx_cycle->log
expr while ($log->writer != ngx_log_memory_writer) { $log = $log->next; }
expr ngx_log_memory_buf_t *$buf = (ngx_log_memory_buf_t *) $log->wdata
memory read --force --outfile debug_log.txt --binary $buf->start $buf->end
```

