+++
title = "ngx_http_core_module"
date = 2023-08-15T08:11:25+08:00
weight = 2
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_core_module

https://nginx.org/en/docs/http/ngx_http_core_module.html

## Directives 指令

### absolute_redirect

Syntax:`absolute_redirect on | off;`

​	​语法： `absolute_redirect on | off;`

Default: `absolute_redirect on;`

​	​默认值： `absolute_redirect on;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 1.11.8.

​	​此指令出现在 1.11.8 版本中。

If disabled, redirects issued by nginx will be relative.

​	​如果禁用，nginx 发出的重定向将是相对的。

See also [server_name_in_redirect](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server_name_in_redirect) and [port_in_redirect](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#port_in_redirect) directives.

​	​另请参阅 server_name_in_redirect 和 port_in_redirect 指令。

### aio

Syntax:`aio on | off | threads[=pool];`

​	​语法： `aio on | off | threads[=pool];`

Default: `aio off;`

​	​默认值： `aio off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 0.8.11.

​	​此指令出现在 0.8.11 版本中。

Enables or disables the use of asynchronous file I/O (AIO) on FreeBSD and Linux:

​	​在 FreeBSD 和 Linux 上启用或禁用异步文件 I/O (AIO) 的使用：

```
location /video/ {
    aio            on;
    output_buffers 1 64k;
}
```

On FreeBSD, AIO can be used starting from FreeBSD 4.3. Prior to FreeBSD 11.0, AIO can either be linked statically into a kernel:

​	​在 FreeBSD 上，从 FreeBSD 4.3 开始可以使用 AIO。在 FreeBSD 11.0 之前，AIO 可以静态链接到内核中：

```
options VFS_AIO
```

or loaded dynamically as a kernel loadable module:

​	​或者作为内核可加载模块动态加载：

```
kldload aio
```

