+++
title = "核心功能"
date = 2023-08-15T08:06:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Core functionality - 核心功能



https://nginx.org/en/docs/ngx_core_module.html

## 示例配置 Example Configuration



```
user www www;
worker_processes 2;

error_log /var/log/nginx-error.log info;

events {
    use kqueue;
    worker_connections 2048;
}

...
```





## 指令 Directives

### accept_mutex

  Syntax:`accept_mutex on | off;`

  Default: `accept_mutex off;`

  Context: `events`

If `accept_mutex` is enabled, worker processes will accept new connections by turn. Otherwise, all worker processes will be notified about new connections, and if volume of new connections is low, some of the worker processes may just waste system resources.

​	如果启用了 `accept_mutex`，工作进程将轮流接受新连接。否则，所有工作进程将被通知有关新连接的情况，如果新连接的数量较低，一些工作进程可能会浪费系统资源。

There is no need to enable `accept_mutex` on systems that support the [EPOLLEXCLUSIVE](https://nginx.org/en/docs/events.html#epoll) flag (1.11.3) or when using [reuseport](https://nginx.org/en/docs/http/ngx_http_core_module.html#reuseport).

​	在支持 [EPOLLEXCLUSIVE](https://nginx.org/en/docs/events.html#epoll) 标志（1.11.3 版本）的系统上或使用 [reuseport](https://nginx.org/en/docs/http/ngx_http_core_module.html#reuseport) 时，无需启用 `accept_mutex`。



Prior to version 1.11.3, the default value was `on`.

​	在版本 1.11.3 之前，默认值为 `on`。



### accept_mutex_delay

  Syntax:  `accept_mutex_delay time;`

  Default: `accept_mutex_delay 500ms;`

  Context: `events`

If [accept_mutex](https://nginx.org/en/docs/ngx_core_module.html#accept_mutex) is enabled, specifies the maximum time during which a worker process will try to restart accepting new connections if another worker process is currently accepting new connections.

​	如果启用了 [accept_mutex](https://nginx.org/en/docs/ngx_core_module.html#accept_mutex)，则指定另一个工作进程正在接受新连接时，工作进程将尝试重新启动接受新连接的最大时间。



### daemon

  Syntax:`daemon on | off;`

  Default: `daemon on;`

  Context: `main`

Determines whether nginx should become a daemon. Mainly used during development.

​	确定 nginx 是否应该成为守护进程。主要在开发过程中使用。



### debug_connection

  Syntax:`debug_connection address | CIDR | unix:;`

  Default: —

  Context: `events`

Enables debugging log for selected client connections. Other connections will use logging level set by the [error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive. Debugged connections are specified by IPv4 or IPv6 (1.3.0, 1.2.1) address or network. A connection may also be specified using a hostname. For connections using UNIX-domain sockets (1.3.0, 1.2.1), debugging log is enabled by the “`unix:`” parameter.

​	为选定的客户端连接启用调试日志。其他连接将使用由 [error_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) 指令设置的日志级别。调试的连接由 IPv4 或 IPv6（1.3.0、1.2.1 版本）地址或网络指定。也可以使用主机名来指定连接。对于使用 UNIX 域套接字（1.3.0、1.2.1 版本）的连接，可以通过 "unix:" 参数来启用调试日志。

```
events {
    debug_connection 127.0.0.1;
    debug_connection localhost;
    debug_connection 192.0.2.0/24;
    debug_connection ::1;
    debug_connection 2001:0db8::/32;
    debug_connection unix:;
    ...
}
```



For this directive to work, nginx needs to be built with `--with-debug`, see “[A debugging log](https://nginx.org/en/docs/debugging_log.html)”.

​	为了使此指令正常工作，nginx 需要使用 `--with-debug` 编译，详见 "[调试日志](https://nginx.org/en/docs/debugging_log.html)"。



### debug_points

  Syntax:`debug_points abort | stop;`

  Default: —

  Context: `main`

This directive is used for debugging.

​	此指令用于调试。

When internal error is detected, e.g. the leak of sockets on restart of working processes, enabling `debug_points` leads to a core file creation (`abort`) or to stopping of a process (`stop`) for further analysis using a system debugger.

​	在检测到内部错误时，例如工作进程重新启动时的套接字泄漏，启用 `debug_points` 将导致创建核心文件 (`abort`) 或停止进程 (`stop`)，以便使用系统调试器进行进一步分析。



### env

  Syntax:`env variable[=value];`

  Default: `env TZ;`

  Context: `main`

By default, nginx removes all environment variables inherited from its parent process except the TZ variable. This directive allows preserving some of the inherited variables, changing their values, or creating new environment variables. These variables are then:

​	默认情况下，nginx 会删除从其父进程继承的所有环境变量，除了 TZ 变量。此指令允许保留一些继承的变量、更改它们的值或创建新的环境变量。然后这些变量:

- inherited during a [live upgrade](https://nginx.org/en/docs/control.html#upgrade) of an executable file;
- used by the [ngx_http_perl_module](../ngx_http_perl_module) module;
- used by worker processes. One should bear in mind that controlling system libraries in this way is not always possible as it is common for libraries to check variables only during initialization, well before they can be set using this directive. An exception from this is an above mentioned [live upgrade](https://nginx.org/en/docs/control.html#upgrade) of an executable file.
- 在可执行文件的 [实时升级](https://nginx.org/en/docs/control.html#upgrade) 期间继承；
- 被 [ngx_http_perl_module](../ngx_http_perl_module) 模块使用；
- 被工作进程使用。需要记住的是，以这种方式控制系统库并不总是可能的，因为库通常只在初始化期间检查变量，而这发生在使用此指令设置变量之前。唯一的例外是上述提到的 [实时升级](https://nginx.org/en/docs/control.html#upgrade)。



The TZ variable is always inherited and available to the [ngx_http_perl_module](../ngx_http_perl_module) module, unless it is configured explicitly.

​	TZ 变量始终会被继承，并且对于 [ngx_http_perl_module](../ngx_http_perl_module) 模块始终可用，除非明确配置。

Usage example:

​	用法示例：

```
env MALLOC_OPTIONS;
env PERL5LIB=/data/site/modules;
env OPENSSL_ALLOW_PROXY_CERTS=1;
```

The NGINX environment variable is used internally by nginx and should not be set directly by the user.

​	NGINX 环境变量在 nginx 内部使用，不应由用户直接设置。



### error_log

  Syntax:  `error_log file [level];`

  Default: `error_log logs/error.log error;`

  Context: `main`, `http`, `mail`, `stream`, `server`, `location`

Configures logging. Several logs can be specified on the same configuration level (1.5.2). If on the `main` configuration level writing a log to a file is not explicitly defined, the default file will be used.

​	配置日志记录。可以在同一配置级别上指定多个日志。如果在 `main` 配置级别上没有显式定义将日志写入文件，则将使用默认文件。

The first parameter defines a `file` that will store the log. The special value `stderr` selects the standard error file. Logging to [syslog](https://nginx.org/en/docs/syslog.html) can be configured by specifying the “`syslog:`” prefix. Logging to a [cyclic memory buffer](https://nginx.org/en/docs/debugging_log.html#memory) can be configured by specifying the “`memory:`” prefix and buffer `size`, and is generally used for debugging (1.7.11).

​	第一个参数定义将存储日志的 `file`。特殊值 `stderr` 选择标准错误文件。可以通过指定 “`syslog:`” 前缀来配置记录到 [syslog](https://nginx.org/en/docs/syslog.html)。可以通过指定 “`memory:`” 前缀和缓冲区 `size` 来配置记录到 [循环内存缓冲区](https://nginx.org/en/docs/debugging_log.html#memory)，通常用于调试（1.7.11 版本）。

The second parameter determines the `level` of logging, and can be one of the following: `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, or `emerg`. Log levels above are listed in the order of increasing severity. Setting a certain log level will cause all messages of the specified and more severe log levels to be logged. For example, the default level `error` will cause `error`, `crit`, `alert`, and `emerg` messages to be logged. If this parameter is omitted then `error` is used.

​	第二个参数确定日志的 `level`，可以是以下之一: `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, 或 `emerg`。上面的日志级别按严重性递增的顺序列出。设置特定的日志级别会导致将指定和更严重日志级别的所有消息记录。例如，默认级别 `error` 会导致记录 `error`、`crit`、`alert` 和 `emerg` 消息。如果省略此参数，则使用 `error`。

For `debug` logging to work, nginx needs to be built with `--with-debug`, see “[A debugging log](https://nginx.org/en/docs/debugging_log.html)”.

​	要使 `debug` 记录工作，nginx 需要使用 `--with-debug` 构建，详见 “[调试日志](https://nginx.org/en/docs/debugging_log.html)”。

The directive can be specified on the `stream` level starting from version 1.7.11, and on the `mail` level starting from version 1.9.0.

​	从版本 1.7.11 开始，可以在 `stream` 级别上指定此指令，在版本 1.9.0 开始，可以在 `mail` 级别上指定。



### events

  Syntax:`events { ... }`

  Default: —

  Context: `main`

Provides the configuration file context in which the directives that affect connection processing are specified.

​	提供指令的配置文件上下文，这些指令影响连接处理。



### include

  Syntax:`include file | mask;`

  Default: —

  Context: `any`

Includes another `file`, or files matching the specified `mask`, into configuration. Included files should consist of syntactically correct directives and blocks.

​	将另一个 `file` 或与指定 `mask` 匹配的文件包含到配置中。包含的文件应该由语法正确的指令和块组成。

Usage example:

​	用法示例：

```
include mime.types;
include vhosts/*.conf;
```



### load_module

  Syntax:`load_module file;`

  Default: —

  Context: `main`

This directive appeared in version 1.9.11.

​	此指令出现在版本 1.9.11 中。

Loads a dynamic module.

​	加载一个动态模块。

Example:

​	示例：

```
load_module modules/ngx_mail_module.so;
```





### lock_file

  Syntax:  `lock_file file;`

  Default: `lock_file logs/nginx.lock;`

  Context: `main`

nginx uses the locking mechanism to implement [accept_mutex](https://nginx.org/en/docs/ngx_core_module.html#accept_mutex) and serialize access to shared memory. On most systems the locks are implemented using atomic operations, and this directive is ignored. On other systems the “lock file” mechanism is used. This directive specifies a prefix for the names of lock files.

​	nginx 使用锁定机制来实现 [accept_mutex](https://nginx.org/en/docs/ngx_core_module.html#accept_mutex) 并对共享内存进行序列化访问。在大多数系统上，锁定使用原子操作实现，忽略此指令。在其他系统上，使用“锁文件”机制。此指令指定锁文件名称的前缀。



### master_process

  Syntax:`master_process on | off;`

  Default: `master_process on;`

  Context: `main`

Determines whether worker processes are started. This directive is intended for nginx developers.

​	确定是否启动工作进程。此指令适用于 nginx 开发人员。



### multi_accept

  Syntax:`multi_accept on | off;`

  Default: `multi_accept off;`

  Context: `events`

If `multi_accept` is disabled, a worker process will accept one new connection at a time. Otherwise, a worker process will accept all new connections at a time.

​	如果禁用了 `multi_accept`，一个工作进程将一次接受一个新连接。否则，一个工作进程将一次接受所有新连接。

The directive is ignored if [kqueue](https://nginx.org/en/docs/events.html#kqueue) connection processing method is used, because it reports the number of new connections waiting to be accepted.

​	如果使用 [kqueue](https://nginx.org/en/docs/events.html#kqueue) 连接处理方法，则此指令将被忽略，因为它会报告等待被接受的新连接数量。



### pcre_jit

  Syntax:`pcre_jit on | off;`

  Default: `pcre_jit off;`

  Context: `main`

This directive appeared in version 1.1.12.

​	此指令于版本 1.1.12 中出现。

Enables or disables the use of “just-in-time compilation” (PCRE JIT) for the regular expressions known by the time of configuration parsing.

​	启用或禁用“即时编译”（PCRE JIT）用于配置解析时已知的正则表达式。

PCRE JIT can speed up processing of regular expressions significantly.

​	PCRE JIT 可显著加速正则表达式的处理。

The JIT is available in PCRE libraries starting from version 8.20 built with the `--enable-jit` configuration parameter. When the PCRE library is built with nginx (`--with-pcre=`), the JIT support is enabled via the `--with-pcre-jit` configuration parameter.

​	JIT 在 PCRE 库中从版本 8.20 开始可用，构建时需要使用 `--enable-jit` 配置参数。当使用 nginx 构建 PCRE 库（`--with-pcre=`）时，可以通过 `--with-pcre-jit` 配置参数启用 JIT 支持。





### pid

  Syntax:  `pid file;`

  Default: `pid logs/nginx.pid;`

  Context: `main`

Defines a `file` that will store the process ID of the main process.

​	定义将存储主进程的进程 ID 的 `file`。



### ssl_engine

  Syntax:`ssl_engine device;`

  Default: —

  Context: `main`

Defines the name of the hardware SSL accelerator.

​	定义硬件 SSL 加速器的名称。



### thread_pool

  Syntax:`thread_pool name threads=number [max_queue=number];`

  Default: `thread_pool default threads=32 max_queue=65536;`

  Context: `main`

This directive appeared in version 1.7.11.

​	此指令于版本 1.7.11 中出现。

Defines the `name` and parameters of a thread pool used for multi-threaded reading and sending of files [without blocking](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio) worker processes.

​	定义用于多线程读取和发送文件的线程池的 `name` 和参数，这些线程池在工作进程中进行，而不会阻塞。

The `threads` parameter defines the number of threads in the pool.

​	`threads` 参数定义线程池中的线程数。

In the event that all threads in the pool are busy, a new task will wait in the queue. The `max_queue` parameter limits the number of tasks allowed to be waiting in the queue. By default, up to 65536 tasks can wait in the queue. When the queue overflows, the task is completed with an error.

​	在线程池中的所有线程都忙碌的情况下，新任务将在队列中等待。`max_queue` 参数限制允许在队列中等待的任务数。默认情况下，队列中最多可以等待 65536 个任务。当队列溢出时，任务将以错误完成。



### timer_resolution

  Syntax:`timer_resolution interval;`

  Default: —

  Context: `main`

Reduces timer resolution in worker processes, thus reducing the number of `gettimeofday()` system calls made. By default, `gettimeofday()` is called each time a kernel event is received. With reduced resolution, `gettimeofday()` is only called once per specified `interval`.

​	降低工作进程中的计时器分辨率，从而减少调用 `gettimeofday()` 系统调用的次数。默认情况下，每次接收到内核事件时都会调用 `gettimeofday()`。通过降低分辨率，`gettimeofday()` 每指定的 `interval` 只会调用一次。

Example:

​	示例：

```
timer_resolution 100ms;
```

Internal implementation of the interval depends on the method used:

​	间隔的内部实现取决于所使用的方法：

- the `EVFILT_TIMER` filter if `kqueue` is used;
- `timer_create()` if `eventport` is used;
- `setitimer()` otherwise.
- 使用 `kqueue` 时为 `EVFILT_TIMER` 过滤器；
- 使用 `eventport` 时为 `timer_create()`；
- 否则为 `setitimer()`。





### use

  Syntax:`use method;`

  Default: —

  Context: `events`

Specifies the [connection processing](https://nginx.org/en/docs/events.html) `method` to use. There is normally no need to specify it explicitly, because nginx will by default use the most efficient method.

​	指定要使用的 [连接处理](https://nginx.org/en/docs/events.html) `method`。通常不需要显式指定，因为 nginx 默认会使用最有效的方法。



### user

  Syntax:  `user user [group];`

  Default: `user nobody nobody;`

  Context: `main`

Defines `user` and `group` credentials used by worker processes. If `group` is omitted, a group whose name equals that of `user` is used.

​	定义工作进程使用的 `user` 和 `group` 凭证。如果省略 `group`，则使用与 `user` 名称相同的组。



### worker_aio_requests

  Syntax:`worker_aio_requests number;`

  Default: `worker_aio_requests 32;`

  Context: `events`

This directive appeared in versions 1.1.4 and 1.0.7.

​	此指令于版本 1.1.4 和 1.0.7 中出现。

When using [aio](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio) with the [epoll](https://nginx.org/en/docs/events.html#epoll) connection processing method, sets the maximum `number` of outstanding asynchronous I/O operations for a single worker process.

​	在使用 [aio](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio) 和 [epoll](https://nginx.org/en/docs/events.html#epoll) 连接处理方法时，设置单个工作进程的最大异步 I/O 操作数。

### worker_connections

  Syntax:`worker_connections number;`

  Default: `worker_connections 512;`

  Context: `events`

Sets the maximum number of simultaneous connections that can be opened by a worker process.

​	设置工作进程可以打开的最大同时连接数。

It should be kept in mind that this number includes all connections (e.g. connections with proxied servers, among others), not only connections with clients. Another consideration is that the actual number of simultaneous connections cannot exceed the current limit on the maximum number of open files, which can be changed by [worker_rlimit_nofile](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile).

​	需要记住，此数字包括所有连接（例如，与代理服务器的连接等），不仅限于与客户端的连接。另一个考虑因素是实际的同时连接数不能超过最大打开文件数的当前限制，可以通过 [worker_rlimit_nofile](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile) 进行更改。



### worker_cpu_affinity

  Syntax:`worker_cpu_affinity cpumask ...;` `worker_cpu_affinity auto [cpumask];`

  Default: —

  Context: `main`

Binds worker processes to the sets of CPUs. Each CPU set is represented by a bitmask of allowed CPUs. There should be a separate set defined for each of the worker processes. By default, worker processes are not bound to any specific CPUs.

​	将工作进程绑定到 CPU 集。每个 CPU 集由允许的 CPU 的位掩码表示。每个工作进程应定义一个单独的 CPU 集。默认情况下，工作进程未绑定到任何特定的 CPU。

For example,

​	例如，

```
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;
```

binds each worker process to a separate CPU, while

将每个工作进程绑定到单独的 CPU，而

```
worker_processes    2;
worker_cpu_affinity 0101 1010;
```

binds the first worker process to CPU0/CPU2, and the second worker process to CPU1/CPU3. The second example is suitable for hyper-threading.

将第一个工作进程绑定到 CPU0/CPU2，将第二个工作进程绑定到 CPU1/CPU3。第二个示例适用于超线程。

The special value `auto` (1.9.10) allows binding worker processes automatically to available CPUs:

​	特殊值 `auto` (1.9.10) 允许将工作进程自动绑定到可用的 CPU：

```
worker_processes auto;
worker_cpu_affinity auto;
```

The optional mask parameter can be used to limit the CPUs available for automatic binding:

​	可以使用可选的掩码参数来限制用于自动绑定的 CPU：

```
worker_cpu_affinity auto 01010101;
```

The directive is only available on FreeBSD and Linux.

​	此指令仅在 FreeBSD 和 Linux 上可用。

### worker_priority

  Syntax:`worker_priority number;`

  Default: `worker_priority 0;`

  Context: `main`

Defines the scheduling priority for worker processes like it is done by the `nice` command: a negative `number` means higher priority. Allowed range normally varies from -20 to 20.

​	为工作进程定义调度优先级，类似于 `nice` 命令的操作：负数的 `number` 表示更高的优先级。允许的范围通常为 -20 到 20。

Example:

​	示例：

```
worker_priority -10;
```





### worker_processes

  Syntax:`worker_processes number | auto;`

  Default: `worker_processes 1;`

  Context: `main`

Defines the number of worker processes.

​	定义工作进程的数量。

The optimal value depends on many factors including (but not limited to) the number of CPU cores, the number of hard disk drives that store data, and load pattern. When one is in doubt, setting it to the number of available CPU cores would be a good start (the value “`auto`” will try to autodetect it).

​	最佳值取决于许多因素，包括（但不限于）CPU 核心数、存储数据的硬盘驱动器数和负载模式。如果不确定，将其设置为可用的 CPU 核心数可能是一个好的起点（值“`auto`”将尝试自动检测）。

The `auto` parameter is supported starting from versions 1.3.8 and 1.2.5.

​	`auto` 参数支持从版本 1.3.8 和 1.2.5 开始。



### worker_rlimit_core

  Syntax:`worker_rlimit_core size;`

  Default: —

  Context: `main`


Changes the limit on the largest size of a core file (`RLIMIT_CORE`) for worker processes. Used to increase the limit without restarting the main process.

​	更改工作进程的核心文件最大大小限制（`RLIMIT_CORE`）。用于在不重新启动主进程的情况下增加限制。

### worker_rlimit_nofile

  Syntax:`worker_rlimit_nofile number;`

  Default: —

  Context: `main`


Changes the limit on the maximum number of open files (`RLIMIT_NOFILE`) for worker processes. Used to increase the limit without restarting the main process.

​	更改工作进程的最大打开文件数限制（`RLIMIT_NOFILE`）。用于在不重新启动主进程的情况下增加限制。

### worker_shutdown_timeout

  Syntax:`worker_shutdown_timeout time;`

  Default: —

  Context: `main`

This directive appeared in version 1.11.11.

​	此指令于版本 1.11.11 中出现。

Configures a timeout for a graceful shutdown of worker processes. When the `time` expires, nginx will try to close all the connections currently open to facilitate shutdown.

​	为工作进程的优雅关闭配置超时。当超时结束时，nginx 将尝试关闭所有当前打开的连接以便于关闭。

### working_directory

  Syntax:`working_directory directory;`

  Default: —

  Context: `main`

Defines the current working directory for a worker process. It is primarily used when writing a core-file, in which case a worker process should have write permission for the specified directory.

​	为工作进程定义当前工作目录。主要在编写核心文件时使用，此时工作进程应具有指定目录的写权限。