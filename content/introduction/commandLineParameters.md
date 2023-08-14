+++
title = "命令行参数"
date = 2023-08-14T16:52:55+08:00
weight = 110
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Command-line parameters - 命令行参数

https://nginx.org/en/docs/switches.html

nginx supports the following command-line parameters:

​	nginx 支持以下命令行参数：

- `-?` | `-h` — print help for command-line parameters.

- `-?` | `-h` — 打印命令行参数的帮助信息。

- `-c file` — use an alternative configuration `file` instead of a default file.

- `-c file` — 使用替代配置文件 `file`，而不是默认的文件。

- `-e file` — use an alternative error log `file` to store the log instead of a default file (1.19.5). The special value `stderr` selects the standard error file.

- `-e file` — 使用替代的错误日志文件 `file` 来存储日志，而不是默认的文件（1.19.5）。特殊值 `stderr` 选择标准错误文件。

- `-g directives` — set global configuration directives, for example,

- `-g directives` — 设置全局配置指令，例如：

  ```
  nginx -g "pid /var/run/nginx.pid; worker_processes `sysctl -n hw.ncpu`;"
  ```

  

- `-p prefix` — set nginx path prefix, i.e. a directory that will keep server files (default value is `/usr/local/nginx`).

- `-p prefix` — 设置 nginx 路径前缀，即保存服务器文件的目录（默认值为 `/usr/local/nginx`）。

- `-q` — suppress non-error messages during configuration testing.

- `-q` — 在配置测试期间禁止非错误消息的输出。

- `-s signal` — send a signal to the master process. The argument signal can be one of:

- `-s signal` — 向主进程发送信号。参数 signal 可以是以下之一：

  - `stop` — shut down quickly
  - `stop` — 快速关闭
  - `quit` — shut down gracefully
  - `quit` — 优雅关闭
  - `reload` — reload configuration, start the new worker process with a new configuration, gracefully shut down old worker processes.
  - `reload` — 重新加载配置，使用新配置启动新的工作进程，优雅关闭旧的工作进程。
  - `reopen` — reopen log files
  - `reopen` — 重新打开日志文件

- `-t` — test the configuration file: nginx checks the configuration for correct syntax, and then tries to open files referred in the configuration.

- `-t` — 测试配置文件：nginx 检查配置的正确语法，然后尝试打开配置中引用的文件。

- `-T` — same as `-t`, but additionally dump configuration files to standard output (1.9.2).

- `-T` — 与 `-t` 相同，但额外将配置文件输出到标准输出（1.9.2）。

- `-v` — print nginx version.

- `-v` — 打印 nginx 版本。

- `-V` — print nginx version, compiler version, and configure parameters.

- `-V` — 打印 nginx 版本、编译器版本和配置参数。