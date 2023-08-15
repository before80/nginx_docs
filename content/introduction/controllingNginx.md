+++
title = "控制 nginx"
date = 2023-08-14T16:50:38+08:00
weight = 50
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Controlling nginx - 控制 nginx

https://nginx.org/en/docs/control.html



nginx can be controlled with signals. The process ID of the master process is written to the file `/usr/local/nginx/logs/nginx.pid` by default. This name may be changed at configuration time, or in `nginx.conf` using the [pid](https://nginx.org/en/docs/ngx_core_module.html#pid) directive. The master process supports the following signals:

​	nginx可以通过信号进行控制。默认情况下，主进程的进程ID会写入文件 `/usr/local/nginx/logs/nginx.pid`。在配置时可以更改这个名称，或者在 `nginx.conf` 中使用 [pid](https://nginx.org/en/docs/ngx_core_module.html#pid) 指令进行更改。主进程支持以下信号：

| TERM, INT | fast shutdown 快速关闭                                       |
| --------- | ------------------------------------------------------------ |
| QUIT      | graceful shutdown 优雅关闭                                   |
| HUP       | changing configuration, keeping up with a changed time zone (only for FreeBSD and Linux), starting new worker processes with a new configuration, graceful shutdown of old worker processes  更改配置，跟踪更改的时区（仅适用于FreeBSD和Linux），使用新配置启动新的工作进程，优雅关闭旧的工作进程 |
| USR1      | re-opening log files 重新打开日志文件                        |
| USR2      | upgrading an executable file 更新可执行文件                  |
| WINCH     | graceful shutdown of worker processes 优雅关闭工作进程       |



Individual worker processes can be controlled with signals as well, though it is not required. The supported signals are:

​	每个独立的工作进程也可以通过信号进行控制，尽管这不是必需的。支持的信号有：

| TERM, INT | fast shutdown 快速关闭                                       |
| --------- | ------------------------------------------------------------ |
| QUIT      | graceful shutdown 优雅关闭                                   |
| USR1      | re-opening log files 重新打开日志文件                        |
| WINCH     | abnormal termination for debugging (requires [debug_points](https://nginx.org/en/docs/ngx_core_module.html#debug_points) to be enabled) 异常终止以进行调试（需要启用 [debug_points](https://nginx.org/en/docs/ngx_core_module.html#debug_points)） |

## 更改配置 - Changing Configuration

In order for nginx to re-read the configuration file, a HUP signal should be sent to the master process. The master process first checks the syntax validity, then tries to apply new configuration, that is, to open log files and new listen sockets. If this fails, it rolls back changes and continues to work with old configuration. If this succeeds, it starts new worker processes, and sends messages to old worker processes requesting them to shut down gracefully. Old worker processes close listen sockets and continue to service old clients. After all clients are serviced, old worker processes are shut down.

​	为了使nginx重新读取配置文件，应向主进程发送HUP信号。主进程首先检查语法的有效性，然后尝试应用新的配置，也就是说，打开日志文件和新的监听套接字。如果失败，它会回滚更改并继续使用旧配置工作。如果成功，它会启动新的工作进程，并向旧的工作进程发送消息，要求它们优雅地关闭。旧的工作进程关闭监听套接字，并继续为旧客户端提供服务。在所有客户端得到服务之后，旧的工作进程会被关闭。

Let’s illustrate this by example. Imagine that nginx is run on FreeBSD and the command

​	我们用一个示例来说明这个过程。假设在FreeBSD上运行nginx，并且执行以下命令：

```
ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | egrep '(nginx|PID)'
```

produces the following output:

将产生以下输出：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
33127 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33128 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
33129 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
```



If HUP is sent to the master process, the output becomes:

​	如果向主进程发送HUP信号，输出将变为：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33129 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```



One of the old worker processes with PID 33129 still continues to work. After some time it exits:

​	其中的旧工作进程PID 33129 仍在继续工作。过一段时间后，它退出：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```



## 轮转日志文件 - Rotating Log-files

In order to rotate log files, they need to be renamed first. After that USR1 signal should be sent to the master process. The master process will then re-open all currently open log files and assign them an unprivileged user under which the worker processes are running, as an owner. After successful re-opening, the master process closes all open files and sends the message to worker process to ask them to re-open files. Worker processes also open new files and close old files right away. As a result, old files are almost immediately available for post processing, such as compression.

​	要进行日志文件的轮转，首先需要对它们进行重命名。然后应向主进程发送USR1信号。主进程将重新打开当前打开的所有日志文件，并将它们分配给正在运行的工作进程所在的非特权用户作为所有者。成功重新打开后，主进程关闭所有打开的文件，并向工作进程发送消息，要求它们重新打开文件。工作进程也会立即打开新文件并关闭旧文件。因此，旧文件几乎立即可以用于后续处理，如压缩等。



## 在线升级可执行文件 - Upgrading Executable on the Fly

In order to upgrade the server executable, the new executable file should be put in place of an old file first. After that USR2 signal should be sent to the master process. The master process first renames its file with the process ID to a new file with the `.oldbin` suffix, e.g. `/usr/local/nginx/logs/nginx.pid.oldbin`, then starts a new executable file that in turn starts new worker processes:

​	为了升级服务器可执行文件，首先需要将新的可执行文件放置在旧文件的位置。然后应向主进程发送USR2信号。主进程首先将其带有进程ID的文件重命名为带有 `.oldbin` 后缀的新文件，例如 `/usr/local/nginx/logs/nginx.pid.oldbin`，然后启动一个新的可执行文件，然后新的可执行文件会启动新的工作进程：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```



After that all worker processes (old and new ones) continue to accept requests. If the WINCH signal is sent to the first master process, it will send messages to its worker processes, requesting them to shut down gracefully, and they will start to exit:

​	之后，所有工作进程（包括旧的和新的）将继续接受请求。如果将WINCH信号发送给第一个主进程，它将向其工作进程发送消息，要求它们优雅地关闭，它们将开始退出：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33135 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

After some time, only the new worker processes will process requests:

​	过一段时间后，只有新的工作进程将处理请求：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

It should be noted that the old master process does not close its listen sockets, and it can be managed to start its worker processes again if needed. If for some reason the new executable file works unacceptably, one of the following can be done:

​	需要注意的是，旧的主进程不会关闭其监听套接字，如果需要的话，可以管理它以再次启动其工作进程。如果由于某种原因新的可执行文件工作得不可接受，可以执行以下操作之一：

- Send the HUP signal to the old master process. The old master process will start new worker processes without re-reading the configuration. After that, all new processes can be shut down gracefully, by sending the QUIT signal to the new master process.
- 向旧的主进程发送HUP信号。旧的主进程将在不重新读取配置的情况下启动新的工作进程。之后，通过向新的主进程发送QUIT信号，所有新进程可以优雅地关闭。
- Send the TERM signal to the new master process. It will then send a message to its worker processes requesting them to exit immediately, and they will all exit almost immediately. (If new processes do not exit for some reason, the KILL signal should be sent to them to force them to exit.) When the new master process exits, the old master process will start new worker processes automatically.
- 向新的主进程发送TERM信号。然后它会向其工作进程发送消息，请求它们立即退出，它们将几乎立即退出。（如果由于某种原因新进程不退出，则应向它们发送KILL信号以强制它们退出。）当新的主进程退出时，旧的主进程将自动启动新的工作进程。

If the new master process exits then the old master process discards the `.oldbin` suffix from the file name with the process ID.

​	如果新的主进程退出，则旧的主进程会从带有进程ID的文件名中删除 `.oldbin` 后缀。

If upgrade was successful, then the QUIT signal should be sent to the old master process, and only new processes will stay:

​	如果升级成功，则应向旧的主进程发送QUIT信号，只有新的进程将保留：

```
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
36264     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

