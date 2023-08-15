+++
title = "适用于 Windows 的 nginx"
date = 2023-08-14T16:53:44+08:00
weight = 120
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# nginx for Windows - 适用于 Windows 的 nginx

https://nginx.org/en/docs/windows.html



Version of nginx for Windows uses the native Win32 API (not the Cygwin emulation layer). Only the `select()` and `poll()` (1.15.9) connection processing methods are currently used, so high performance and scalability should not be expected. Due to this and some other known issues version of nginx for Windows is considered to be a *beta* version. At this time, it provides almost the same functionality as a UNIX version of nginx except for XSLT filter, image filter, GeoIP module, and embedded Perl language.

​	适用于 Windows 版本的 nginx 使用本机的 Win32 API（而不是 Cygwin 模拟层）。目前仅使用了 `select()` 和 `poll()`（1.15.9）连接处理方法，因此不应期望高性能和可扩展性。由于这个原因以及其他一些已知问题，适用于 Windows 的 nginx 版本被视为 *测试版*。目前，除了 XSLT 过滤器、图像过滤器、GeoIP 模块和嵌入式 Perl 语言外，它几乎提供了与 UNIX 版本的 nginx 相同的功能。

To install nginx/Windows, [download](https://nginx.org/en/download.html) the latest mainline version distribution (1.25.1), since the mainline branch of nginx contains all known fixes. Then unpack the distribution, go to the nginx-1.25.1 directory, and run `nginx`. Here is an example for the drive C: root directory:

​	要安装 nginx/Windows，[下载](https://nginx.org/en/download.html) 最新的主线版本分发（1.25.1），因为 nginx 的主线分支包含了所有已知的修复。然后解压分发包，进入 nginx-1.25.1 目录，并运行 `nginx`。以下是在 C: 根目录下的示例：

```
cd c:\
unzip nginx-1.25.1.zip
cd nginx-1.25.1
start nginx
```

Run the `tasklist` command-line utility to see nginx processes:

​	使用 `tasklist` 命令行工具查看 nginx 进程：

```
C:\nginx-1.25.1>tasklist /fi "imagename eq nginx.exe"

Image Name           PID Session Name     Session#    Mem Usage
=============== ======== ============== ========== ============
nginx.exe            652 Console                 0      2 780 K
nginx.exe           1332 Console                 0      3 112 K
```

One of the processes is the master process and another is the worker process. If nginx does not start, look for the reason in the error log file `logs\error.log`. If the log file has not been created, the reason for this should be reported in the Windows Event Log. If an error page is displayed instead of the expected page, also look for the reason in the `logs\error.log` file.

​	其中一个进程是主进程，另一个是工作进程。如果 nginx 未能启动，请在错误日志文件 `logs\error.log` 中查找原因。如果日志文件尚未创建，应在 Windows 事件日志中报告此问题。如果显示错误页面而不是预期页面，请在 `logs\error.log` 文件中查找原因。

nginx/Windows uses the directory where it has been run as the prefix for relative paths in the configuration. In the example above, the prefix is `C:\nginx-1.25.1\`. Paths in a configuration file must be specified in UNIX-style using forward slashes:

​	nginx/Windows 使用运行的目录作为配置中相对路径的前缀。在上面的示例中，前缀是 `C:\nginx-1.25.1\`。配置文件中的路径必须使用正斜杠以 UNIX 风格指定：

```
access_log   logs/site.log;
root         C:/web/html;
```

nginx/Windows runs as a standard console application (not a service), and it can be managed using the following commands:

​	nginx/Windows 以标准控制台应用程序（而不是服务）运行，可以使用以下命令进行管理：

| nginx -s stop   | fast shutdown 快速关闭                                       |
| --------------- | ------------------------------------------------------------ |
| nginx -s quit   | graceful shutdown 优雅关闭                                   |
| nginx -s reload | changing configuration, starting new worker processes with a new configuration, graceful shutdown of old worker processes 更改配置，使用新配置启动新的工作进程，优雅关闭旧的工作进程 |
| nginx -s reopen | re-opening log files 重新打开日志文件                        |



## 已知问题 Known issues

- Although several workers can be started, only one of them actually does any work.
- The UDP proxy functionality is not supported.
- 尽管可以启动多个 worker 进程，但只有其中一个实际上会执行任何工作。
- 不支持 UDP 代理功能。



## 可能的未来增强功能 - Possible future enhancements

- Running as a service.
- Using the I/O completion ports as a connection processing method.
- Using multiple worker threads inside a single worker process.
- 作为服务运行。
- 使用 I/O 完成端口作为连接处理方法。
- 在单个 worker 进程内使用多个工作线程。