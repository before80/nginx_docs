+++
title = "安装 nginx"
date = 2023-08-14T16:49:55+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Installing nginx - 安装 nginx

https://nginx.org/en/docs/install.html

nginx can be installed differently, depending on the operating system.

​	nginx 的安装方式因操作系统而异。



## 在 Linux 上安装 - Installation on Linux

For Linux, nginx [packages](https://nginx.org/en/linux_packages.html) from nginx.org can be used.

​	对于 Linux，可以使用 nginx.org 上的 [软件包](https://nginx.org/en/linux_packages.html) 进行安装。



## 在 FreeBSD 上安装 - Installation on FreeBSD

On FreeBSD, nginx can be installed either from the [packages](https://docs.freebsd.org/en/books/handbook/ports/#pkgng-intro) or through the [ports](https://docs.freebsd.org/en/books/handbook/ports/#ports-using) system. The ports system provides greater flexibility, allowing selection among a wide range of options. The port will compile nginx with the specified options and install it.

​	在 FreeBSD 上，可以通过 [软件包](https://docs.freebsd.org/en/books/handbook/ports/#pkgng-intro) 或 [ports](https://docs.freebsd.org/en/books/handbook/ports/#ports-using) 系统安装 nginx。ports 系统提供了更大的灵活性，允许在广泛的选项中进行选择。该端口将使用指定的选项编译 nginx 并安装它。

## 从源代码构建 - Building from Sources

If some special functionality is required, not available with packages and ports, nginx can also be compiled from source files. While more flexible, this approach may be complex for a beginner. For more information, see [Building nginx from Sources](https://nginx.org/en/docs/configure.html).

​	如果需要一些在软件包和 ports 中不可用的特殊功能，还可以从源代码编译 nginx。虽然更加灵活，但对于初学者来说，这种方法可能会更加复杂。有关更多信息，请参阅 [从源代码构建 nginx](https://nginx.org/en/docs/configure.html)。