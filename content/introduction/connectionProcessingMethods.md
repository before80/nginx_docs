+++
title = "连接处理方法"
date = 2023-08-14T16:50:50+08:00
weight = 60
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Connection processing methods - 连接处理方法

https://nginx.org/en/docs/events.html

nginx supports a variety of connection processing methods. The availability of a particular method depends on the platform used. On platforms that support several methods nginx will normally select the most efficient method automatically. However, if needed, a connection processing method can be selected explicitly with the [use]({{< ref "/mod_ref/ngx_core_module#use">}}) directive.

​	nginx支持多种连接处理方法。特定方法的可用性取决于所使用的平台。在支持多种方法的平台上，nginx通常会自动选择最有效的方法。但是，如果需要的话，可以使用 [use]({{< ref "/mod_ref/ngx_core_module#use">}}) 指令显式地选择连接处理方法。

The following connection processing methods are supported:

​	以下连接处理方法受到支持：

- `select` — standard method. The supporting module is built automatically on platforms that lack more efficient methods. The `--with-select_module` and `--without-select_module` configuration parameters can be used to forcibly enable or disable the build of this module.

- `select` — 标准方法。在缺乏更有效方法的平台上，支持模块会自动构建。可以使用 `--with-select_module` 和 `--without-select_module` 配置参数来强制启用或禁用构建该模块。

- `poll` — standard method. The supporting module is built automatically on platforms that lack more efficient methods. The `--with-poll_module` and `--without-poll_module` configuration parameters can be used to forcibly enable or disable the build of this module.

- `poll` — 标准方法。在缺乏更有效方法的平台上，支持模块会自动构建。可以使用 `--with-poll_module` 和 `--without-poll_module` 配置参数来强制启用或禁用构建该模块。

- `kqueue` — efficient method used on FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0, and macOS.

- `kqueue` — 在FreeBSD 4.1+、OpenBSD 2.9+、NetBSD 2.0和macOS上使用的高效方法。

- `epoll` — efficient method used on Linux 2.6+.

- `epoll` — 在Linux 2.6+上使用的高效方法。

  > The `EPOLLRDHUP` (Linux 2.6.17, glibc 2.8) and `EPOLLEXCLUSIVE` (Linux 4.5, glibc 2.24) flags are supported since 1.11.3.
  >
  > 自1.11.3版本起，支持 `EPOLLRDHUP` (Linux 2.6.17, glibc 2.8) 和 `EPOLLEXCLUSIVE` (Linux 4.5, glibc 2.24) 标志。

  > Some older distributions like SuSE 8.2 provide patches that add epoll support to 2.4 kernels.
  >
  > 一些旧版本的发行版，如SuSE 8.2，提供了将epoll支持添加到2.4内核的补丁。

  

- `/dev/poll` — efficient method used on Solaris 7 11/99+, HP/UX 11.22+ (eventport), IRIX 6.5.15+, and Tru64 UNIX 5.1A+.

- `/dev/poll` — 在Solaris 7 11/99+、HP/UX 11.22+ (eventport)、IRIX 6.5.15+和Tru64 UNIX 5.1A+上使用的高效方法。

- `eventport` — event ports, method used on Solaris 10+ (due to known issues, it is recommended using the `/dev/poll` method instead).

- `eventport` — 在Solaris 10+上使用的事件端口方法（由于已知问题，建议使用 `/dev/poll` 方法代替）。