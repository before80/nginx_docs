+++
title = "初学者指南"
date = 2023-08-14T16:50:27+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Beginner’s Guide - 初学者指南

https://nginx.org/en/docs/beginners_guide.html



This guide gives a basic introduction to nginx and describes some simple tasks that can be done with it. It is supposed that nginx is already installed on the reader’s machine. If it is not, see the [Installing nginx](https://nginx.org/en/docs/install.html) page. This guide describes how to start and stop nginx, and reload its configuration, explains the structure of the configuration file and describes how to set up nginx to serve out static content, how to configure nginx as a proxy server, and how to connect it with a FastCGI application.

​	本指南对nginx进行了基本介绍，并描述了一些可以使用它执行的简单任务。假设读者的机器上已经安装了nginx。如果没有安装，可以查看[安装nginx](https://nginx.org/en/docs/install.html)页面。本指南描述了如何启动和停止nginx，并重新加载其配置，解释了配置文件的结构，并描述了如何设置nginx来提供静态内容，如何将nginx配置为代理服务器，以及如何将其与FastCGI应用程序连接起来。

nginx has one master process and several worker processes. The main purpose of the master process is to read and evaluate configuration, and maintain worker processes. Worker processes do actual processing of requests. nginx employs event-based model and OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined in the configuration file and may be fixed for a given configuration or automatically adjusted to the number of available CPU cores (see [worker_processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)).

​	nginx有一个主进程和多个工作进程。主进程的主要目的是读取和评估配置，并维护工作进程。工作进程执行实际的请求处理。nginx采用基于事件的模型和依赖于操作系统的机制，以高效地在工作进程之间分发请求。工作进程的数量在配置文件中定义，可以针对特定配置固定，也可以根据可用CPU核心的数量自动调整（参见[worker_processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)）。

The way nginx and its modules work is determined in the configuration file. By default, the configuration file is named `nginx.conf` and placed in the directory `/usr/local/nginx/conf`, `/etc/nginx`, or `/usr/local/etc/nginx`.

​	nginx及其模块的工作方式是在配置文件中确定的。默认情况下，配置文件的名称为`nginx.conf`，并放置在目录`/usr/local/nginx/conf`、`/etc/nginx`或`/usr/local/etc/nginx`中。



## 启动、停止和重新加载配置 - Starting, Stopping, and Reloading Configuration

To start nginx, run the executable file. Once nginx is started, it can be controlled by invoking the executable with the `-s` parameter. Use the following syntax:

​	要启动nginx，运行可执行文件。一旦nginx启动，就可以通过使用带有`-s`参数调用可执行文件来控制它。使用以下语法：

```
nginx -s signal
```

Where *signal* may be one of the following:

​	其中 *signal* 可以是以下之一：

- `stop` — fast shutdown
- `quit` — graceful shutdown
- `reload` — reloading the configuration file
- `reopen` — reopening the log files
- `stop` — 快速关闭
- `quit` — 优雅关闭
- `reload` — 重新加载配置文件
- `reopen` — 重新打开日志文件

For example, to stop nginx processes with waiting for the worker processes to finish serving current requests, the following command can be executed:

​	例如，要停止nginx进程并等待工作进程完成当前请求服务，可以执行以下命令：

```
nginx -s quit
```

This command should be executed under the same user that started nginx.
​	此命令应在启动nginx的同一用户下执行。

Changes made in the configuration file will not be applied until the command to reload configuration is sent to nginx or it is restarted. To reload configuration, execute:

​	在配置文件中进行的更改只有在发送重新加载配置的命令或重新启动nginx后才会生效。要重新加载配置，执行：

```
nginx -s reload
```

Once the master process receives the signal to reload configuration, it checks the syntax validity of the new configuration file and tries to apply the configuration provided in it. If this is a success, the master process starts new worker processes and sends messages to old worker processes, requesting them to shut down. Otherwise, the master process rolls back the changes and continues to work with the old configuration. Old worker processes, receiving a command to shut down, stop accepting new connections and continue to service current requests until all such requests are serviced. After that, the old worker processes exit.

​	一旦主进程接收到重新加载配置的信号，它会检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果成功，主进程将启动新的工作进程，并向旧的工作进程发送消息，要求它们关闭。否则，主进程会撤销更改，并继续使用旧的配置工作。旧的工作进程在接收到关闭命令后，停止接受新连接，并继续服务当前请求，直到所有此类请求都得到服务为止。然后，旧的工作进程退出。

A signal may also be sent to nginx processes with the help of Unix tools such as the `kill` utility. In this case a signal is sent directly to a process with a given process ID. The process ID of the nginx master process is written, by default, to the `nginx.pid` in the directory `/usr/local/nginx/logs` or `/var/run`. For example, if the master process ID is 1628, to send the QUIT signal resulting in nginx’s graceful shutdown, execute:

​	也可以使用类似于`kill`实用程序的Unix工具向nginx进程发送信号。在这种情况下，信号直接发送给具有给定进程ID的进程。nginx主进程的进程ID默认情况下写入目录`/usr/local/nginx/logs`或`/var/run`中的`nginx.pid`文件。例如，如果主进程ID为1628，要发送QUIT信号以导致nginx优雅关闭，执行：

```
kill -s QUIT 1628
```

For getting the list of all running nginx processes, the `ps` utility may be used, for example, in the following way:

​	要获取所有运行中的nginx进程列表，可以使用`ps`实用程序，例如：

```
ps -ax | grep nginx
```

For more information on sending signals to nginx, see [Controlling nginx](https://nginx.org/en/docs/control.html).

​	有关向nginx发送信号的更多信息，请参见[控制nginx](https://nginx.org/en/docs/control.html)。



## 配置文件的结构 - Configuration File’s Structure

nginx consists of modules which are controlled by directives specified in the configuration file. Directives are divided into simple directives and block directives. A simple directive consists of the name and parameters separated by spaces and ends with a semicolon (`;`). A block directive has the same structure as a simple directive, but instead of the semicolon it ends with a set of additional instructions surrounded by braces (`{` and `}`). If a block directive can have other directives inside braces, it is called a context (examples: [events](https://nginx.org/en/docs/ngx_core_module.html#events), [http](https://nginx.org/en/docs/http/ngx_http_core_module.html#http), [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server), and [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)).

​	nginx由模块组成，这些模块由配置文件中指定的指令控制。指令分为简单指令和块指令。简单指令由名称和由空格分隔的参数组成，并以分号(`;`)结尾。块指令具有与简单指令相同的结构，但是它以一组由大括号(`{`和`}`)包围的附加指令结束，而不是分号。如果块指令内部可以有其他指令，它被称为上下文（示例：[events](https://nginx.org/en/docs/ngx_core_module.html#events)、[http](https://nginx.org/en/docs/http/ngx_http_core_module.html#http)、[server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server)和[location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)）。

Directives placed in the configuration file outside of any contexts are considered to be in the [main](https://nginx.org/en/docs/ngx_core_module.html) context. The `events` and `http` directives reside in the `main` context, `server` in `http`, and `location` in `server`.

​	放置在配置文件中任何上下文之外的指令被认为是在[main](https://nginx.org/en/docs/ngx_core_module.html)上下文中。`events`和`http`指令位于`main`上下文中，`server`位于`http`中，`location`位于`server`中。

The rest of a line after the `#` sign is considered a comment.

​	在`#`号标记后的其余行被认为是注释。



## 提供静态内容 - Serving Static Content

An important web server task is serving out files (such as images or static HTML pages). You will implement an example where, depending on the request, files will be served from different local directories: `/data/www` (which may contain HTML files) and `/data/images` (containing images). This will require editing of the configuration file and setting up of a [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block inside the [http](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) block with two [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) blocks.

​	一个重要的Web服务器任务是提供文件（如图像或静态HTML页面）。您将实现一个示例，根据请求，文件将从不同的本地目录提供：`/data/www`（其中可能包含HTML文件）和`/data/images`（包含图像）。这将需要编辑配置文件，并在[http](https://nginx.org/en/docs/http/ngx_http_core_module.html#http)块中的一个[server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server)块内设置两个[location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)块。

First, create the `/data/www` directory and put an `index.html` file with any text content into it and create the `/data/images` directory and place some images in it.

​	首先，创建`/data/www`目录，并将一个包含任意文本内容的`index.html`文件放入其中，并创建`/data/images`目录，并将一些图像放入其中。

Next, open the configuration file. The default configuration file already includes several examples of the `server` block, mostly commented out. For now comment out all such blocks and start a new `server` block:

​	接下来，打开配置文件。默认配置文件已经包含了几个`server`块的示例，大多数都被注释掉了。现在，注释掉所有这些块，并开始一个新的`server`块：

```
http {
    server {
    }
}
```

Generally, the configuration file may include several `server` blocks [distinguished](https://nginx.org/en/docs/http/request_processing.html) by ports on which they [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) to and by [server names](https://nginx.org/en/docs/http/server_names.html). Once nginx decides which `server` processes a request, it tests the URI specified in the request’s header against the parameters of the `location` directives defined inside the `server` block.

​	通常，配置文件可能包括几个通过它们所监听的端口来[区分](https://nginx.org/en/docs/http/request_processing.html)的`server`块，并通过[server names](https://nginx.org/en/docs/http/server_names.html)来标识。一旦nginx确定了哪个`server`来处理请求，它会将请求头中指定的URI与在`server`块内定义的`location`指令的参数进行比较。

Add the following `location` block to the `server` block:

​	将以下`location`块添加到`server`块中：

```
location / {
    root /data/www;
}
```

This `location` block specifies the “`/`” prefix compared with the URI from the request. For matching requests, the URI will be added to the path specified in the [root](https://nginx.org/en/docs/http/ngx_http_core_module.html#root) directive, that is, to `/data/www`, to form the path to the requested file on the local file system. If there are several matching `location` blocks nginx selects the one with the longest prefix. The `location` block above provides the shortest prefix, of length one, and so only if all other `location` blocks fail to provide a match, this block will be used.

​	此`location`块指定与请求中的URI进行比较的“`/`”前缀。对于匹配的请求，URI将添加到[root](https://nginx.org/en/docs/http/ngx_http_core_module.html#root)指令中指定的路径，即`/data/www`，以形成本地文件系统上所请求文件的路径。如果有多个匹配的`location`块，nginx会选择具有最长前缀的那一个。上面的`location`块提供了最短的前缀，长度为1，因此只有在所有其他`location`块都无法提供匹配时，才会使用此块。

Next, add the second `location` block:

​	接下来，添加第二个`location`块：

```
location /images/ {
    root /data;
}
```

It will be a match for requests starting with `/images/` (`location /` also matches such requests, but has shorter prefix).

​	它将匹配以`/images/`开头的请求（`location /`也会匹配这些请求，但前缀较短）。

The resulting configuration of the `server` block should look like this:

​	`server`块的最终配置应如下所示：

```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```

This is already a working configuration of a server that listens on the standard port 80 and is accessible on the local machine at `http://localhost/`. In response to requests with URIs starting with `/images/`, the server will send files from the `/data/images` directory. For example, in response to the `http://localhost/images/example.png` request nginx will send the `/data/images/example.png` file. If such file does not exist, nginx will send a response indicating the 404 error. Requests with URIs not starting with `/images/` will be mapped onto the `/data/www` directory. For example, in response to the `http://localhost/some/example.html` request nginx will send the `/data/www/some/example.html` file.

​	这已经是一个在标准端口80上监听并且可以通过`http://localhost/`在本地机器上访问的服务器的工作配置。响应以`/images/`开头的请求，服务器将从`/data/images`目录发送文件。例如，响应`http://localhost/images/example.png`请求，nginx将发送`/data/images/example.png`文件。如果此类文件不存在，nginx将发送一个指示404错误的响应。以`/images/`开头的URI请求将被映射到`/data/www`目录。例如，响应`http://localhost/some/example.html`请求，nginx将发送`/data/www/some/example.html`文件。

To apply the new configuration, start nginx if it is not yet started or send the `reload` signal to the nginx’s master process, by executing:

​	要应用新的配置，如果尚未启动nginx，则启动它，或者通过执行以下命令向nginx的主进程发送`reload`信号：

```
nginx -s reload
```

In case something does not work as expected, you may try to find out the reason in `access.log` and `error.log` files in the directory `/usr/local/nginx/logs` or `/var/log/nginx`.
​	如果某些事情不按预期工作，您可以尝试在目录`/usr/local/nginx/logs`或`/var/log/nginx`中查找`access.log`和`error.log`文件中的原因。

## 设置简单代理服务器 - Setting Up a Simple Proxy Server

One of the frequent uses of nginx is setting it up as a proxy server, which means a server that receives requests, passes them to the proxied servers, retrieves responses from them, and sends them to the clients.

​	nginx的一个常见用途是将其设置为代理服务器，即接收请求，将它们传递给代理服务器，从中检索响应，并将其发送给客户端。

We will configure a basic proxy server, which serves requests of images with files from the local directory and sends all other requests to a proxied server. In this example, both servers will be defined on a single nginx instance.

​	我们将配置一个基本的代理服务器，该服务器根据请求从本地目录中提供图像文件，并将所有其他请求发送到代理服务器。在这个示例中，两个服务器都将在单个nginx实例上定义。

First, define the proxied server by adding one more `server` block to the nginx’s configuration file with the following contents:

​	首先，通过在nginx的配置文件中添加另一个`server`块来定义代理服务器，内容如下：

```
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```

This will be a simple server that listens on the port 8080 (previously, the `listen` directive has not been specified since the standard port 80 was used) and maps all requests to the `/data/up1` directory on the local file system. Create this directory and put the `index.html` file into it. Note that the `root` directive is placed in the `server` context. Such `root` directive is used when the `location` block selected for serving a request does not include its own `root` directive.

​	这将是一个简单的服务器，它在端口8080上监听（之前，没有指定`listen`指令，因为使用了标准端口80），并将所有请求映射到本地文件系统上的`/data/up1`目录。创建此目录，并将`index.html`文件放入其中。注意，`root`指令位于`server`上下文中。当选择用于服务请求的`location`块不包含自己的`root`指令时，将使用这样的`root`指令。

Next, use the server configuration from the previous section and modify it to make it a proxy server configuration. In the first `location` block, put the [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive with the protocol, name and port of the proxied server specified in the parameter (in our case, it is `http://localhost:8080`):

​	接下来，使用上一节中的服务器配置，并对其进行修改，使其成为代理服务器配置。在第一个`location`块中，使用[proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)指令，指定参数中指定的协议、名称和代理服务器的端口（在我们的示例中为`http://localhost:8080`）：

```
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```

We will modify the second `location` block, which currently maps requests with the `/images/` prefix to the files under the `/data/images` directory, to make it match the requests of images with typical file extensions. The modified `location` block looks like this:

​	我们将修改第二个`location`块，该块当前将具有`/images/`前缀的请求映射到`/data/images`目录下的文件，以使其与具有典型文件扩展名的图像请求匹配。修改后的`location`块如下：

```
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

The parameter is a regular expression matching all URIs ending with `.gif`, `.jpg`, or `.png`. A regular expression should be preceded with `~`. The corresponding requests will be mapped to the `/data/images` directory.

​	该参数是一个正则表达式，匹配所有以`.gif`、`.jpg`或`.png`结尾的URI。正则表达式应在前面加上`~`。相应的请求将映射到`/data/images`目录。

When nginx selects a `location` block to serve a request it first checks [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) directives that specify prefixes, remembering `location` with the longest prefix, and then checks regular expressions. If there is a match with a regular expression, nginx picks this `location` or, otherwise, it picks the one remembered earlier.

​	当nginx选择要处理请求的`location`块时，它首先检查指定前缀的[location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)指令，记住具有最长前缀的`location`，然后检查正则表达式。如果与正则表达式匹配，nginx会选择此`location`，否则它会选择之前记住的那个。

The resulting configuration of a proxy server will look like this:

​	代理服务器的最终配置如下所示：

```
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

This server will filter requests ending with `.gif`, `.jpg`, or `.png` and map them to the `/data/images` directory (by adding URI to the `root` directive’s parameter) and pass all other requests to the proxied server configured above.

​	此服务器将过滤以`.gif`、`.jpg`或`.png`结尾的请求，并将它们映射到`/data/images`目录（通过将URI添加到`root`指令的参数中），并将所有其他请求发送到上面配置的代理服务器。

To apply new configuration, send the `reload` signal to nginx as described in the previous sections.

​	要应用新的配置，请按照前面章节中的描述向nginx发送`reload`信号。

There are many [more](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) directives that may be used to further configure a proxy connection.

​	还有许多[其他](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)指令可用于进一步配置代理连接。



## 设置FastCGI代理 - Setting Up FastCGI Proxying

nginx can be used to route requests to FastCGI servers which run applications built with various frameworks and programming languages such as PHP.

​	nginx可以用于将请求路由到运行使用各种框架和编程语言构建的FastCGI服务器，例如PHP应用。

The most basic nginx configuration to work with a FastCGI server includes using the [fastcgi_pass](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass) directive instead of the `proxy_pass` directive, and [fastcgi_param](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_param) directives to set parameters passed to a FastCGI server. Suppose the FastCGI server is accessible on `localhost:9000`. Taking the proxy configuration from the previous section as a basis, replace the `proxy_pass` directive with the `fastcgi_pass` directive and change the parameter to `localhost:9000`. In PHP, the `SCRIPT_FILENAME` parameter is used for determining the script name, and the `QUERY_STRING` parameter is used to pass request parameters. The resulting configuration would be:

​	与FastCGI服务器配合工作的最基本的nginx配置包括使用[fastcgi_pass](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass)指令代替`proxy_pass`指令，以及使用[fastcgi_param](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_param)指令来设置传递给FastCGI服务器的参数。假设FastCGI服务器可在`localhost:9000`上访问。以前一节中的代理配置为基础，将`proxy_pass`指令替换为`fastcgi_pass`指令，并将参数更改为`localhost:9000`。在PHP中，`SCRIPT_FILENAME`参数用于确定脚本名称，`QUERY_STRING`参数用于传递请求参数。结果配置如下所示：

```
server {
    location / {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

This will set up a server that will route all requests except for requests for static images to the proxied server operating on `localhost:9000` through the FastCGI protocol.

​	这将设置一个服务器，它将所有请求路由到在`localhost:9000`上运行的代理服务器，通过FastCGI协议进行通信。