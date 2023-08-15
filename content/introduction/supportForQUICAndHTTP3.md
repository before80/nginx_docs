+++
title = "支持 QUIC 和 HTTP/3"
date = 2023-08-14T16:54:09+08:00
weight = 130
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Support for QUIC and HTTP/3 - 支持 QUIC 和 HTTP/3

https://nginx.org/en/docs/quic.html



Support for [QUIC](https://datatracker.ietf.org/doc/html/rfc9000) and [HTTP/3](https://datatracker.ietf.org/doc/html/rfc9114) protocols is available since 1.25.0. Also, since 1.25.0, the QUIC and HTTP/3 support is available in Linux [binary packages](https://nginx.org/en/linux_packages.html).

​	自 1.25.0 版本开始，nginx 支持 [QUIC](https://datatracker.ietf.org/doc/html/rfc9000) 和 [HTTP/3](https://datatracker.ietf.org/doc/html/rfc9114) 协议。此外，自 1.25.0 版本以来，QUIC 和 HTTP/3 支持在 Linux [二进制包](https://nginx.org/en/linux_packages.html)中也是可用的。

The QUIC and HTTP/3 support is experimental, caveat emptor applies.
​	QUIC 和 HTTP/3 支持是实验性的，适用于购买者注意



## 从源码构建 - Building from sources

The build is configured using the `configure` command. Please refer to [Building nginx from Sources](https://nginx.org/en/docs/configure.html) for details.

​	构建使用 `configure` 命令进行配置。详细信息请参阅 [从源代码构建 nginx](https://nginx.org/en/docs/configure.html)。

When configuring nginx, it is possible to enable QUIC and HTTP/3 using the [`--with-http_v3_module`](https://nginx.org/en/docs/configure.html#http_v3_module) configuration parameter.

​	在配置 nginx 时，可以使用 [`--with-http_v3_module`](https://nginx.org/en/docs/configure.html#http_v3_module) 配置参数启用 QUIC 和 HTTP/3。

An SSL library that provides QUIC support is recommended to build nginx, such as [BoringSSL](https://boringssl.googlesource.com/boringssl), [LibreSSL](https://www.libressl.org/), or [QuicTLS](https://github.com/quictls/openssl). Otherwise, the [OpenSSL](https://openssl.org/) compatibility layer will be used that does not support [early data](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_early_data).

​	建议构建 nginx 时使用提供 QUIC 支持的 SSL 库，例如 [BoringSSL](https://boringssl.googlesource.com/boringssl)、[LibreSSL](https://www.libressl.org/) 或 [QuicTLS](https://github.com/quictls/openssl)。否则，将使用不支持 [early data](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_early_data) 的 [OpenSSL](https://openssl.org/) 兼容性层。

Use the following command to configure nginx with [BoringSSL](https://boringssl.googlesource.com/boringssl):

​	使用以下命令使用 [BoringSSL](https://boringssl.googlesource.com/boringssl) 配置 nginx：

```
./configure
 --with-debug
 --with-http_v3_module
 --with-cc-opt="-I../boringssl/include"
 --with-ld-opt="-L../boringssl/build/ssl
                -L../boringssl/build/crypto"
```

Alternatively, nginx can be configured with [QuicTLS](https://github.com/quictls/openssl):

​	或者，可以使用 [QuicTLS](https://github.com/quictls/openssl) 配置 nginx：

```
./configure
 --with-debug
 --with-http_v3_module
 --with-cc-opt="-I../quictls/build/include"
 --with-ld-opt="-L../quictls/build/lib"
```

Alternatively, nginx can be configured with a modern version of [LibreSSL](https://www.libressl.org/):

​	或者，也可以使用现代版本的 [LibreSSL](https://www.libressl.org/) 配置 nginx：

```
./configure
 --with-debug
 --with-http_v3_module
 --with-cc-opt="-I../libressl/build/include"
 --with-ld-opt="-L../libressl/build/lib"
```

After configuration, nginx is compiled and installed using `make`.

​	配置完成后，使用 `make` 编译并安装 nginx。



## 配置 - Configuration

The [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive in [ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html) module got a new parameter [quic](https://nginx.org/en/docs/http/ngx_http_core_module.html#quic) which enables HTTP/3 over QUIC on the specified port.

​	在 [ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html) 模块中的 [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令添加了一个新的参数 [quic](https://nginx.org/en/docs/http/ngx_http_core_module.html#quic)，该参数在指定的端口上启用基于 QUIC 的 HTTP/3。

Along with the `quic` parameter it is also possible to specify the [reuseport](https://nginx.org/en/docs/http/ngx_http_core_module.html#reuseport) parameter to make it work properly with multiple workers.

​	除了 `quic` 参数外，还可以使用 [reuseport](https://nginx.org/en/docs/http/ngx_http_core_module.html#reuseport) 参数，以便与多个 worker 正常工作。

For the list of directives, see [ngx_http_v3_module](https://nginx.org/en/docs/http/ngx_http_v3_module.html).

​	有关指令的列表，请参阅 [ngx_http_v3_module](https://nginx.org/en/docs/http/ngx_http_v3_module.html)。

To [enable](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_retry) address validation:

​	要 [启用](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_retry) 地址验证：

```
quic_retry on;
```

To [enable](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_early_data) 0-RTT:

​	要 [启用](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_early_data) 0-RTT：

```
ssl_early_data on;
```

To [enable](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_gso) GSO (Generic Segmentation Offloading):

​	要 [启用](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_gso) GSO（通用分段卸载）：

```
quic_gso on;
```

To [set](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_host_key) host key for various tokens:

​	要 [设置](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_host_key) 主机密钥以用于各种令牌：

```
quic_host_key <filename>;
```



QUIC requires TLSv1.3 protocol version which is enabled by default in the [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) directive.

​	QUIC 需要 TLSv1.3 协议版本，默认在 [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) 指令中启用。

By default, [GSO Linux-specific optimization](http://vger.kernel.org/lpc_net2018_talks/willemdebruijn-lpc2018-udpgso-paper-DRAFT-1.pdf) is disabled. Enable it in case a corresponding network interface is configured to support GSO.

​	默认情况下，已禁用 [GSO Linux 特定优化](http://vger.kernel.org/lpc_net2018_talks/willemdebruijn-lpc2018-udpgso-paper-DRAFT-1.pdf)。在配置了支持 GSO 的相应网络接口的情况下启用它。



## 示例配置 Example Configuration

```
http {
 log_format quic '$remote_addr - $remote_user [$time_local] '
                 '"$request" $status $body_bytes_sent '
                 '"$http_referer" "$http_user_agent" "$http3"';

 access_log logs/access.log quic;

 server {
     # for better compatibility it's recommended
     # to use the same port for quic and https
     # 为了更好的兼容性，建议
     # 在 quic 和 https 上使用相同的端口
     listen 8443 quic reuseport;
     listen 8443 ssl;

     ssl_certificate     certs/example.com.crt;
     ssl_certificate_key certs/example.com.key;

     location / {
         # required for browsers to direct them to quic port
         # 为了让浏览器将请求指向 quic 端口，这是必需的
         add_header Alt-Svc 'h3=":8443"; ma=86400';
     }
 }
}
```





## 故障排除 Troubleshooting

Tips that may help to identify problems:

​	以下是有助于识别问题的提示：

- Ensure nginx is built with the proper SSL library.

- 确保 nginx 使用正确的 SSL 库进行构建。

- Ensure nginx is using the proper SSL library in runtime (the `nginx -V` shows what it is currently used).

- 确保运行时 nginx 正在使用正确的 SSL 库（`nginx -V` 显示当前使用的库）。

- Ensure a client is actually sending requests over QUIC. It is recommended to start with a simple console client such as [ngtcp2](https://nghttp2.org/ngtcp2) to ensure the server is configured properly before trying with real browsers that may be quite picky with certificates.

- 确保客户端实际上正在通过 QUIC 发送请求。建议首先使用简单的控制台客户端（如 [ngtcp2](https://nghttp2.org/ngtcp2)）进行配置，以确保服务器配置正确，然后再尝试使用可能对证书非常挑剔的实际浏览器。

- Build nginx with [debug support](https://nginx.org/en/docs/debugging_log.html) and check the debug log. It should contain all details about the connection and why it failed. All related messages contain the “`quic`” prefix and can be easily filtered out.

- 使用 [debug 支持](https://nginx.org/en/docs/debugging_log.html) 构建 nginx 并检查调试日志。它应该包含有关连接的所有详细信息以及失败的原因。所有相关的消息都包含“`quic`”前缀，可以轻松地进行过滤。

- For a deeper investigation, additional debugging can be enabled using the following macros: `NGX_QUIC_DEBUG_PACKETS`, `NGX_QUIC_DEBUG_FRAMES`, `NGX_QUIC_DEBUG_ALLOC`, `NGX_QUIC_DEBUG_CRYPTO`.

- 对于更深入的调查，可以使用以下宏进行附加调试：`NGX_QUIC_DEBUG_PACKETS`、`NGX_QUIC_DEBUG_FRAMES`、`NGX_QUIC_DEBUG_ALLOC`、`NGX_QUIC_DEBUG_CRYPTO`。

  ```
  ./configure
   --with-http_v3_module
   --with-debug
   --with-cc-opt="-DNGX_QUIC_DEBUG_PACKETS -DNGX_QUIC_DEBUG_CRYPTO"
  ```
  
  