On Linux, AIO can be used starting from kernel version 2.6.22. Also, it is necessary to enable [directio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#directio), or otherwise reading will be blocking:

​	​在 Linux 上，从内核版本 2.6.22 开始可以使用 AIO。此外，有必要启用 directio，否则读取将被阻塞：

```
location /video/ {
    aio            on;
    directio       512;
    output_buffers 1 128k;
}
```

On Linux, [directio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#directio) can only be used for reading blocks that are aligned on 512-byte boundaries (or 4K for XFS). File’s unaligned end is read in blocking mode. The same holds true for byte range requests and for FLV requests not from the beginning of a file: reading of unaligned data at the beginning and end of a file will be blocking.

​	​在 Linux 上，directio 只能用于读取对齐在 512 字节边界（或 XFS 的 4K）上的块。文件的未对齐结尾以阻塞模式读取。对于字节范围请求和不是从文件开头开始的 FLV 请求也是如此：在文件开头和结尾读取未对齐的数据将被阻塞。

When both AIO and [sendfile](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#sendfile) are enabled on Linux, AIO is used for files that are larger than or equal to the size specified in the [directio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#directio) directive, while [sendfile](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#sendfile) is used for files of smaller sizes or when [directio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#directio) is disabled.

​	​当在 Linux 上同时启用 AIO 和 sendfile 时，AIO 用于大于或等于 directio 指令中指定大小的文件，而 sendfile 用于较小文件或禁用 directio 时。

```
location /video/ {
    sendfile       on;
    aio            on;
    directio       8m;
}
```

Finally, files can be read and [sent](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#sendfile) using multi-threading (1.7.11), without blocking a worker process:

​	​最后，可以使用多线程（1.7.11）读取和发送文件，而不会阻塞工作进程：

```
location /video/ {
    sendfile       on;
    aio            threads;
}
```

Read and send file operations are offloaded to threads of the specified [pool](https://before80.github.io/nginx_docs/mod_ref/ngx_core_module/#thread_pool). If the pool name is omitted, the pool with the name “`default`” is used. The pool name can also be set with variables:

​	​读写文件操作已卸载到指定池的线程。如果省略池名称，则使用名称为“ `default` ”的池。池名称也可以用变量设置：

```
aio threads=pool$disk;
```

By default, multi-threading is disabled, it should be enabled with the `--with-threads` configuration parameter. Currently, multi-threading is compatible only with the [epoll](https://before80.github.io/nginx_docs/introduction/connectionProcessingMethods/#epoll), [kqueue](https://before80.github.io/nginx_docs/introduction/connectionProcessingMethods/#kqueue), and [eventport](https://before80.github.io/nginx_docs/introduction/connectionProcessingMethods/#eventport) methods. Multi-threaded sending of files is only supported on Linux.

​	​默认情况下，多线程被禁用，它应该使用 `--with-threads` 配置参数启用。目前，多线程仅与 epoll、kqueue 和 eventport 方法兼容。多线程发送文件仅在 Linux 上受支持。

See also the [sendfile](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#sendfile) directive.

​	​另请参阅 sendfile 指令。

### aio_write

Syntax: `aio_write on | off;`

​	​语法： `aio_write on | off;`

Default: `aio_write off;`

​	​默认值： `aio_write off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 1.9.13.

​	​此指令出现在 1.9.13 版中。

If [aio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#aio) is enabled, specifies whether it is used for writing files. Currently, this only works when using `aio threads` and is limited to writing temporary files with data received from proxied servers.

​	​如果启用了 aio，则指定是否将其用于写入文件。目前，这仅在使用 `aio threads` 时有效，并且仅限于使用从代理服务器接收的数据写入临时文件。

### alias 别名

Syntax:`alias path;`

​	​语法： `alias path;`

Default: —

​	​默认值：—

Context: `location`

​	​上下文： `location`

Defines a replacement for the specified location. For example, with the following configuration

​	​定义指定位置的替换。例如，使用以下配置

```
location /i/ {
    alias /data/w3/images/;
}
```

on request of “`/i/top.gif`”, the file `/data/w3/images/top.gif` will be sent.

​	​根据“ `/i/top.gif` ”的请求，将发送文件 `/data/w3/images/top.gif` 。

The `path` value can contain variables, except `$document_root` and `$realpath_root`.

​	​ `path` 值可以包含变量，但 `$document_root` 和 `$realpath_root` 除外。

If `alias` is used inside a location defined with a regular expression then such regular expression should contain captures and `alias` should refer to these captures (0.7.40), for example:

​	​如果在使用正则表达式定义的位置内使用 `alias` ，则此类正则表达式应包含捕获，并且 `alias` 应引用这些捕获（0.7.40），例如：

```
location ~ ^/users/(.+\.(?:gif|jpe?g|png))$ {
    alias /data/w3/images/$1;
}
```

When location matches the last part of the directive’s value:

​	​当位置与指令值的最后部分匹配时：

```
location /images/ {
    alias /data/w3/images/;
}
```

it is better to use the [root](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#root) directive instead:

​	​最好改用根指令：

```
location /images/ {
    root /data/w3;
}
```

### auth_delay

Syntax: `auth_delay time;`

​	​语法： `auth_delay time;`

Default: `auth_delay 0s;`

​	​默认值： `auth_delay 0s;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

This directive appeared in version 1.17.10.

​	​此指令出现在 1.17.10 版本中。

Delays processing of unauthorized requests with 401 response code to prevent timing attacks when access is limited by [password](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html), by the [result of subrequest](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html), or by [JWT](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html).

​	​通过 401 响应代码延迟处理未经授权的请求，以防止在通过密码、子请求的结果或 JWT 限制访问时出现时序攻击。

### chunked_transfer_encoding

Syntax:`chunked_transfer_encoding on | off;`

​	​语法： `chunked_transfer_encoding on | off;`

Default: `chunked_transfer_encoding on;`

​	​默认值： `chunked_transfer_encoding on;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Allows disabling chunked transfer encoding in HTTP/1.1. It may come in handy when using a software failing to support chunked encoding despite the standard’s requirement.

​	​允许在 HTTP/1.1 中禁用块传输编码。尽管标准有此要求，但当使用无法支持块编码的软件时，此功能可能非常有用。

### client_body_buffer_size

Syntax: `client_body_buffer_size size;`

​	​语法： `client_body_buffer_size size;`

Default: `client_body_buffer_size 8k|16k;`

​	​默认值： `client_body_buffer_size 8k|16k;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Sets buffer size for reading client request body. In case the request body is larger than the buffer, the whole body or only its part is written to a [temporary file](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#client_body_temp_path). By default, buffer size is equal to two memory pages. This is 8K on x86, other 32-bit platforms, and x86-64. It is usually 16K on other 64-bit platforms.

​	​设置用于读取客户端请求正文的缓冲区大小。如果请求正文大于缓冲区，则整个正文或其一部分将被写入临时文件。默认情况下，缓冲区大小等于两个内存页。在 x86、其他 32 位平台和 x86-64 上为 8K。在其他 64 位平台上通常为 16K。

### client_body_in_file_only

Syntax:`client_body_in_file_only on | clean | off;`

​	​语法： `client_body_in_file_only on | clean | off;`

Default: `client_body_in_file_only off;`

​	​默认值： `client_body_in_file_only off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Determines whether nginx should save the entire client request body into a file. This directive can be used during debugging, or when using the `$request_body_file` variable, or the [$r->request_body_file](https://before80.github.io/nginx_docs/mod_ref/ngx_http_perl_module/#methods) method of the module [ngx_http_perl_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_perl_module).

​	​确定 nginx 是否应将整个客户端请求正文保存到文件中。此指令可在调试期间使用，或在使用 `$request_body_file` 变量或模块 ngx_http_perl_module 的 $r->request_body_file 方法时使用。

When set to the value `on`, temporary files are not removed after request processing.

​	​当设置为值 `on` 时，临时文件在请求处理后不会被删除。

The value `clean` will cause the temporary files left after request processing to be removed.

​	​值 `clean` 将导致请求处理后留下的临时文件被删除。

### client_body_in_single_buffer

Syntax:`client_body_in_single_buffer on | off;`

​	​语法： `client_body_in_single_buffer on | off;`

Default: `client_body_in_single_buffer off;`

​	​默认值： `client_body_in_single_buffer off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Determines whether nginx should save the entire client request body in a single buffer. The directive is recommended when using the `$request_body` variable, to save the number of copy operations involved.

​	​确定 nginx 是否应将整个客户端请求正文保存在一个缓冲区中。当使用 `$request_body` 变量时，建议使用此指令，以节省涉及的复制操作数。

### client_body_temp_path

Syntax:`client_body_temp_path path [level1 [level2 [level3]]];`

​	​语法： `client_body_temp_path path [level1 [level2 [level3]]];`

Default: `client_body_temp_path client_body_temp;`

​	​默认值： `client_body_temp_path client_body_temp;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Defines a directory for storing temporary files holding client request bodies. Up to three-level subdirectory hierarchy can be used under the specified directory. For example, in the following configuration

​	​定义一个目录，用于存储包含客户端请求正文的临时文件。可以在指定目录下使用最多三级子目录层次结构。例如，在以下配置中

```
client_body_temp_path /spool/nginx/client_temp 1 2;
```

a path to a temporary file might look like this:

​	​临时文件的路径可能如下所示：

```
/spool/nginx/client_temp/7/45/00000123457
```

### client_body_timeout

Syntax: `client_body_timeout time;`

​	​语法： `client_body_timeout time;`

Default: `client_body_timeout 60s;`

​	​默认值： `client_body_timeout 60s;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Defines a timeout for reading client request body. The timeout is set only for a period between two successive read operations, not for the transmission of the whole request body. If a client does not transmit anything within this time, the request is terminated with the 408 (Request Time-out) error.

​	​定义一个超时时间，用于读取客户端请求正文。超时时间仅针对两次连续读取操作之间的时段设置，不针对整个请求正文的传输设置。如果客户端在此时间内未传输任何内容，则请求将终止，并显示 408（请求超时）错误。

### client_header_buffer_size

Syntax:`client_header_buffer_size size;`

​	​语法： `client_header_buffer_size size;`

Default: `client_header_buffer_size 1k;`

​	​默认值： `client_header_buffer_size 1k;`

Context: `http`, `server`

​	​上下文： `http` , `server`

Sets buffer size for reading client request header. For most requests, a buffer of 1K bytes is enough. However, if a request includes long cookies, or comes from a WAP client, it may not fit into 1K. If a request line or a request header field does not fit into this buffer then larger buffers, configured by the [large_client_header_buffers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#large_client_header_buffers) directive, are allocated.

​	​设置读取客户端请求标头的缓冲区大小。对于大多数请求，1K 字节的缓冲区就足够了。但是，如果请求包含较长的 cookie，或来自 WAP 客户端，则可能无法放入 1K。如果请求行或请求标头字段不适合此缓冲区，则会分配由 large_client_header_buffers 指令配置的较大缓冲区。

If the directive is specified on the [server](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server) level, the value from the default server can be used. Details are provided in the “[Virtual server selection](https://before80.github.io/nginx_docs/introduction/serverNames/#虚拟服务器选择---virtual-server-selection)” section.

​	​如果在服务器级别指定该指令，则可以使用默认服务器中的值。详细信息在“虚拟服务器选择”部分中提供。

### client_header_timeout

Syntax:`client_header_timeout time;`

​	​语法： `client_header_timeout time;`

Default: `client_header_timeout 60s;`

​	​默认值： `client_header_timeout 60s;`

Context: `http`, `server`

​	​上下文： `http` , `server`

Defines a timeout for reading client request header. If a client does not transmit the entire header within this time, the request is terminated with the 408 (Request Time-out) error.

​	​定义读取客户端请求标头的超时时间。如果客户端未在此时间内传输整个标头，则请求将终止，并显示 408（请求超时）错误。

### client_max_body_size

Syntax:`client_max_body_size size;`

​	​语法： `client_max_body_size size;`

Default: `client_max_body_size 1m;`

​	​默认值： `client_max_body_size 1m;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Sets the maximum allowed size of the client request body. If the size in a request exceeds the configured value, the 413 (Request Entity Too Large) error is returned to the client. Please be aware that browsers cannot correctly display this error. Setting `size` to 0 disables checking of client request body size.

​	​设置客户端请求正文允许的最大大小。如果请求中的大小超过配置的值，则会向客户端返回 413（请求实体过大）错误。请注意，浏览器无法正确显示此错误。将 `size` 设置为 0 将禁用对客户端请求正文大小的检查。

### connection_pool_size

Syntax: `connection_pool_size size;`

​	​语法： `connection_pool_size size;`

Default: `connection_pool_size 256|512;`

​	​默认值： `connection_pool_size 256|512;`

Context: `http`, `server`

​	​上下文： `http` , `server`

Allows accurate tuning of per-connection memory allocations. This directive has minimal impact on performance and should not generally be used. By default, the size is equal to 256 bytes on 32-bit platforms and 512 bytes on 64-bit platforms.

​	​允许对每个连接的内存分配进行精确调整。此指令对性能的影响很小，通常不应使用。默认情况下，大小在 32 位平台上等于 256 字节，在 64 位平台上等于 512 字节。

Prior to version 1.9.8, the default value was 256 on all platforms.

​	​在 1.9.8 版本之前，默认值在所有平台上均为 256。

### default_type 默认类型

Syntax: `default_type mime-type;`

​	​语法： `default_type mime-type;`

Default: `default_type text/plain;`

​	​默认值： `default_type text/plain;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Defines the default MIME type of a response. Mapping of file name extensions to MIME types can be set with the [types](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#types) directive.

​	​定义响应的默认 MIME 类型。可以使用 types 指令设置文件名扩展名到 MIME 类型的映射。

### directio

Syntax: `directio size | off;`

​	​语法： `directio size | off;`

Default: `directio off;`

​	​默认值： `directio off;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

This directive appeared in version 0.7.7.

​	​此指令出现在 0.7.7 版本中。

Enables the use of the `O_DIRECT` flag (FreeBSD, Linux), the `F_NOCACHE` flag (macOS), or the `directio()` function (Solaris), when reading files that are larger than or equal to the specified `size`. The directive automatically disables (0.7.15) the use of [sendfile](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#sendfile) for a given request. It can be useful for serving large files:

​	​在读取大于或等于指定 `size` 的文件时，启用使用 `O_DIRECT` 标志（FreeBSD、Linux）、 `F_NOCACHE` 标志（macOS）或 `directio()` 函数（Solaris）。该指令会自动禁用（0.7.15）对给定请求使用 sendfile。它可用于提供大型文件：

```
directio 4m;
```

or when using [aio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#aio) on Linux.

​	​或在 Linux 上使用 aio 时。

### directio_alignment

Syntax: `directio_alignment size;`

​	​语法： `directio_alignment size;`

Default: `directio_alignment 512;`

​	​默认值： `directio_alignment 512;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 0.8.11.

​	​此指令出现在 0.8.11 版中。

Sets the alignment for [directio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#directio). In most cases, a 512-byte alignment is enough. However, when using XFS under Linux, it needs to be increased to 4K.

​	​设置 directio 的对齐方式。在大多数情况下，512 字节的对齐方式就足够了。但是，在 Linux 下使用 XFS 时，需要将其增加到 4K。

### disable_symlinks

Syntax:`disable_symlinks off;` `disable_symlinks on | if_not_owner [from=part];`

​	​语法： `disable_symlinks off;` `disable_symlinks on | if_not_owner [from=part];`

Default: `disable_symlinks off;`

​	​默认值： `disable_symlinks off;`

Context: `http`, `server`, `location`

​	​上下文： `http` 、 `server` 、 `location`

This directive appeared in version 1.1.15.

​	​此指令出现在版本 1.1.15 中。

Determines how symbolic links should be treated when opening files:

​	​确定打开文件时如何处理符号链接：

- `off`

  Symbolic links in the pathname are allowed and not checked. This is the default behavior.

  ​	​允许路径名中的符号链接，且不检查。这是默认行为。

- `on`

  If any component of the pathname is a symbolic link, access to a file is denied.

  ​	​如果路径名的任何组件是符号链接，则拒绝访问文件。

- `if_not_owner`

  Access to a file is denied if any component of the pathname is a symbolic link, and the link and object that the link points to have different owners.

  ​	​如果路径名的任何组件是符号链接，并且链接和链接指向的对象具有不同的所有者，则拒绝访问文件。

- `from`=`part`

  When checking symbolic links (parameters `on` and `if_not_owner`), all components of the pathname are normally checked. Checking of symbolic links in the initial part of the pathname may be avoided by specifying additionally the `from`=`part` parameter. In this case, symbolic links are checked only from the pathname component that follows the specified initial part. If the value is not an initial part of the pathname checked, the whole pathname is checked as if this parameter was not specified at all. If the value matches the whole file name, symbolic links are not checked. The parameter value can contain variables.

  ​	​在检查符号链接（参数 `on` 和 `if_not_owner` ）时，通常会检查路径名的所有组件。可以通过另外指定 `from` = `part` 参数来避免检查路径名初始部分中的符号链接。在这种情况下，仅从指定初始部分之后的路径名组件检查符号链接。如果该值不是所检查路径名的初始部分，则会像根本未指定此参数一样检查整个路径名。如果该值与整个文件名匹配，则不检查符号链接。参数值可以包含变量。

Example:

​	​示例：

```
disable_symlinks on from=$document_root;
```

This directive is only available on systems that have the `openat()` and `fstatat()` interfaces. Such systems include modern versions of FreeBSD, Linux, and Solaris.

​	​此指令仅适用于具有 `openat()` 和 `fstatat()` 接口的系统。此类系统包括 FreeBSD、Linux 和 Solaris 的现代版本。

Parameters `on` and `if_not_owner` add a processing overhead.

​	​参数 `on` 和 `if_not_owner` 会增加处理开销。

On systems that do not support opening of directories only for search, to use these parameters it is required that worker processes have read permissions for all directories being checked.

​	​在仅支持打开目录以进行搜索的系统上，要使用这些参数，需要工作进程对正在检查的所有目录具有读取权限。

The [ngx_http_autoindex_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_autoindex_module), [ngx_http_random_index_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_random_index_module), and [ngx_http_dav_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_dav_module) modules currently ignore this directive.

​	​ngx_http_autoindex_module、ngx_http_random_index_module 和 ngx_http_dav_module 模块当前忽略此指令。

### error_page

Syntax: `error_page code ... [=[response]] uri;`

​	​语法： `error_page code ... [=[response]] uri;`

Default: —

​	​默认值：—

Context: `http`, `server`, `location`, `if in location`

​	​上下文： `http` , `server` , `location` , `if in location`

Defines the URI that will be shown for the specified errors. A `uri` value can contain variables.

​	​定义将针对指定错误显示的 URI。 `uri` 值可以包含变量。

Example:

​	​示例：

```
error_page 404             /404.html;
error_page 500 502 503 504 /50x.html;
```

This causes an internal redirect to the specified `uri` with the client request method changed to “`GET`” (for all methods other than “`GET`” and “`HEAD`”).

​	​这会导致内部重定向到指定的 `uri` ，其中客户端请求方法更改为“ `GET` ”（对于除“ `GET` ”和“ `HEAD` ”之外的所有方法）。

Furthermore, it is possible to change the response code to another using the “`=` `response`” syntax, for example:

​	​此外，可以使用“ `=` `response` ”语法将响应代码更改为另一个，例如：

```
error_page 404 =200 /empty.gif;
```

If an error response is processed by a proxied server or a FastCGI/uwsgi/SCGI/gRPC server, and the server may return different response codes (e.g., 200, 302, 401 or 404), it is possible to respond with the code it returns:

​	​如果错误响应由代理服务器或 FastCGI/uwsgi/SCGI/gRPC 服务器处理，并且服务器可能返回不同的响应代码（例如，200、302、401 或 404），则可以响应它返回的代码：

```
error_page 404 = /404.php;
```

If there is no need to change URI and method during internal redirection it is possible to pass error processing into a named location:

​	​如果在内部重定向期间不需要更改 URI 和方法，则可以将错误处理传递到已命名的位置：

```
location / {
    error_page 404 = @fallback;
}

location @fallback {
    proxy_pass http://backend;
}
```

If `uri` processing leads to an error, the status code of the last occurred error is returned to the client.

​	​如果 `uri` 处理导致错误，则将最后发生的错误的状态代码返回给客户端。

It is also possible to use URL redirects for error processing:

​	​也可以对错误处理使用 URL 重定向：

```
error_page 403      http://example.com/forbidden.html;
error_page 404 =301 http://example.com/notfound.html;
```

In this case, by default, the response code 302 is returned to the client. It can only be changed to one of the redirect status codes (301, 302, 303, 307, and 308).

​	​在这种情况下，默认情况下，响应代码 302 会返回给客户端。它只能更改为其中一个重定向状态代码（301、302、303、307 和 308）。

The code 307 was not treated as a redirect until versions 1.1.16 and 1.0.13.

​	​在 1.1.16 和 1.0.13 版本之前，代码 307 未被视为重定向。

The code 308 was not treated as a redirect until version 1.13.0.

​	​在 1.13.0 版本之前，代码 308 未被视为重定向。

These directives are inherited from the previous configuration level if and only if there are no `error_page` directives defined on the current level.

​	​当且仅当当前级别上没有定义 `error_page` 指令时，这些指令才会从上一个配置级别继承。

### etag

Syntax: `etag on | off;`

​	​语法： `etag on | off;`

Default: `etag on;`

​	​默认值： `etag on;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

This directive appeared in version 1.3.3.

​	​此指令出现在 1.3.3 版本中。

Enables or disables automatic generation of the “ETag” response header field for static resources.

​	​启用或禁用为静态资源自动生成“ETag”响应头字段。

### http

Syntax:`http { ... }`

​	​语法： `http { ... }`

Default: —

​	​默认值：—

Context: `main`

​	​上下文： `main`

Provides the configuration file context in which the HTTP server directives are specified.

​	​提供指定 HTTP 服务器指令的配置文件上下文。

### if_modified_since

Syntax:`if_modified_since off | exact | before;`

​	​语法： `if_modified_since off | exact | before;`

Default: `if_modified_since exact;`

​	​默认值： `if_modified_since exact;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

This directive appeared in version 0.7.24.

​	​此指令出现在 0.7.24 版本中。

Specifies how to compare modification time of a response with the time in the “If-Modified-Since” request header field:

​	​指定如何将响应的修改时间与“If-Modified-Since”请求头字段中的时间进行比较：

- `off`

  the response is always considered modified (0.7.34);

  ​	​响应始终被视为已修改（0.7.34）；

- `exact`

  exact match;

  ​	​完全匹配；

- `before`

  modification time of the response is less than or equal to the time in the “If-Modified-Since” request header field.

  ​	​响应的修改时间小于或等于“If-Modified-Since”请求头字段中的时间。

### ignore_invalid_headers

Syntax:`ignore_invalid_headers on | off;`

​	​语法： `ignore_invalid_headers on | off;`

Default: `ignore_invalid_headers on;`

​	​默认值： `ignore_invalid_headers on;`

Context: `http`, `server`

​	​上下文： `http` ， `server`

Controls whether header fields with invalid names should be ignored. Valid names are composed of English letters, digits, hyphens, and possibly underscores (as controlled by the [underscores_in_headers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#underscores_in_headers) directive).

​	​控制是否忽略名称无效的头字段。有效名称由英文字母、数字、连字符和可能的下划线（由 underscores_in_headers 指令控制）组成。

If the directive is specified on the [server](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server) level, the value from the default server can be used. Details are provided in the “[Virtual server selection](https://before80.github.io/nginx_docs/introduction/serverNames/#虚拟服务器选择---virtual-server-selection)” section.

​	​如果在服务器级别指定该指令，则可以使用默认服务器中的值。详细信息在“虚拟服务器选择”部分中提供。

### internal 内部

Syntax:`internal;`

​	​语法： `internal;`

Default: —

​	​默认值：—

Context: `location`

​	​上下文： `location`

Specifies that a given location can only be used for internal requests. For external requests, the client error 404 (Not Found) is returned. Internal requests are the following:

​	​指定给定位置只能用于内部请求。对于外部请求，将返回客户端错误 404（未找到）。内部请求如下：

- requests redirected by the [error_page](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#error_page), [index](https://before80.github.io/nginx_docs/mod_ref/ngx_http_index_module/#index), [internal_redirect](https://before80.github.io/nginx_docs/mod_ref/ngx_http_internal_redirect_module/#internal_redirect), [random_index](https://before80.github.io/nginx_docs/mod_ref/ngx_http_random_index_module/#random_index), and [try_files](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#try_files) directives;
  由 error_page、index、internal_redirect、random_index 和 try_files 指令重定向的请求；
- requests redirected by the “X-Accel-Redirect” response header field from an upstream server;
  由上游服务器的“X-Accel-Redirect”响应头字段重定向的请求；
- subrequests formed by the “`include virtual`” command of the [ngx_http_ssi_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_ssi_module) module, by the [ngx_http_addition_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_addition_module) module directives, and by [auth_request](https://before80.github.io/nginx_docs/mod_ref/ngx_http_auth_request_module/#auth_request) and [mirror](https://before80.github.io/nginx_docs/mod_ref/ngx_http_mirror_module/#mirror) directives;
  由 ngx_http_ssi_module 模块的“ `include virtual` ”命令、ngx_http_addition_module 模块指令以及 auth_request 和 mirror 指令形成的子请求；
- requests changed by the [rewrite](https://before80.github.io/nginx_docs/mod_ref/ngx_http_rewrite_module/#rewrite) directive.
  由 rewrite 指令更改的请求。

Example:

​	​示例：

```
error_page 404 /404.html;

location = /404.html {
    internal;
}
```

There is a limit of 10 internal redirects per request to prevent request processing cycles that can occur in incorrect configurations. If this limit is reached, the error 500 (Internal Server Error) is returned. In such cases, the “rewrite or internal redirection cycle” message can be seen in the error log.

​	​每个请求最多只能进行 10 次内部重定向，以防止在错误配置中可能发生的请求处理循环。如果达到此限制，则会返回错误 500（内部服务器错误）。在这种情况下，可以在错误日志中看到“重写或内部重定向循环”消息。

### keepalive_disable

Syntax:`keepalive_disable none | browser ...;`

​	​语法： `keepalive_disable none | browser ...;`

Default: `keepalive_disable msie6;`

​	​默认值： `keepalive_disable msie6;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Disables keep-alive connections with misbehaving browsers. The `browser` parameters specify which browsers will be affected. The value `msie6` disables keep-alive connections with old versions of MSIE, once a POST request is received. The value `safari` disables keep-alive connections with Safari and Safari-like browsers on macOS and macOS-like operating systems. The value `none` enables keep-alive connections with all browsers.

​	​禁用与行为不当的浏览器之间的长连接。 `browser` 参数指定受影响的浏览器。值 `msie6` 在收到 POST 请求后禁用与旧版 MSIE 的长连接。值 `safari` 禁用与 macOS 和类似 macOS 的操作系统上的 Safari 和类似 Safari 的浏览器的长连接。值 `none` 启用与所有浏览器的长连接。

Prior to version 1.1.18, the value `safari` matched all Safari and Safari-like browsers on all operating systems, and keep-alive connections with them were disabled by default.

​	​在 1.1.18 版之前，值 `safari` 匹配所有操作系统上的所有 Safari 和类似 Safari 的浏览器，并且默认情况下禁用与它们的保持活动连接。

### keepalive_requests

Syntax:`keepalive_requests number;`

​	​语法： `keepalive_requests number;`

Default: `keepalive_requests 1000;`

​	​默认值： `keepalive_requests 1000;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 0.8.0.

​	​此指令出现在 0.8.0 版中。

Sets the maximum number of requests that can be served through one keep-alive connection. After the maximum number of requests are made, the connection is closed.

​	​设置可通过一个长连接提供服务的最大请求数。在达到最大请求数后，连接将关闭。

Closing connections periodically is necessary to free per-connection memory allocations. Therefore, using too high maximum number of requests could result in excessive memory usage and not recommended.

​	​定期关闭连接对于释放每个连接的内存分配是必要的。因此，使用过高的最大请求数可能会导致过度的内存使用，不建议这样做。

Prior to version 1.19.10, the default value was 100.

​	​在 1.19.10 版之前，默认值为 100。

### keepalive_time 保持活动时间

Syntax: `keepalive_time time;`

​	​语法： `keepalive_time time;`

Default: `keepalive_time 1h;`

​	​默认值： `keepalive_time 1h;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 1.19.10.

​	​此指令出现在 1.19.10 版本中。

Limits the maximum time during which requests can be processed through one keep-alive connection. After this time is reached, the connection is closed following the subsequent request processing.

​	​限制通过一个保持活动连接处理请求的最长时间。达到此时间后，在后续请求处理后关闭连接。

### keepalive_timeout 保持活动超时

Syntax:`keepalive_timeout timeout [header_timeout];`

​	​语法： `keepalive_timeout timeout [header_timeout];`

Default: `keepalive_timeout 75s;`

​	​默认值： `keepalive_timeout 75s;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

The first parameter sets a timeout during which a keep-alive client connection will stay open on the server side. The zero value disables keep-alive client connections. The optional second parameter sets a value in the “Keep-Alive: timeout=`time`” response header field. Two parameters may differ.

​	​第一个参数设置一个超时时间，在此期间，保持活动客户端连接将在服务器端保持打开状态。零值禁用保持活动客户端连接。可选的第二个参数在“Keep-Alive: timeout= `time` ”响应头字段中设置一个值。两个参数可能不同。

The “Keep-Alive: timeout=`time`” header field is recognized by Mozilla and Konqueror. MSIE closes keep-alive connections by itself in about 60 seconds.

​	​“Keep-Alive: timeout= `time` ”头字段被 Mozilla 和 Konqueror 识别。MSIE 在大约 60 秒内自行关闭保持活动连接。

### large_client_header_buffers

Syntax:`large_client_header_buffers number size;`

​	​语法： `large_client_header_buffers number size;`

Default: `large_client_header_buffers 4 8k;`

​	​默认值： `large_client_header_buffers 4 8k;`

Context: `http`, `server`

​	​上下文： `http` ， `server`

Sets the maximum `number` and `size` of buffers used for reading large client request header. A request line cannot exceed the size of one buffer, or the 414 (Request-URI Too Large) error is returned to the client. A request header field cannot exceed the size of one buffer as well, or the 400 (Bad Request) error is returned to the client. Buffers are allocated only on demand. By default, the buffer size is equal to 8K bytes. If after the end of request processing a connection is transitioned into the keep-alive state, these buffers are released.

​	​设置用于读取大型客户端请求标头的缓冲区的最大 `number` 和 `size` 。请求行不能超过一个缓冲区的大小，否则会向客户端返回 414（请求 URI 过大）错误。请求头字段也不能超过一个缓冲区的大小，否则会向客户端返回 400（错误的请求）错误。仅按需分配缓冲区。默认情况下，缓冲区大小等于 8K 字节。如果在请求处理结束后，连接转换到保持活动状态，则会释放这些缓冲区。

If the directive is specified on the [server](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server) level, the value from the default server can be used. Details are provided in the “[Virtual server selection](https://before80.github.io/nginx_docs/introduction/serverNames/#虚拟服务器选择---virtual-server-selection)” section.

​	​如果在服务器级别指定指令，则可以使用默认服务器中的值。详细信息在“虚拟服务器选择”部分中提供。

### limit_except

Syntax:`limit_except method ... { ... }`

​	​语法： `limit_except method ... { ... }`

Default: —

​	​默认值：—

Context: `location`

​	​上下文： `location`

Limits allowed HTTP methods inside a location. The `method` parameter can be one of the following: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `MKCOL`, `COPY`, `MOVE`, `OPTIONS`, `PROPFIND`, `PROPPATCH`, `LOCK`, `UNLOCK`, or `PATCH`. Allowing the `GET` method makes the `HEAD` method also allowed. Access to other methods can be limited using the [ngx_http_access_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_access_module), [ngx_http_auth_basic_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_auth_basic_module), and [ngx_http_auth_jwt_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_auth_jwt_module) (1.13.10) modules directives:

​	​限制位置内允许的 HTTP 方法。 `method` 参数可以是以下之一： `GET` 、 `HEAD` 、 `POST` 、 `PUT` 、 `DELETE` 、 `MKCOL` 、 `COPY` 、 `MOVE` 、 `OPTIONS` 、 `PROPFIND` 、 `PROPPATCH` 、 `LOCK` 、 `UNLOCK` 或 `PATCH` 。允许 `GET` 方法也会允许 `HEAD` 方法。可以使用 ngx_http_access_module、ngx_http_auth_basic_module 和 ngx_http_auth_jwt_module (1.13.10) 模块指令限制对其他方法的访问：

```
limit_except GET {
    allow 192.168.1.0/32;
    deny  all;
}
```

Please note that this will limit access to all methods except GET and HEAD.

​	​请注意，这将限制对 GET 和 HEAD 之外的所有方法的访问。

### limit_rate

Syntax: `limit_rate rate;`

​	​语法： `limit_rate rate;`

Default: `limit_rate 0;`

​	​默认值： `limit_rate 0;`

Context: `http`, `server`, `location`, `if in location`

​	​上下文： `http` ， `server` ， `location` ， `if in location`

Limits the rate of response transmission to a client. The `rate` is specified in bytes per second. The zero value disables rate limiting. The limit is set per a request, and so if a client simultaneously opens two connections, the overall rate will be twice as much as the specified limit.

​	​限制向客户端响应传输的速率。 `rate` 以每秒字节数指定。零值禁用速率限制。限制是针对每个请求设置的，因此，如果客户端同时打开两个连接，则总体速率将是指定限制的两倍。

Parameter value can contain variables (1.17.0). It may be useful in cases where rate should be limited depending on a certain condition:

​	​参数值可以包含变量 (1.17.0)。在根据特定条件限制速率的情况下，这可能很有用：

```
map $slow $rate {
    1     4k;
    2     8k;
}

limit_rate $rate;
```

Rate limit can also be set in the [`$limit_rate`](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#var_limit_rate) variable, however, since version 1.17.0, this method is not recommended:

​	​速率限制也可以在 `$limit_rate` 变量中设置，但是，自 1.17.0 版本以来，不建议使用此方法：

```
server {

    if ($slow) {
        set $limit_rate 4k;
    }

    ...
}
```

Rate limit can also be set in the “X-Accel-Limit-Rate” header field of a proxied server response. This capability can be disabled using the [proxy_ignore_headers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_proxy_module/#proxy_ignore_headers), [fastcgi_ignore_headers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_fastcgi_module/#fastcgi_ignore_headers), [uwsgi_ignore_headers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_uwsgi_module/#uwsgi_ignore_headers), and [scgi_ignore_headers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_scgi_module/#scgi_ignore_headers) directives.

​	​速率限制也可以在代理服务器响应的“X-Accel-Limit-Rate”标头字段中设置。可以使用 proxy_ignore_headers、fastcgi_ignore_headers、uwsgi_ignore_headers 和 scgi_ignore_headers 指令禁用此功能。

### limit_rate_after

Syntax: `limit_rate_after size;`

​	​语法： `limit_rate_after size;`

Default: `limit_rate_after 0;`

​	​默认值： `limit_rate_after 0;`

Context: `http`, `server`, `location`, `if in location`

​	​上下文： `http` ， `server` ， `location` ， `if in location`

This directive appeared in version 0.8.0.

​	​此指令出现在 0.8.0 版本中。

Sets the initial amount after which the further transmission of a response to a client will be rate limited. Parameter value can contain variables (1.17.0).

​	​设置初始数量，在此数量之后，对客户端的响应的进一步传输将受到速率限制。参数值可以包含变量 (1.17.0)。

Example:

​	​示例：

```
location /flv/ {
    flv;
    limit_rate_after 500k;
    limit_rate       50k;
}
```

### lingering_close

Syntax:`lingering_close off | on | always;`

​	​语法： `lingering_close off | on | always;`

Default: `lingering_close on;`

​	​默认值： `lingering_close on;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in versions 1.1.0 and 1.0.6.

​	​此指令出现在版本 1.1.0 和 1.0.6 中。

Controls how nginx closes client connections.

​	​控制 nginx 如何关闭客户端连接。

The default value “`on`” instructs nginx to [wait for](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#lingering_timeout) and [process](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#lingering_time) additional data from a client before fully closing a connection, but only if heuristics suggests that a client may be sending more data.

​	​默认值 “ `on` ” 指示 nginx 等待并处理来自客户端的附加数据，然后再完全关闭连接，但仅当启发式方法表明客户端可能正在发送更多数据时。

The value “`always`” will cause nginx to unconditionally wait for and process additional client data.

​	​值 “ `always` ” 将导致 nginx 无条件地等待并处理附加客户端数据。

The value “`off`” tells nginx to never wait for more data and close the connection immediately. This behavior breaks the protocol and should not be used under normal circumstances.

​	​值 “ `off` ” 告诉 nginx 永远不要等待更多数据并立即关闭连接。此行为违反协议，不应在正常情况下使用。

To control closing [HTTP/2](https://nginx.org/en/docs/http/ngx_http_v2_module.html) connections, the directive must be specified on the [server](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server) level (1.19.1).

​	​要控制关闭 HTTP/2 连接，必须在服务器级别指定该指令 (1.19.1)。

### lingering_time

Syntax: `lingering_time time;`

​	​语法： `lingering_time time;`

Default: `lingering_time 30s;`

​	​默认值： `lingering_time 30s;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

When [lingering_close](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#lingering_close) is in effect, this directive specifies the maximum time during which nginx will process (read and ignore) additional data coming from a client. After that, the connection will be closed, even if there will be more data.

​	​当 lingering_close 生效时，此指令指定 nginx 将处理（读取和忽略）来自客户端的其他数据的最长时间。之后，即使有更多数据，连接也将关闭。

### lingering_timeout

Syntax: `lingering_timeout time;`

​	​语法： `lingering_timeout time;`

Default: `lingering_timeout 5s;`

​	​默认值： `lingering_timeout 5s;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

When [lingering_close](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#lingering_close) is in effect, this directive specifies the maximum waiting time for more client data to arrive. If data are not received during this time, the connection is closed. Otherwise, the data are read and ignored, and nginx starts waiting for more data again. The “wait-read-ignore” cycle is repeated, but no longer than specified by the [lingering_time](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#lingering_time) directive.

​	​当 lingering_close 生效时，此指令指定等待更多客户端数据到达的最大等待时间。如果在此期间未收到数据，则连接将关闭。否则，将读取并忽略数据，并且 nginx 重新开始等待更多数据。“等待-读取-忽略”循环将重复，但不会超过 lingering_time 指令指定的时间。

### listen

Syntax:`listen address[:port] [default_server] [ssl] [http2 | quic] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];` `listen port [default_server] [ssl] [http2 | quic] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];` `listen unix:path [default_server] [ssl] [http2 | quic] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];`

​	​语法： `listen address[:port] [default_server] [ssl] [http2 | quic] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];` `listen port [default_server] [ssl] [http2 | quic] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];` `listen unix:path [default_server] [ssl] [http2 | quic] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];`

Default: `listen *:80 | *:8000;`

​	​默认值： `listen *:80 | *:8000;`

Context: `server`

​	​上下文： `server`

Sets the `address` and `port` for IP, or the `path` for a UNIX-domain socket on which the server will accept requests. Both `address` and `port`, or only `address` or only `port` can be specified. An `address` may also be a hostname, for example:

​	​为 IP 设置 `address` 和 `port` ，或为服务器将接受请求的 UNIX 域套接字设置 `path` 。可以同时指定 `address` 和 `port` ，也可以仅指定 `address` 或仅指定 `port` 。一个 `address` 也可以是一个主机名，例如：

```
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;
```

IPv6 addresses (0.7.36) are specified in square brackets:

​	​IPv6 地址（0.7.36）用方括号指定：

```
listen [::]:8000;
listen [::1];
```

UNIX-domain sockets (0.8.21) are specified with the “`unix:`” prefix:

​	​UNIX 域套接字（0.8.21）用“ `unix:` ”前缀指定：

```
listen unix:/var/run/nginx.sock;
```

If only `address` is given, the port 80 is used.

​	​如果仅给出 `address` ，则使用端口 80。

If the directive is not present then either `*:80` is used if nginx runs with the superuser privileges, or `*:8000` otherwise.

​	​如果指令不存在，则如果 nginx 以超级用户权限运行，则使用 `*:80` ，否则使用 `*:8000` 。

The `default_server` parameter, if present, will cause the server to become the default server for the specified `address`:`port` pair. If none of the directives have the `default_server` parameter then the first server with the `address`:`port` pair will be the default server for this pair.

​	​如果存在 `default_server` 参数，服务器将成为指定 `address` : `port` 对的默认服务器。如果没有任何指令具有 `default_server` 参数，则具有 `address` : `port` 对的第一台服务器将成为此对的默认服务器。

In versions prior to 0.8.21 this parameter is named simply `default`.

​	​在 0.8.21 之前的版本中，此参数仅命名为 `default` 。

The `ssl` parameter (0.7.14) allows specifying that all connections accepted on this port should work in SSL mode. This allows for a more compact [configuration](https://nginx.org/en/docs/http/configuring_https_servers.html#single_http_https_server) for the server that handles both HTTP and HTTPS requests.

​	​ `ssl` 参数 (0.7.14) 允许指定在此端口上接受的所有连接都应在 SSL 模式下工作。这允许为同时处理 HTTP 和 HTTPS 请求的服务器提供更紧凑的配置。

The `http2` parameter (1.9.5) configures the port to accept [HTTP/2](https://nginx.org/en/docs/http/ngx_http_v2_module.html) connections. Normally, for this to work the `ssl` parameter should be specified as well, but nginx can also be configured to accept HTTP/2 connections without SSL.

​	​ `http2` 参数 (1.9.5) 配置端口以接受 HTTP/2 连接。通常，为此工作，还应指定 `ssl` 参数，但 nginx 也可以配置为在没有 SSL 的情况下接受 HTTP/2 连接。

The parameter is deprecated, the [http2](https://nginx.org/en/docs/http/ngx_http_v2_module.html#http2) directive should be used instead.

​	​该参数已弃用，应改用 http2 指令。

The `quic` parameter (1.25.0) configures the port to accept [QUIC](https://nginx.org/en/docs/http/ngx_http_v3_module.html) connections.

​	​ `quic` 参数 (1.25.0) 配置端口以接受 QUIC 连接。

The `proxy_protocol` parameter (1.5.12) allows specifying that all connections accepted on this port should use the [PROXY protocol](http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt).

​	​ `proxy_protocol` 参数 (1.5.12) 允许指定在此端口上接受的所有连接都应使用 PROXY 协议。

The PROXY protocol version 2 is supported since version 1.13.11.

​	​自 1.13.11 版本以来支持 PROXY 协议版本 2。

The `listen` directive can have several additional parameters specific to socket-related system calls. These parameters can be specified in any `listen` directive, but only once for a given `address`:`port` pair.

​	​ `listen` 指令可以有几个特定于套接字相关系统调用的附加参数。这些参数可以在任何 `listen` 指令中指定，但对于给定的 `address` : `port` 对仅能指定一次。

In versions prior to 0.8.21, they could only be specified in the `listen` directive together with the `default` parameter.

​	​在 0.8.21 之前的版本中，它们只能与 `default` 参数一起在 `listen` 指令中指定。

- `setfib`=`number`

  this parameter (0.8.44) sets the associated routing table, FIB (the `SO_SETFIB` option) for the listening socket. This currently works only on FreeBSD.

  ​	​此参数 (0.8.44) 为侦听套接字设置关联的路由表 FIB（ `SO_SETFIB` 选项）。目前仅在 FreeBSD 上有效。

- `fastopen`=`number`

  enables “[TCP Fast Open](http://en.wikipedia.org/wiki/TCP_Fast_Open)” for the listening socket (1.5.8) and [limits](https://datatracker.ietf.org/doc/html/rfc7413#section-5.1) the maximum length for the queue of connections that have not yet completed the three-way handshake.Do not enable this feature unless the server can handle receiving the [same SYN packet with data](https://datatracker.ietf.org/doc/html/rfc7413#section-6.1) more than once.

  ​	​为侦听套接字启用“TCP 快速打开”（1.5.8），并限制尚未完成三向握手的连接队列的最大长度。除非服务器可以处理多次接收带有数据的相同 SYN 数据包，否则不要启用此功能。

- `backlog`=`number`

  sets the `backlog` parameter in the `listen()` call that limits the maximum length for the queue of pending connections. By default, `backlog` is set to -1 on FreeBSD, DragonFly BSD, and macOS, and to 511 on other platforms.

  ​	​在 `listen()` 调用中设置 `backlog` 参数，该参数限制挂起连接队列的最大长度。默认情况下， `backlog` 在 FreeBSD、DragonFly BSD 和 macOS 上设置为 -1，在其他平台上设置为 511。

- `rcvbuf`=`size`

  sets the receive buffer size (the `SO_RCVBUF` option) for the listening socket.

  ​	​为侦听套接字设置接收缓冲区大小（ `SO_RCVBUF` 选项）。

- `sndbuf`=`size`

  sets the send buffer size (the `SO_SNDBUF` option) for the listening socket.

  ​	​设置监听套接字的发送缓冲区大小（ `SO_SNDBUF` 选项）。

- `accept_filter`=`filter`

  sets the name of accept filter (the `SO_ACCEPTFILTER` option) for the listening socket that filters incoming connections before passing them to `accept()`. This works only on FreeBSD and NetBSD 5.0+. Possible values are [dataready](http://man.freebsd.org/accf_data) and [httpready](http://man.freebsd.org/accf_http).

  ​	​设置监听套接字的接受过滤器名称（ `SO_ACCEPTFILTER` 选项），该过滤器在将传入连接传递给 `accept()` 之前对其进行过滤。这仅适用于 FreeBSD 和 NetBSD 5.0+。可能的值为 dataready 和 httpready。

- `deferred`

  instructs to use a deferred `accept()` (the `TCP_DEFER_ACCEPT` socket option) on Linux.

  ​	​指示在 Linux 上使用延迟 `accept()` （ `TCP_DEFER_ACCEPT` 套接字选项）。

- `bind`

  instructs to make a separate `bind()` call for a given `address`:`port` pair. This is useful because if there are several `listen` directives with the same port but different addresses, and one of the `listen` directives listens on all addresses for the given port (`*:port`), nginx will `bind()` only to `*:port`. It should be noted that the `getsockname()` system call will be made in this case to determine the address that accepted the connection. If the `setfib`, `fastopen`, `backlog`, `rcvbuf`, `sndbuf`, `accept_filter`, `deferred`, `ipv6only`, `reuseport`, or `so_keepalive` parameters are used then for a given `address`:`port` pair a separate `bind()` call will always be made.

  ​	​指示对给定的 `address` : `port` 对进行单独的 `bind()` 调用。这很有用，因为如果有多个 `listen` 指令具有相同的端口但不同的地址，并且其中一个 `listen` 指令侦听给定端口的所有地址（ `*:port` ），nginx 将仅 `bind()` 到 `*:port` 。应该注意的是，在这种情况下将进行 `getsockname()` 系统调用以确定接受连接的地址。如果使用 `setfib` 、 `fastopen` 、 `backlog` 、 `rcvbuf` 、 `sndbuf` 、 `accept_filter` 、 `deferred` 、 `ipv6only` 、 `reuseport` 或 `so_keepalive` 参数，那么对于给定的 `address` : `port` 对，将始终进行单独的 `bind()` 调用。

- `ipv6only`=`on`|`off`

  this parameter (0.7.42) determines (via the `IPV6_V6ONLY` socket option) whether an IPv6 socket listening on a wildcard address `[::]` will accept only IPv6 connections or both IPv6 and IPv4 connections. This parameter is turned on by default. It can only be set once on start.Prior to version 1.3.4, if this parameter was omitted then the operating system’s settings were in effect for the socket.

  ​	​此参数 (0.7.42) 确定 (通过 `IPV6_V6ONLY` 套接字选项) 监听通配符地址 `[::]` 的 IPv6 套接字是否只接受 IPv6 连接或 IPv6 和 IPv4 连接。此参数默认开启。它只能在启动时设置一次。在 1.3.4 版本之前，如果省略此参数，则套接字将采用操作系统的设置。

- `reuseport`

  this parameter (1.9.1) instructs to create an individual listening socket for each worker process (using the `SO_REUSEPORT` socket option on Linux 3.9+ and DragonFly BSD, or `SO_REUSEPORT_LB` on FreeBSD 12+), allowing a kernel to distribute incoming connections between worker processes. This currently works only on Linux 3.9+, DragonFly BSD, and FreeBSD 12+ (1.15.1).Inappropriate use of this option may have its security [implications](http://man7.org/linux/man-pages/man7/socket.7.html).

  ​	​此参数 (1.9.1) 指示为每个工作进程创建一个单独的监听套接字（在 Linux 3.9+ 和 DragonFly BSD 上使用 `SO_REUSEPORT` 套接字选项，或在 FreeBSD 12+ 上使用 `SO_REUSEPORT_LB` ），允许内核在工作进程之间分配传入连接。此功能目前仅适用于 Linux 3.9+、DragonFly BSD 和 FreeBSD 12+ (1.15.1)。不当使用此选项可能会产生安全隐患。

- `so_keepalive`=`on`|`off`|[`keepidle`]:[`keepintvl`]:[`keepcnt`]

  this parameter (1.1.11) configures the “TCP keepalive” behavior for the listening socket. If this parameter is omitted then the operating system’s settings will be in effect for the socket. If it is set to the value “`on`”, the `SO_KEEPALIVE` option is turned on for the socket. If it is set to the value “`off`”, the `SO_KEEPALIVE` option is turned off for the socket. Some operating systems support setting of TCP keepalive parameters on a per-socket basis using the `TCP_KEEPIDLE`, `TCP_KEEPINTVL`, and `TCP_KEEPCNT` socket options. On such systems (currently, Linux 2.4+, NetBSD 5+, and FreeBSD 9.0-STABLE), they can be configured using the `keepidle`, `keepintvl`, and `keepcnt` parameters. One or two parameters may be omitted, in which case the system default setting for the corresponding socket option will be in effect. For example,`so_keepalive=30m::10`will set the idle timeout (`TCP_KEEPIDLE`) to 30 minutes, leave the probe interval (`TCP_KEEPINTVL`) at its system default, and set the probes count (`TCP_KEEPCNT`) to 10 probes.

  ​	​此参数 (1.1.11) 配置侦听套接字的“TCP 保活”行为。如果省略此参数，则操作系统设置将对套接字生效。如果将其设置为“ `on` ”值，则为套接字启用 `SO_KEEPALIVE` 选项。如果将其设置为“ `off` ”值，则为套接字禁用 `SO_KEEPALIVE` 选项。某些操作系统支持使用 `TCP_KEEPIDLE` 、 `TCP_KEEPINTVL` 和 `TCP_KEEPCNT` 套接字选项逐个套接字设置 TCP 保活参数。在这些系统上（目前为 Linux 2.4+、NetBSD 5+ 和 FreeBSD 9.0-STABLE），可以使用 `keepidle` 、 `keepintvl` 和 `keepcnt` 参数对其进行配置。可以省略一个或两个参数，在这种情况下，将生效相应套接字选项的系统默认设置。例如， `so_keepalive=30m::10` 将空闲超时（ `TCP_KEEPIDLE` ）设置为 30 分钟，将探测间隔（ `TCP_KEEPINTVL` ）保留为其系统默认值，并将探测计数（ `TCP_KEEPCNT` ）设置为 10 个探测。

Example:

​	​示例：

```
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024;
```

### location 位置

Syntax:`location [ = | ~ | ~* | ^~ ] uri { ... }` `location @name { ... }`

​	​语法： `location [ = | ~ | ~* | ^~ ] uri { ... }` `location @name { ... }`

Default: —

​	​默认值：—

Context: `server`, `location`

​	​上下文： `server` 、 `location`

Sets configuration depending on a request URI.

​	​根据请求 URI 设置配置。

The matching is performed against a normalized URI, after decoding the text encoded in the “`%XX`” form, resolving references to relative path components “`.`” and “`..`”, and possible [compression](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#merge_slashes) of two or more adjacent slashes into a single slash.

​	​匹配针对规范化 URI 执行，在解码以 “ `%XX` ” 形式编码的文本、解析对相对路径组件 “ `.` ” 和 “ `..` ” 的引用，以及可能将两个或更多相邻斜杠压缩为单个斜杠之后。

A location can either be defined by a prefix string, or by a regular expression. Regular expressions are specified with the preceding “`~*`” modifier (for case-insensitive matching), or the “`~`” modifier (for case-sensitive matching). To find location matching a given request, nginx first checks locations defined using the prefix strings (prefix locations). Among them, the location with the longest matching prefix is selected and remembered. Then regular expressions are checked, in the order of their appearance in the configuration file. The search of regular expressions terminates on the first match, and the corresponding configuration is used. If no match with a regular expression is found then the configuration of the prefix location remembered earlier is used.

​	​位置可以通过前缀字符串或正则表达式定义。正则表达式使用前导 “ `~*` ” 修饰符（用于不区分大小写的匹配）或 “ `~` ” 修饰符（用于区分大小写的匹配）指定。要查找与给定请求匹配的位置，nginx 首先检查使用前缀字符串（前缀位置）定义的位置。在这些位置中，选择匹配前缀最长的位置并记住它。然后按其在配置文件中出现的顺序检查正则表达式。正则表达式的搜索在第一次匹配时终止，并使用相应的配置。如果未找到与正则表达式的匹配，则使用先前记住的前缀位置的配置。

`location` blocks can be nested, with some exceptions mentioned below.

​	​ `location` 块可以嵌套，但有一些例外情况，如下所述。

For case-insensitive operating systems such as macOS and Cygwin, matching with prefix strings ignores a case (0.7.7). However, comparison is limited to one-byte locales.

​	​对于 macOS 和 Cygwin 等不区分大小写的操作系统，与前缀字符串匹配时会忽略大小写（0.7.7）。但是，比较仅限于单字节区域设置。

Regular expressions can contain captures (0.7.40) that can later be used in other directives.

​	​正则表达式可以包含捕获（0.7.40），这些捕获稍后可以在其他指令中使用。

If the longest matching prefix location has the “`^~`” modifier then regular expressions are not checked.

​	​如果最长的匹配前缀位置具有“ `^~` ”修饰符，则不会检查正则表达式。

Also, using the “`=`” modifier it is possible to define an exact match of URI and location. If an exact match is found, the search terminates. For example, if a “`/`” request happens frequently, defining “`location = /`” will speed up the processing of these requests, as search terminates right after the first comparison. Such a location cannot obviously contain nested locations.

​	​此外，使用“ `=` ”修饰符可以定义 URI 和位置的确切匹配。如果找到确切匹配，则搜索终止。例如，如果“ `/` ”请求经常发生，则定义“ `location = /` ”将加快这些请求的处理速度，因为搜索在第一次比较后立即终止。显然，这样的位置不能包含嵌套位置。

In versions from 0.7.1 to 0.8.41, if a request matched the prefix location without the “`=`” and “`^~`” modifiers, the search also terminated and regular expressions were not checked.

​	​在 0.7.1 至 0.8.41 版本中，如果请求与不带“ `=` ”和“ `^~` ”修饰符的前缀位置匹配，则搜索也会终止，并且不会检查正则表达式。

Let’s illustrate the above by an example:

​	​我们通过一个示例来说明以上内容：

```
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

The “`/`” request will match configuration A, the “`/index.html`” request will match configuration B, the “`/documents/document.html`” request will match configuration C, the “`/images/1.gif`” request will match configuration D, and the “`/documents/1.jpg`” request will match configuration E.

​	​“ `/` ” 请求将匹配配置 A，“ `/index.html` ” 请求将匹配配置 B，“ `/documents/document.html` ” 请求将匹配配置 C，“ `/images/1.gif` ” 请求将匹配配置 D，而 “ `/documents/1.jpg` ” 请求将匹配配置 E。

The “`@`” prefix defines a named location. Such a location is not used for a regular request processing, but instead used for request redirection. They cannot be nested, and cannot contain nested locations.

​	​“ `@` ” 前缀定义了一个命名位置。此类位置不用于常规请求处理，而是用于请求重定向。它们不能嵌套，也不能包含嵌套位置。

If a location is defined by a prefix string that ends with the slash character, and requests are processed by one of [proxy_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_proxy_module/#proxy_pass), [fastcgi_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_fastcgi_module/#fastcgi_pass), [uwsgi_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_uwsgi_module/#uwsgi_pass), [scgi_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_scgi_module/#scgi_pass), [memcached_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_memcached_module/#memcached_pass), or [grpc_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_grpc_module/#grpc_pass), then the special processing is performed. In response to a request with URI equal to this string, but without the trailing slash, a permanent redirect with the code 301 will be returned to the requested URI with the slash appended. If this is not desired, an exact match of the URI and location could be defined like this:

​	​如果一个位置由以斜杠字符结尾的前缀字符串定义，并且请求由 proxy_pass、fastcgi_pass、uwsgi_pass、scgi_pass、memcached_pass 或 grpc_pass 之一处理，那么将执行特殊处理。对于 URI 等于此字符串但没有尾部斜杠的请求，将返回一个带有代码 301 的永久重定向到附加了斜杠的请求 URI。如果不需要这样做，则可以像这样定义 URI 和位置的确切匹配：

```
location /user/ {
    proxy_pass http://user.example.com;
}

location = /user {
    proxy_pass http://login.example.com;
}
```

### log_not_found

Syntax: `log_not_found on | off;`

​	​语法： `log_not_found on | off;`

Default: `log_not_found on;`

​	​默认值： `log_not_found on;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Enables or disables logging of errors about not found files into [error_log](https://before80.github.io/nginx_docs/mod_ref/ngx_core_module/#error_log).

​	​启用或禁用将有关未找到文件的错误记录到 error_log 中。

### log_subrequest

Syntax: `log_subrequest on | off;`

​	​语法： `log_subrequest on | off;`

Default: `log_subrequest off;`

​	​默认值： `log_subrequest off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Enables or disables logging of subrequests into [access_log](https://before80.github.io/nginx_docs/mod_ref/ngx_http_log_module/#access_log).

​	​启用或禁用将子请求记录到 access_log 中。

### max_ranges

Syntax: `max_ranges number;`

​	​语法： `max_ranges number;`

Default: —

​	​默认值：—

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

This directive appeared in version 1.1.2.

​	​此指令出现在 1.1.2 版本中。

Limits the maximum allowed number of ranges in byte-range requests. Requests that exceed the limit are processed as if there were no byte ranges specified. By default, the number of ranges is not limited. The zero value disables the byte-range support completely.

​	​限制字节范围请求中允许的最大范围数。超出限制的请求将被处理为未指定字节范围。默认情况下，范围数不受限制。零值将完全禁用字节范围支持。

### merge_slashes

Syntax:`merge_slashes on | off;`

​	​语法： `merge_slashes on | off;`

Default: `merge_slashes on;`

​	​默认值： `merge_slashes on;`

Context: `http`, `server`

​	​上下文： `http` ， `server`

Enables or disables compression of two or more adjacent slashes in a URI into a single slash.

​	​启用或禁用 URI 中两个或多个相邻斜杠压缩为单个斜杠。

Note that compression is essential for the correct matching of prefix string and regular expression locations. Without it, the “`//scripts/one.php`” request would not match

​	​请注意，压缩对于前缀字符串和正则表达式位置的正确匹配至关重要。如果没有它，“ `//scripts/one.php` ”请求将不匹配

```
location /scripts/ {
    ...
}
```

and might be processed as a static file. So it gets converted to “`/scripts/one.php`”.

​	​并且可能被视为静态文件。因此它被转换为“ `/scripts/one.php` ”。

Turning the compression `off` can become necessary if a URI contains base64-encoded names, since base64 uses the “`/`” character internally. However, for security considerations, it is better to avoid turning the compression off.

​	​如果 URI 包含 base64 编码的名称，则关闭压缩 `off` 可能变得有必要，因为 base64 在内部使用“ `/` ”字符。但是，出于安全考虑，最好避免关闭压缩。

If the directive is specified on the [server](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server) level, the value from the default server can be used. Details are provided in the “[Virtual server selection](https://before80.github.io/nginx_docs/introduction/serverNames/#虚拟服务器选择---virtual-server-selection)” section.

​	​如果在服务器级别指定该指令，则可以使用来自默认服务器的值。详细信息在“虚拟服务器选择”部分中提供。

### msie_padding

Syntax: `msie_padding on | off;`

​	​语法： `msie_padding on | off;`

Default: `msie_padding on;`

​	​默认值： `msie_padding on;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Enables or disables adding comments to responses for MSIE clients with status greater than 400 to increase the response size to 512 bytes.

​	​启用或禁用为状态大于 400 的 MSIE 客户端向响应中添加注释，以将响应大小增加到 512 字节。

### msie_refresh

Syntax: `msie_refresh on | off;`

​	​语法： `msie_refresh on | off;`

Default: `msie_refresh off;`

​	​默认值： `msie_refresh off;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Enables or disables issuing refreshes instead of redirects for MSIE clients.

​	​启用或禁用为 MSIE 客户端发出刷新而不是重定向。

### open_file_cache

Syntax:`open_file_cache off;` `open_file_cache max=N [inactive=time];`

​	​语法： `open_file_cache off;` `open_file_cache max=N [inactive=time];`

Default: `open_file_cache off;`

​	​默认值： `open_file_cache off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Configures a cache that can store:

​	​配置一个可以存储以下内容的缓存：

- open file descriptors, their sizes and modification times;

  ​	​打开的文件描述符、它们的大小和修改时间；

- information on existence of directories;

  ​	​目录存在的信息；

- file lookup errors, such as “file not found”, “no read permission”, and so on.

  ​	​文件查找错误，例如“文件未找到”、“没有读取权限”等。

  Caching of errors should be enabled separately by the [open_file_cache_errors](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#open_file_cache_errors) directive.

  ​	​错误缓存应通过 open_file_cache_errors 指令单独启用。

The directive has the following parameters:

​	​该指令具有以下参数：

- `max`

  sets the maximum number of elements in the cache; on cache overflow the least recently used (LRU) elements are removed;

  ​	​设置缓存中的最大元素数；当缓存溢出时，将删除最近最少使用 (LRU) 的元素；

- `inactive`

  defines a time after which an element is removed from the cache if it has not been accessed during this time; by default, it is 60 seconds;

  ​	​定义在该时间段内未访问元素后将其从缓存中删除的时间；默认情况下，为 60 秒；

- `off`

  disables the cache.

  ​	​禁用缓存。

Example:

​	​示例：

```
open_file_cache          max=1000 inactive=20s;
open_file_cache_valid    30s;
open_file_cache_min_uses 2;
open_file_cache_errors   on;
```

### open_file_cache_errors

Syntax:`open_file_cache_errors on | off;`

​	​语法： `open_file_cache_errors on | off;`

Default: `open_file_cache_errors off;`

​	​默认值： `open_file_cache_errors off;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Enables or disables caching of file lookup errors by [open_file_cache](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#open_file_cache).

​	​启用或禁用 open_file_cache 对文件查找错误进行缓存。

### open_file_cache_min_uses

Syntax:`open_file_cache_min_uses number;`

​	​语法： `open_file_cache_min_uses number;`

Default: `open_file_cache_min_uses 1;`

​	​默认值： `open_file_cache_min_uses 1;`

Context: `http`, `server`, `location`

​	​上下文: `http` , `server` , `location`

Sets the minimum `number` of file accesses during the period configured by the `inactive` parameter of the [open_file_cache](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#open_file_cache) directive, required for a file descriptor to remain open in the cache.

​	​设置 open_file_cache 指令的 `inactive` 参数配置期间文件访问的最小 `number` ，文件描述符在缓存中保持打开状态所必需。

### open_file_cache_valid

Syntax:`open_file_cache_valid time;`

​	​语法: `open_file_cache_valid time;`

Default: `open_file_cache_valid 60s;`

​	​默认值: `open_file_cache_valid 60s;`

Context: `http`, `server`, `location`

​	​上下文: `http` , `server` , `location`

Sets a time after which [open_file_cache](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#open_file_cache) elements should be validated.

​	​设置 open_file_cache 元素应被验证的时间。

### output_buffers

Syntax:`output_buffers number size;`

​	​语法: `output_buffers number size;`

Default: `output_buffers 2 32k;`

​	​默认值: `output_buffers 2 32k;`

Context: `http`, `server`, `location`

​	​上下文: `http` , `server` , `location`

Sets the `number` and `size` of the buffers used for reading a response from a disk.

​	​设置用于从磁盘读取响应的缓冲区的 `number` 和 `size` 。

Prior to version 1.9.5, the default value was 1 32k.

​	​在 1.9.5 版本之前，默认值为 1 32k。

### port_in_redirect

Syntax:`port_in_redirect on | off;`

​	​语法: `port_in_redirect on | off;`

Default: `port_in_redirect on;`

​	​默认值: `port_in_redirect on;`

Context: `http`, `server`, `location`

​	​上下文: `http` , `server` , `location`

Enables or disables specifying the port in [absolute](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#absolute_redirect) redirects issued by nginx.

​	​启用或禁用在 nginx 发出的绝对重定向中指定端口。

The use of the primary server name in redirects is controlled by the [server_name_in_redirect](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server_name_in_redirect) directive.

​	​重定向中使用主服务器名称受 server_name_in_redirect 指令控制。

### postpone_output

Syntax: `postpone_output size;`

​	​语法： `postpone_output size;`

Default: `postpone_output 1460;`

​	​默认值： `postpone_output 1460;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

If possible, the transmission of client data will be postponed until nginx has at least `size` bytes of data to send. The zero value disables postponing data transmission.

​	​如果可能，客户端数据传输将被推迟，直到 nginx 至少有 `size` 字节的数据要发送。零值禁用推迟数据传输。

### read_ahead

Syntax: `read_ahead size;`

​	​语法： `read_ahead size;`

Default: `read_ahead 0;`

​	​默认值： `read_ahead 0;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Sets the amount of pre-reading for the kernel when working with file.

​	​设置使用文件时内核的预读数量。

On Linux, the `posix_fadvise(0, 0, 0, POSIX_FADV_SEQUENTIAL)` system call is used, and so the `size` parameter is ignored.

​	​在 Linux 上，使用 `posix_fadvise(0, 0, 0, POSIX_FADV_SEQUENTIAL)` 系统调用，因此忽略 `size` 参数。

On FreeBSD, the `fcntl(O_READAHEAD,` `size``)` system call, supported since FreeBSD 9.0-CURRENT, is used. FreeBSD 7 has to be [patched](http://sysoev.ru/freebsd/patch.readahead.txt).

​	​在 FreeBSD 上，使用 `fcntl(O_READAHEAD,` `size``)` 系统调用，自 FreeBSD 9.0-CURRENT 起支持。FreeBSD 7 必须打补丁。

### recursive_error_pages

Syntax:`recursive_error_pages on | off;`

​	​语法： `recursive_error_pages on | off;`

Default: `recursive_error_pages off;`

​	​默认值： `recursive_error_pages off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Enables or disables doing several redirects using the [error_page](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#error_page) directive. The number of such redirects is [limited](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#internal).

​	​启用或禁用使用 error_page 指令进行多次重定向。此类重定向的数量有限。

### request_pool_size

Syntax:`request_pool_size size;`

​	​语法： `request_pool_size size;`

Default: `request_pool_size 4k;`

​	​默认值： `request_pool_size 4k;`

Context: `http`, `server`

​	​上下文： `http` , `server`

Allows accurate tuning of per-request memory allocations. This directive has minimal impact on performance and should not generally be used.

​	​允许精确调整每个请求的内存分配。此指令对性能的影响最小，通常不应使用。

### reset_timedout_connection

Syntax:`reset_timedout_connection on | off;`

​	​语法： `reset_timedout_connection on | off;`

Default: `reset_timedout_connection off;`

​	​默认值： `reset_timedout_connection off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Enables or disables resetting timed out connections and connections [closed](https://before80.github.io/nginx_docs/mod_ref/ngx_http_rewrite_module/#return) with the non-standard code 444 (1.15.2). The reset is performed as follows. Before closing a socket, the `SO_LINGER` option is set on it with a timeout value of 0. When the socket is closed, TCP RST is sent to the client, and all memory occupied by this socket is released. This helps avoid keeping an already closed socket with filled buffers in a FIN_WAIT1 state for a long time.

​	​启用或禁用重置超时连接和使用非标准代码 444（1.15.2）关闭的连接。重置执行如下。在关闭套接字之前， `SO_LINGER` 选项在其中设置，超时值为 0。当套接字关闭时，TCP RST 会发送到客户端，并且此套接字占用的所有内存都会释放。这有助于避免长时间将已关闭的套接字与已填充缓冲区保持在 FIN_WAIT1 状态。

It should be noted that timed out keep-alive connections are closed normally.

​	​应当注意，超时保持活动连接会正常关闭。

### resolver 解析器

Syntax:`resolver address ... [valid=time] [ipv4=on|off] [ipv6=on|off] [status_zone=zone];`

​	​语法： `resolver address ... [valid=time] [ipv4=on|off] [ipv6=on|off] [status_zone=zone];`

Default: —

​	​默认值：—

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Configures name servers used to resolve names of upstream servers into addresses, for example:

​	​配置用于将上游服务器的名称解析为地址的名称服务器，例如：

```
resolver 127.0.0.1 [::1]:5353;
```

The address can be specified as a domain name or IP address, with an optional port (1.3.1, 1.2.2). If port is not specified, the port 53 is used. Name servers are queried in a round-robin fashion.

​	​地址可以指定为域名或 IP 地址，并带有可选端口（1.3.1、1.2.2）。如果未指定端口，则使用端口 53。名称服务器将以循环方式进行查询。

Before version 1.1.7, only a single name server could be configured. Specifying name servers using IPv6 addresses is supported starting from versions 1.3.1 and 1.2.2.

​	​在 1.1.7 版本之前，只能配置一个名称服务器。从 1.3.1 和 1.2.2 版本开始，支持使用 IPv6 地址指定名称服务器。

By default, nginx will look up both IPv4 and IPv6 addresses while resolving. If looking up of IPv4 or IPv6 addresses is not desired, the `ipv4=off` (1.23.1) or the `ipv6=off` parameter can be specified.

​	​默认情况下，nginx 在解析时将同时查找 IPv4 和 IPv6 地址。如果不需要查找 IPv4 或 IPv6 地址，则可以指定 `ipv4=off` （1.23.1）或 `ipv6=off` 参数。

Resolving of names into IPv6 addresses is supported starting from version 1.5.8.

​	​从 1.5.8 版本开始，支持将名称解析为 IPv6 地址。

By default, nginx caches answers using the TTL value of a response. An optional `valid` parameter allows overriding it:

​	​默认情况下，nginx 使用响应的 TTL 值缓存答案。可选的 `valid` 参数允许覆盖它：

```
resolver 127.0.0.1 [::1]:5353 valid=30s;
```

Before version 1.1.9, tuning of caching time was not possible, and nginx always cached answers for the duration of 5 minutes.

​	​在 1.1.9 版本之前，无法调整缓存时间，nginx 始终将答案缓存 5 分钟。

To prevent DNS spoofing, it is recommended configuring DNS servers in a properly secured trusted local network.

​	​为防止 DNS 欺骗，建议在适当安全受信任的本地网络中配置 DNS 服务器。

The optional `status_zone` parameter (1.17.1) enables [collection](https://before80.github.io/nginx_docs/mod_ref/ngx_http_api_module/#resolvers_) of DNS server statistics of requests and responses in the specified `zone`. The parameter is available as part of our [commercial subscription](http://nginx.com/products/).

​	​可选 `status_zone` 参数 (1.17.1) 启用在指定的 `zone` 中收集 DNS 服务器请求和响应的统计信息。该参数作为我们的商业订阅的一部分提供。

### resolver_timeout

Syntax: `resolver_timeout time;`

​	​语法: `resolver_timeout time;`

Default: `resolver_timeout 30s;`

​	​默认值: `resolver_timeout 30s;`

Context: `http`, `server`, `location`

​	​上下文: `http` , `server` , `location`

Sets a timeout for name resolution, for example:

​	​设置名称解析超时，例如：

```
resolver_timeout 5s;
```

### root

Syntax: `root path;`

​	​语法: `root path;`

Default: `root html;`

​	​默认值: `root html;`

Context: `http`, `server`, `location`, `if in location`

​	​上下文: `http` , `server` , `location` , `if in location`

Sets the root directory for requests. For example, with the following configuration

​	​设置请求的根目录。例如，使用以下配置

```
location /i/ {
    root /data/w3;
}
```

The `/data/w3/i/top.gif` file will be sent in response to the “`/i/top.gif`” request.

​	​ `/data/w3/i/top.gif` 文件将作为对 “ `/i/top.gif` ” 请求的响应发送。

The `path` value can contain variables, except `$document_root` and `$realpath_root`.

​	​ `path` 值可以包含变量，但 `$document_root` 和 `$realpath_root` 除外。

A path to the file is constructed by merely adding a URI to the value of the `root` directive. If a URI has to be modified, the [alias](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#alias) directive should be used.

​	​通过简单地将 URI 添加到 `root` 指令的值来构造到该文件的路径。如果必须修改 URI，则应使用别名指令。

### satisfy 满足

Syntax: `satisfy all | any;`

​	​语法： `satisfy all | any;`

Default: `satisfy all;`

​	​默认值： `satisfy all;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Allows access if all (`all`) or at least one (`any`) of the [ngx_http_access_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_access_module), [ngx_http_auth_basic_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_auth_basic_module), [ngx_http_auth_request_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_auth_request_module), or [ngx_http_auth_jwt_module](https://before80.github.io/nginx_docs/mod_ref/ngx_http_auth_jwt_module) modules allow access.

​	​如果所有（ `all` ）或至少一个（ `any` ）ngx_http_access_module、ngx_http_auth_basic_module、ngx_http_auth_request_module 或 ngx_http_auth_jwt_module 模块允许访问，则允许访问。

Example:

​	​示例：

```
location / {
    satisfy any;

    allow 192.168.1.0/32;
    deny  all;

    auth_basic           "closed site";
    auth_basic_user_file conf/htpasswd;
}
```

### send_lowat

Syntax: `send_lowat size;`

​	​语法： `send_lowat size;`

Default: `send_lowat 0;`

​	​默认值： `send_lowat 0;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

If the directive is set to a non-zero value, nginx will try to minimize the number of send operations on client sockets by using either `NOTE_LOWAT` flag of the [kqueue](https://before80.github.io/nginx_docs/introduction/connectionProcessingMethods/#kqueue) method or the `SO_SNDLOWAT` socket option. In both cases the specified `size` is used.

​	​如果该指令被设置为非零值，nginx 将尝试通过使用 kqueue 方法的 `NOTE_LOWAT` 标志或 `SO_SNDLOWAT` 套接字选项来最大程度地减少客户端套接字上的发送操作次数。在这两种情况下，都会使用指定的 `size` 。

This directive is ignored on Linux, Solaris, and Windows.

​	​在 Linux、Solaris 和 Windows 上忽略此指令。

### send_timeout

Syntax: `send_timeout time;`

​	​语法： `send_timeout time;`

Default: `send_timeout 60s;`

​	​默认值： `send_timeout 60s;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Sets a timeout for transmitting a response to the client. The timeout is set only between two successive write operations, not for the transmission of the whole response. If the client does not receive anything within this time, the connection is closed.

​	​设置向客户端传输响应的超时时间。超时时间仅在两次连续的写入操作之间设置，而不是针对整个响应的传输。如果客户端在此时间内未收到任何内容，则连接将被关闭。

### sendfile

Syntax: `sendfile on | off;`

​	​语法： `sendfile on | off;`

Default: `sendfile off;`

​	​默认值： `sendfile off;`

Context: `http`, `server`, `location`, `if in location`

​	​上下文： `http` , `server` , `location` , `if in location`

Enables or disables the use of `sendfile()`.

​	​启用或禁用 `sendfile()` 的使用。

Starting from nginx 0.8.12 and FreeBSD 5.2.1, [aio](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#aio) can be used to pre-load data for `sendfile()`:

​	​从 nginx 0.8.12 和 FreeBSD 5.2.1 开始，aio 可用于预加载 `sendfile()` 的数据：

```
location /video/ {
    sendfile       on;
    tcp_nopush     on;
    aio            on;
}
```

In this configuration, `sendfile()` is called with the `SF_NODISKIO` flag which causes it not to block on disk I/O, but, instead, report back that the data are not in memory. nginx then initiates an asynchronous data load by reading one byte. On the first read, the FreeBSD kernel loads the first 128K bytes of a file into memory, although next reads will only load data in 16K chunks. This can be changed using the [read_ahead](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#read_ahead) directive.

​	​在此配置中， `sendfile()` 使用 `SF_NODISKIO` 标志调用，这会导致它不会阻塞磁盘 I/O，而是报告数据不在内存中。然后，nginx 通过读取一个字节来启动异步数据加载。在第一次读取时，FreeBSD 内核将文件的第一个 128K 字节加载到内存中，尽管下一次读取只会加载 16K 块中的数据。可以使用 read_ahead 指令更改此设置。

Before version 1.7.11, pre-loading could be enabled with `aio sendfile;`.

​	​在 1.7.11 版本之前，可以使用 `aio sendfile;` 启用预加载。

### sendfile_max_chunk

Syntax: `sendfile_max_chunk size;`

​	​语法： `sendfile_max_chunk size;`

Default: `sendfile_max_chunk 2m;`

​	​默认值： `sendfile_max_chunk 2m;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Limits the amount of data that can be transferred in a single `sendfile()` call. Without the limit, one fast connection may seize the worker process entirely.

​	​限制在单个 `sendfile()` 调用中可以传输的数据量。如果没有限制，一个快速连接可能会完全占用工作进程。

Prior to version 1.21.4, by default there was no limit.

​	​在 1.21.4 版本之前，默认情况下没有限制。

### server 服务器

Syntax:`server { ... }`

​	​语法： `server { ... }`

Default: —

​	​默认值：—

Context: `http`

​	​上下文： `http`

Sets configuration for a virtual server. There is no clear separation between IP-based (based on the IP address) and name-based (based on the “Host” request header field) virtual servers. Instead, the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directives describe all addresses and ports that should accept connections for the server, and the [server_name](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server_name) directive lists all server names. Example configurations are provided in the “[How nginx processes a request](https://before80.github.io/nginx_docs/introduction/howNginxProcessesARequest/)” document.

​	​设置虚拟服务器的配置。基于 IP（基于 IP 地址）和基于名称（基于“Host”请求头字段）的虚拟服务器之间没有明确的分隔。相反，listen 指令描述了所有应该接受服务器连接的地址和端口，而 server_name 指令列出了所有服务器名称。示例配置在“nginx 如何处理请求”文档中提供。

### server_name

Syntax:`server_name name ...;`

​	​语法： `server_name name ...;`

Default: `server_name "";`

​	​默认值： `server_name "";`

Context: `server`

​	​上下文： `server`

Sets names of a virtual server, for example:

​	​设置虚拟服务器的名称，例如：

```
server {
    server_name example.com www.example.com;
}
```

The first name becomes the primary server name.

​	​第一个名称将成为主服务器名称。

Server names can include an asterisk (“*”) replacing the first or last part of a name:

​	​服务器名称可以包含一个星号（“*”），替换名称的第一部分或最后一部分：

```
server {
    server_name example.com *.example.com www.example.*;
}
```

Such names are called wildcard names.

​	​这样的名称称为通配符名称。

The first two of the names mentioned above can be combined in one:

​	​上面提到的前两个名称可以组合成一个：

```
server {
    server_name .example.com;
}
```

It is also possible to use regular expressions in server names, preceding the name with a tilde (“`~`”):

​	​也可以在服务器名称中使用正则表达式，在名称前加上波浪号（“ `~` ”）：

```
server {
    server_name www.example.com ~^www\d+\.example\.com$;
}
```

Regular expressions can contain captures (0.7.40) that can later be used in other directives:

​	​正则表达式可以包含捕获（0.7.40），稍后可以在其他指令中使用：

```
server {
    server_name ~^(www\.)?(.+)$;

    location / {
        root /sites/$2;
    }
}

server {
    server_name _;

    location / {
        root /sites/default;
    }
}
```

Named captures in regular expressions create variables (0.8.25) that can later be used in other directives:

​	​正则表达式中的命名捕获创建变量（0.8.25），稍后可以在其他指令中使用：

```
server {
    server_name ~^(www\.)?(?<domain>.+)$;

    location / {
        root /sites/$domain;
    }
}

server {
    server_name _;

    location / {
        root /sites/default;
    }
}
```

If the directive’s parameter is set to “`$hostname`” (0.9.4), the machine’s hostname is inserted.

​	​如果指令的参数设置为“ `$hostname` ”（0.9.4），则插入机器的主机名。

It is also possible to specify an empty server name (0.7.11):

​	​也可以指定一个空服务器名称（0.7.11）：

```
server {
    server_name www.example.com "";
}
```

It allows this server to process requests without the “Host” header field — instead of the default server — for the given address:port pair. This is the default setting.

​	​它允许此服务器为给定的地址：端口对处理没有“Host”头字段的请求，而不是默认服务器。这是默认设置。

Before 0.8.48, the machine’s hostname was used by default.

​	​在 0.8.48 之前，默认使用机器的主机名。

During searching for a virtual server by name, if the name matches more than one of the specified variants, (e.g. both a wildcard name and regular expression match), the first matching variant will be chosen, in the following order of priority:

​	​在按名称搜索虚拟服务器期间，如果名称与多个指定变体匹配（例如，通配符名称和正则表达式匹配），则将选择第一个匹配变体，按以下优先级顺序：

1. the exact name
   确切名称
2. the longest wildcard name starting with an asterisk, e.g. “`*.example.com`”
   以星号开头的最长通配符名称，例如“ `*.example.com` ”
3. the longest wildcard name ending with an asterisk, e.g. “`mail.*`”
   以星号结尾的最长通配符名称，例如“ `mail.*` ”
4. the first matching regular expression (in order of appearance in the configuration file)
   第一个匹配的正则表达式（按在配置文件中出现的顺序）

Detailed description of server names is provided in a separate [Server names](https://before80.github.io/nginx_docs/introduction/serverNames/) document.

​	​服务器名称的详细说明在单独的服务器名称文档中提供。

### server_name_in_redirect

Syntax:`server_name_in_redirect on | off;`

​	​语法： `server_name_in_redirect on | off;`

Default: `server_name_in_redirect off;`

​	​默认值： `server_name_in_redirect off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Enables or disables the use of the primary server name, specified by the [server_name](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server_name) directive, in [absolute](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#absolute_redirect) redirects issued by nginx. When the use of the primary server name is disabled, the name from the “Host” request header field is used. If this field is not present, the IP address of the server is used.

​	​启用或禁用 nginx 发出的绝对重定向中由 server_name 指令指定的服务器主名称的使用。禁用服务器主名称的使用时，将使用“Host”请求头字段中的名称。如果此字段不存在，则使用服务器的 IP 地址。

The use of a port in redirects is controlled by the [port_in_redirect](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#port_in_redirect) directive.

​	​重定向中端口的使用由 port_in_redirect 指令控制。

### server_names_hash_bucket_size

Syntax: `server_names_hash_bucket_size size;`

​	​语法： `server_names_hash_bucket_size size;`

Default: `server_names_hash_bucket_size 32|64|128;`

​	​默认值： `server_names_hash_bucket_size 32|64|128;`

Context: `http`

​	​上下文： `http`

Sets the bucket size for the server names hash tables. The default value depends on the size of the processor’s cache line. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	​设置服务器名称哈希表的存储桶大小。默认值取决于处理器的缓存行大小。哈希表设置的详细信息在单独的文档中提供。

### server_names_hash_max_size

Syntax:`server_names_hash_max_size size;`

​	​语法： `server_names_hash_max_size size;`

Default: `server_names_hash_max_size 512;`

​	​默认值： `server_names_hash_max_size 512;`

Context: `http`

​	​上下文： `http`

Sets the maximum `size` of the server names hash tables. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	​设置服务器名称哈希表的最大 `size` 。哈希表设置的详细信息在单独的文档中提供。

### server_tokens

Syntax:`server_tokens on | off | build | string;`

​	​语法： `server_tokens on | off | build | string;`

Default: `server_tokens on;`

​	​默认值： `server_tokens on;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

Enables or disables emitting nginx version on error pages and in the “Server” response header field.

​	​启用或禁用在错误页面和“服务器”响应头字段中发出 nginx 版本。

The `build` parameter (1.11.10) enables emitting a [build name](https://nginx.org/en/docs/configure.html#build) along with nginx version.

​	​ `build` 参数 (1.11.10) 启用发出构建名称以及 nginx 版本。

Additionally, as part of our [commercial subscription](http://nginx.com/products/), starting from version 1.9.13 the signature on error pages and the “Server” response header field value can be set explicitly using the `string` with variables. An empty string disables the emission of the “Server” field.

​	​此外，作为我们商业订阅的一部分，从 1.9.13 版本开始，可以使用带有变量的 `string` 显式设置错误页面和“服务器”响应头字段值。空字符串禁用“服务器”字段的发射。

### subrequest_output_buffer_size

Syntax: `subrequest_output_buffer_size size;`

​	​语法： `subrequest_output_buffer_size size;`

Default: `subrequest_output_buffer_size 4k|8k;`

​	​默认值： `subrequest_output_buffer_size 4k|8k;`

Context: `http`, `server`, `location`

​	​上下文： `http` ， `server` ， `location`

This directive appeared in version 1.13.10.

​	​此指令出现在 1.13.10 版本中。

Sets the `size` of the buffer used for storing the response body of a subrequest. By default, the buffer size is equal to one memory page. This is either 4K or 8K, depending on a platform. It can be made smaller, however.

​	​设置用于存储子请求的响应正文的缓冲区的 `size` 。默认情况下，缓冲区大小等于一个内存页。根据平台的不同，它可能是 4K 或 8K。但是，可以将其缩小。

The directive is applicable only for subrequests with response bodies saved into memory. For example, such subrequests are created by [SSI](https://before80.github.io/nginx_docs/mod_ref/ngx_http_ssi_module/#ssi_include_set).

​	​该指令仅适用于将响应体保存到内存中的子请求。例如，此类子请求由 SSI 创建。

### tcp_nodelay

Syntax: `tcp_nodelay on | off;`

​	​语法： `tcp_nodelay on | off;`

Default: `tcp_nodelay on;`

​	​默认值： `tcp_nodelay on;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Enables or disables the use of the `TCP_NODELAY` option. The option is enabled when a connection is transitioned into the keep-alive state. Additionally, it is enabled on SSL connections, for unbuffered proxying, and for [WebSocket](https://nginx.org/en/docs/http/websocket.html) proxying.

​	​启用或禁用 `TCP_NODELAY` 选项。当连接转换到保持活动状态时，将启用该选项。此外，它在 SSL 连接、无缓冲代理和 WebSocket 代理中启用。

### tcp_nopush

Syntax: `tcp_nopush on | off;`

​	​语法： `tcp_nopush on | off;`

Default: `tcp_nopush off;`

​	​默认值： `tcp_nopush off;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Enables or disables the use of the `TCP_NOPUSH` socket option on FreeBSD or the `TCP_CORK` socket option on Linux. The options are enabled only when [sendfile](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#sendfile) is used. Enabling the option allows

​	​启用或禁用在 FreeBSD 上使用 `TCP_NOPUSH` 套接字选项或在 Linux 上使用 `TCP_CORK` 套接字选项。仅在使用 sendfile 时启用这些选项。启用该选项允许在 Linux 和 FreeBSD 4.* 上在一个数据包中发送响应头和文件开头；

- sending the response header and the beginning of a file in one packet, on Linux and FreeBSD 4.*;
  以完整数据包发送文件。
- sending a file in full packets.
  try_files

### try_files 语法：<b0></b0> <b1></b1>

Syntax:`try_files file ... uri;` `try_files file ... =code;`

​	​默认值：—

Default: —

​	​上下文：<b0></b0> , <b1></b1>

Context: `server`, `location`

​	​按指定顺序检查文件是否存在，并使用第一个找到的文件进行请求处理；处理在当前上下文中执行。文件的路径根据根目录和别名指令从 `server` 参数构建。可以通过在名称末尾指定斜杠来检查目录是否存在，例如“ `location` ”。如果未找到任何文件，则会对最后一个参数中指定的 进行内部重定向。例如：

Checks the existence of files in the specified order and uses the first found file for request processing; the processing is performed in the current context. The path to a file is constructed from the `file` parameter according to the [root](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#root) and [alias](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#alias) directives. It is possible to check directory’s existence by specifying a slash at the end of a name, e.g. “`$uri/`”. If none of the files were found, an internal redirect to the `uri` specified in the last parameter is made. For example:

​	​最后一个参数还可以指向命名位置，如下面的示例所示。从 0.7.51 版开始，最后一个参数还可以是 `file` ：

```
location /images/ {
    try_files $uri /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```

The last parameter can also point to a named location, as shown in examples below. Starting from version 0.7.51, the last parameter can also be a `code`:

​	​在代理 Mongrel 中的示例：

```
location / {
    try_files $uri $uri/index.html $uri.html =404;
}
```

Example in proxying Mongrel:

```
location / {
    try_files /system/maintenance.html
              $uri $uri/index.html $uri.html
              @mongrel;
}

location @mongrel {
    proxy_pass http://mongrel;
}
```

Example for Drupal/FastCGI:

​	​Drupal/FastCGI 示例：

```
location / {
    try_files $uri $uri/ @drupal;
}

location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME     $fastcgi_script_name;
    fastcgi_param QUERY_STRING    $args;

    ... other fastcgi_param's
}

location @drupal {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    fastcgi_param SCRIPT_NAME     /index.php;
    fastcgi_param QUERY_STRING    q=$uri&$args;

    ... other fastcgi_param's
}
```

In the following example,

​	​在以下示例中，

```
location / {
    try_files $uri $uri/ @drupal;
}
```

the `try_files` directive is equivalent to

​	​ `try_files` 指令等效于

```
location / {
    error_page 404 = @drupal;
    log_not_found off;
}
```

And here,

​	​在这里，

```
location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;

    ...
}
```

`try_files` checks the existence of the PHP file before passing the request to the FastCGI server.

​	​ `try_files` 在将请求传递到 FastCGI 服务器之前检查 PHP 文件是否存在。

Example for Wordpress and Joomla:

​	​Wordpress 和 Joomla 示例：

```
location / {
    try_files $uri $uri/ @wordpress;
}

location ~ \.php$ {
    try_files $uri @wordpress;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    ... other fastcgi_param's
}

location @wordpress {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    ... other fastcgi_param's
}
```

### types 类型

Syntax: `types { ... }`

​	​语法： `types { ... }`

Default: `types { text/html html; image/gif gif; image/jpeg jpg; }`

​	​默认值： `types { text/html html; image/gif gif; image/jpeg jpg; }`

Context: `http`, `server`, `location`

​	​上下文： `http` 、 `server` 、 `location`

Maps file name extensions to MIME types of responses. Extensions are case-insensitive. Several extensions can be mapped to one type, for example:

​	​将文件扩展名映射到响应的 MIME 类型。扩展名不区分大小写。多个扩展名可以映射到一个类型，例如：

```
types {
    application/octet-stream bin exe dll;
    application/octet-stream deb;
    application/octet-stream dmg;
}
```

A sufficiently full mapping table is distributed with nginx in the `conf/mime.types` file.

​	​nginx 在 `conf/mime.types` 文件中分发了一个足够完整的映射表。

To make a particular location emit the “`application/octet-stream`” MIME type for all requests, the following configuration can be used:

​	​要使特定位置对所有请求发出“ `application/octet-stream` ”MIME 类型，可以使用以下配置：

```
location /download/ {
    types        { }
    default_type application/octet-stream;
}
```

### types_hash_bucket_size

Syntax:`types_hash_bucket_size size;`

​	​语法： `types_hash_bucket_size size;`

Default: `types_hash_bucket_size 64;`

​	​默认值： `types_hash_bucket_size 64;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Sets the bucket size for the types hash tables. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	​设置类型哈希表的存储桶大小。哈希表设置的详细信息在单独的文档中提供。

Prior to version 1.5.13, the default value depended on the size of the processor’s cache line.

​	​在 1.5.13 版本之前，默认值取决于处理器的缓存行大小。

### types_hash_max_size

Syntax: `types_hash_max_size size;`

​	​语法： `types_hash_max_size size;`

Default: `types_hash_max_size 1024;`

​	​默认值： `types_hash_max_size 1024;`

Context: `http`, `server`, `location`

​	​上下文： `http` , `server` , `location`

Sets the maximum `size` of the types hash tables. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	​设置类型哈希表的最大 `size` 。哈希表设置的详细信息在单独的文档中提供。

### underscores_in_headers

Syntax:`underscores_in_headers on | off;`

​	​语法： `underscores_in_headers on | off;`

Default: `underscores_in_headers off;`

​	​默认值： `underscores_in_headers off;`

Context: `http`, `server`

​	​上下文： `http` , `server`

Enables or disables the use of underscores in client request header fields. When the use of underscores is disabled, request header fields whose names contain underscores are marked as invalid and become subject to the [ignore_invalid_headers](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#ignore_invalid_headers) directive.

​	​启用或禁用在客户端请求头字段中使用下划线。当禁用使用下划线时，名称中包含下划线的请求头字段将被标记为无效，并受 ignore_invalid_headers 指令约束。

If the directive is specified on the [server](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#server) level, the value from the default server can be used. Details are provided in the “[Virtual server selection](https://before80.github.io/nginx_docs/introduction/serverNames/#虚拟服务器选择---virtual-server-selection)” section.

​	如果在服务器级别指定指令，则可以使用默认服务器中的值。详细信息在“虚拟服务器选择”部分中提供。

### variables_hash_bucket_size

Syntax:`variables_hash_bucket_size size;`

​	​语法： `variables_hash_bucket_size size;`

Default: `variables_hash_bucket_size 64;`

​	​默认值： `variables_hash_bucket_size 64;`

Context: `http`

​	​上下文： `http`

Sets the bucket size for the variables hash table. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	​设置变量哈希表的分段大小。哈希表设置的详细信息在单独的文档中提供。

### variables_hash_max_size

Syntax:`variables_hash_max_size size;`

​	​语法： `variables_hash_max_size size;`

Default: `variables_hash_max_size 1024;`

​	​默认值： `variables_hash_max_size 1024;`

Context: `http`

​	​上下文： `http`

Sets the maximum `size` of the variables hash table. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	​设置变量哈希表的最大 `size` 。哈希表设置的详细信息在单独的文档中提供。

Prior to version 1.5.13, the default value was 512.

​	​在 1.5.13 版本之前，默认值为 512。

## Embedded Variables

​	​嵌入式变量

The `ngx_http_core_module` module supports embedded variables with names matching the Apache Server variables. First of all, these are variables representing client request header fields, such as `$http_user_agent`, `$http_cookie`, and so on. Also there are other variables:

​	​ `ngx_http_core_module` 模块支持名称与 Apache 服务器变量匹配的嵌入式变量。首先，这些变量表示客户端请求头字段，例如 `$http_user_agent` 、 `$http_cookie` 等。此外还有其他变量：

### `$arg_name`

  argument `name` in the request line

  ​	​请求行中的参数 `name`

### `$args`

  arguments in the request line

  ​	​请求行中的参数

### `$binary_remote_addr`

  client address in a binary form, value’s length is always 4 bytes for IPv4 addresses or 16 bytes for IPv6 addresses

  ​	​二进制形式的客户端地址，对于 IPv4 地址，值长度始终为 4 个字节，对于 IPv6 地址，值长度始终为 16 个字节

### `$body_bytes_sent`

  number of bytes sent to a client, not counting the response header; this variable is compatible with the “`%B`” parameter of the `mod_log_config` Apache module

  ​	​发送到客户端的字节数，不包括响应头；此变量与 `mod_log_config` Apache 模块的“ `%B` ”参数兼容

### `$bytes_sent`

  number of bytes sent to a client (1.3.8, 1.2.5)

  ​	​发送到客户端的字节数（1.3.8、1.2.5）

### `$connection`

  connection serial number (1.3.8, 1.2.5)

  ​	​连接序列号（1.3.8、1.2.5）

### `$connection_requests`

  current number of requests made through a connection (1.3.8, 1.2.5)

  ​	​通过连接进行的当前请求数（1.3.8、1.2.5）

### `$connection_time`

  connection time in seconds with a milliseconds resolution (1.19.10)

  ​	​以毫秒为单位的连接时间（1.19.10）

### `$content_length`

  “Content-Length” request header field

  ​	​“Content-Length” 请求头字段

### `$content_type`

  “Content-Type” request header field

  ​	​“Content-Type” 请求头字段

### `$cookie_` `name`

  the `name` cookie

  ​	​ `name` cookie

### `$document_root`

  [root](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#root) or [alias](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#alias) directive’s value for the current request

  ​	​当前请求的根或别名指令的值

### `$document_uri`

  same as `$uri`

  ​	​与 `$uri` 相同

### `$host`

  in this order of precedence: host name from the request line, or host name from the “Host” request header field, or the server name matching a request

  ​	​按以下优先级顺序：请求行中的主机名，或“Host”请求头字段中的主机名，或与请求匹配的服务器名

### `$hostname`

  host name

  ​	​主机名

### `$http_` `name`

  arbitrary request header field; the last part of a variable name is the field name converted to lower case with dashes replaced by underscores

  ​	​任意请求头字段；变量名的最后部分是字段名，已转换为小写，破折号替换为下划线

### `$https`

  “`on`” if connection operates in SSL mode, or an empty string otherwise

  ​	​如果连接以 SSL 模式运行，则为“ `on` ”，否则为空字符串

### `$is_args`

  “`?`” if a request line has arguments, or an empty string otherwise

  ​	​如果请求行有参数，则为“ `?` ”，否则为空字符串

### `$limit_rate`

  setting this variable enables response rate limiting; see [limit_rate](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#limit_rate)

  ​	​设置此变量可启用响应速率限制；请参阅 limit_rate

### `$msec`

  current time in seconds with the milliseconds resolution (1.3.9, 1.2.6)

  ​	​以毫秒为单位的当前时间（1.3.9、1.2.6）

### `$nginx_version`

  nginx version

  ​	​nginx 版本

### `$pid`

  PID of the worker process

  ​	​工作进程的 PID

### `$pipe`

  “`p`” if request was pipelined, “`.`” otherwise (1.3.12, 1.2.7)

  ​	​如果请求是流水线，则为“ `p` ”，否则为“ `.` ”（1.3.12、1.2.7）

### `$proxy_protocol_addr`

  client address from the PROXY protocol header (1.5.12)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directive.

  ​	​PROXY 协议头中的客户端地址（1.5.12）必须通过在 listen 指令中设置 `proxy_protocol` 参数来预先启用 PROXY 协议。

### `$proxy_protocol_port`

  client port from the PROXY protocol header (1.11.0)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directive.

  ​	​PROXY 协议头中的客户端端口（1.11.0）必须通过在 listen 指令中设置 `proxy_protocol` 参数来预先启用 PROXY 协议。

### `$proxy_protocol_server_addr`

  server address from the PROXY protocol header (1.17.6)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directive.

  ​	​PROXY 协议头中的服务器地址（1.17.6）必须通过在 listen 指令中设置 `proxy_protocol` 参数来预先启用 PROXY 协议。

### `$proxy_protocol_server_port`

  server port from the PROXY protocol header (1.17.6)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directive.

  ​	​PROXY 协议头中的服务器端口（1.17.6）必须通过在 listen 指令中设置 `proxy_protocol` 参数来预先启用 PROXY 协议。

### `$proxy_protocol_tlv_` `name`

  TLV from the PROXY Protocol header (1.23.2). The `name` can be a TLV type name or its numeric value. In the latter case, the value is hexadecimal and should be prefixed with `0x`:`$proxy_protocol_tlv_alpn $proxy_protocol_tlv_0x01 `SSL TLVs can also be accessed by TLV type name or its numeric value, both prefixed by `ssl_`:`$proxy_protocol_tlv_ssl_version $proxy_protocol_tlv_ssl_0x21 `The following TLV type names are supported:`alpn` (`0x01`) - upper layer protocol used over the connection`authority` (`0x02`) - host name value passed by the client`unique_id` (`0x05`) - unique connection id`netns` (`0x30`) - name of the namespace`ssl` (`0x20`) - binary SSL TLV structureThe following SSL TLV type names are supported:`ssl_version` (`0x21`) - SSL version used in client connection`ssl_cn` (`0x22`) - SSL certificate Common Name`ssl_cipher` (`0x23`) - name of the used cipher`ssl_sig_alg` (`0x24`) - algorithm used to sign the certificate`ssl_key_alg` (`0x25`) - public-key algorithmAlso, the following special SSL TLV type name is supported:`ssl_verify` - client SSL certificate verification result, `0` if the client presented a certificate and it was successfully verified, non-zero otherwise.The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directive.

  ​	​TLV来自PROXY协议头（1.23.2）。 `name` 可以是TLV类型名称或其数字值。 后一种情况下，该值是十六进制的，应以 `0x` 为前缀： `$proxy_protocol_tlv_alpn $proxy_protocol_tlv_0x01 `SSL TLV 还可以通过 TLV 类型名称或其数字值访问，两者都以 `ssl_` 为前缀： `$proxy_protocol_tlv_ssl_version $proxy_protocol_tlv_ssl_0x21 `支持以下 TLV 类型名称： `alpn` （ `0x01` ）- 连接上使用的上层协议 `authority` （ `0x02` ）- 客户端传递的主机名值 `unique_id` （ `0x05` ）- 唯一连接 ID `netns` （ `0x30` ）- 命名空间名称 `ssl` （ `0x20` ）- 二进制 SSL TLV 结构支持以下 SSL TLV 类型名称： `ssl_version` （ `0x21` ）- 客户端连接中使用的 SSL 版本 `ssl_cn` （ `0x22` ）- SSL 证书公用名 `ssl_cipher` （ `0x23` ）- 使用的密码名称 `ssl_sig_alg` （ `0x24` ）- 用于签署证书的算法 `ssl_key_alg` （ `0x25` ）- 公钥算法此外，还支持以下特殊 SSL TLV 类型名称： `ssl_verify` - 客户端 SSL 证书验证结果， `0` 如果客户端出示了证书并且已成功验证，则为非零，否则为零。必须通过在 listen 指令中设置 `proxy_protocol` 参数来预先启用 PROXY 协议。

### `$query_string`

  same as `$args`

  ​	​与 `$args` 相同

### `$realpath_root`

  an absolute pathname corresponding to the [root](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#root) or [alias](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#alias) directive’s value for the current request, with all symbolic links resolved to real paths

  ​	​对应于当前请求的 root 或 alias 指令值的绝对路径名，其中所有符号链接都解析为真实路径

### `$remote_addr`

  client address

  ​	​客户端地址

### `$remote_port`

  client port

  ​	​客户端端口

### `$remote_user`

  user name supplied with the Basic authentication

  ​	​使用基本身份验证提供的用户名

### `$request`

  full original request line

  ​	​完整的原始请求行

### `$request_body`

  request bodyThe variable’s value is made available in locations processed by the [proxy_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_proxy_module/#proxy_pass), [fastcgi_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_fastcgi_module/#fastcgi_pass), [uwsgi_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_uwsgi_module/#uwsgi_pass), and [scgi_pass](https://before80.github.io/nginx_docs/mod_ref/ngx_http_scgi_module/#scgi_pass) directives when the request body was read to a [memory buffer](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#client_body_buffer_size).

  ​	​请求正文当请求正文被读入内存缓冲区时，变量的值在由 proxy_pass、fastcgi_pass、uwsgi_pass 和 scgi_pass 指令处理的位置中可用。

### `$request_body_file`

  name of a temporary file with the request bodyAt the end of processing, the file needs to be removed. To always write the request body to a file, [client_body_in_file_only](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#client_body_in_file_only) needs to be enabled. When the name of a temporary file is passed in a proxied request or in a request to a FastCGI/uwsgi/SCGI server, passing the request body should be disabled by the [proxy_pass_request_body off](https://before80.github.io/nginx_docs/mod_ref/ngx_http_proxy_module/#proxy_pass_request_body), [fastcgi_pass_request_body off](https://before80.github.io/nginx_docs/mod_ref/ngx_http_fastcgi_module/#fastcgi_pass_request_body), [uwsgi_pass_request_body off](https://before80.github.io/nginx_docs/mod_ref/ngx_http_uwsgi_module/#uwsgi_pass_request_body), or [scgi_pass_request_body off](https://before80.github.io/nginx_docs/mod_ref/ngx_http_scgi_module/#scgi_pass_request_body) directives, respectively.

  ​	​带有请求正文的临时文件名称在处理结束时，需要删除该文件。要始终将请求正文写入文件，只需启用 client_body_in_file_only。当在代理请求或对 FastCGI/uwsgi/SCGI 服务器的请求中传递临时文件名称时，应分别通过 proxy_pass_request_body off、fastcgi_pass_request_body off、uwsgi_pass_request_body off 或 scgi_pass_request_body off 指令禁用传递请求正文。

### `$request_completion`

  “`OK`” if a request has completed, or an empty string otherwise

  ​	​如果请求已完成，则为“ `OK` ”，否则为空字符串

### `$request_filename`

  file path for the current request, based on the [root](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#root) or [alias](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#alias) directives, and the request URI

  ​	​基于根或别名指令和请求 URI 的当前请求的文件路径

### `$request_id`

  unique request identifier generated from 16 random bytes, in hexadecimal (1.11.0)

  ​	​从 16 个随机字节生成的十六进制唯一请求标识符 (1.11.0)

### `$request_length`

  request length (including request line, header, and request body) (1.3.12, 1.2.7)

  ​	​请求长度（包括请求行、标头和请求正文）(1.3.12, 1.2.7)

### `$request_method`

  request method, usually “`GET`” or “`POST`”

  ​	​请求方法，通常为“ `GET` ”或“ `POST` ”

### `$request_time`

  request processing time in seconds with a milliseconds resolution (1.3.9, 1.2.6); time elapsed since the first bytes were read from the client

  ​	​以毫秒为单位的请求处理时间（以秒为单位）（1.3.9、1.2.6）；自从从客户端读取第一个字节以来的经过时间

### `$request_uri`

  full original request URI (with arguments)

  ​	​完整的原始请求 URI（带参数）

### `$scheme`

  request scheme, “`http`” or “`https`”

  ​	​请求方案，“ `http` ”或“ `https` ”

### `$sent_http_` `name`

  arbitrary response header field; the last part of a variable name is the field name converted to lower case with dashes replaced by underscores

  ​	​任意响应标头字段；变量名称的最后部分是将字段名称转换为小写并用下划线替换破折号

### `$sent_trailer_` `name`

  arbitrary field sent at the end of the response (1.13.2); the last part of a variable name is the field name converted to lower case with dashes replaced by underscores

  ​	​在响应结束时发送的任意字段 (1.13.2)；变量名称的最后部分是将字段名称转换为小写并用下划线替换破折号

### `$server_addr`

  an address of the server which accepted a requestComputing a value of this variable usually requires one system call. To avoid a system call, the [listen](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#listen) directives must specify addresses and use the `bind` parameter.

  ​	​接受请求的服务器地址计算此变量的值通常需要一个系统调用。为避免系统调用，监听指令必须指定地址并使用 `bind` 参数。

### `$server_name`

  name of the server which accepted a request

  ​	​接受请求的服务器名称

### `$server_port`

  port of the server which accepted a request

  ​	​接受请求的服务器端口

### `$server_protocol`

  request protocol, usually “`HTTP/1.0`”, “`HTTP/1.1`”, “[HTTP/2.0](https://nginx.org/en/docs/http/ngx_http_v2_module.html)”, or “[HTTP/3.0](https://nginx.org/en/docs/http/ngx_http_v3_module.html)”

  ​	​请求协议，通常为“ `HTTP/1.0` ”、“ `HTTP/1.1` ”、“ HTTP/2.0”或“ HTTP/3.0”

### `$status`

  response status (1.3.2, 1.2.2)

  ​	​响应状态（1.3.2、1.2.2）

### `$tcpinfo_rtt`, `$tcpinfo_rttvar`, `$tcpinfo_snd_cwnd`, `$tcpinfo_rcv_space`

  information about the client TCP connection; available on systems that support the `TCP_INFO` socket option

  ​	客户端 TCP 连接信息；在支持 `TCP_INFO` 套接字选项的系统上可用

### `$time_iso8601`

  local time in the ISO 8601 standard format (1.3.12, 1.2.7)

  ​	​ISO 8601 标准格式的本地时间（1.3.12、1.2.7）

### `$time_local`

  local time in the Common Log Format (1.3.12, 1.2.7)

  ​	​通用日志格式的本地时间（1.3.12、1.2.7）

### `$uri`

  current URI in request, [normalized](https://before80.github.io/nginx_docs/mod_ref/ngx_http_core_module/#location)The value of `$uri` may change during request processing, e.g. when doing internal redirects, or when using index files.

  ​	​请求中的当前 URI，已规范化 `$uri` 的值在请求处理期间可能会更改，例如在执行内部重定向或使用索引文件时。