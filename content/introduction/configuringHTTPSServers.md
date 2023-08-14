+++
title = "配置 HTTPS 服务器"
date = 2023-08-14T16:56:09+08:00
weight = 170
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Configuring HTTPS servers - 配置 HTTPS 服务器

https://nginx.org/en/docs/http/configuring_https_servers.html



To configure an HTTPS server, the `ssl` parameter must be enabled on [listening sockets](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) in the [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block, and the locations of the [server certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) and [private key](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key) files should be specified:

​	为了配置一个 HTTPS 服务器，必须在 [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块中的 [listening sockets](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 上启用 `ssl` 参数，并且应指定 [server certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) 和 [private key](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key) 文件的位置：

```
server {
 listen              443 ssl;
 server_name         www.example.com;
 ssl_certificate     www.example.com.crt;
 ssl_certificate_key www.example.com.key;
 ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
 ssl_ciphers         HIGH:!aNULL:!MD5;
 ...
}
```

The server certificate is a public entity. It is sent to every client that connects to the server. The private key is a secure entity and should be stored in a file with restricted access, however, it must be readable by nginx’s master process. The private key may alternately be stored in the same file as the certificate:

​	服务器证书是一个公共实体，会发送给连接到服务器的每个客户端。私钥是一个安全实体，应存储在受限制的访问权限文件中，但是它必须对 nginx 的主进程可读。私钥也可以与证书存储在同一个文件中：

```
 ssl_certificate     www.example.com.cert;
 ssl_certificate_key www.example.com.cert;
```

in which case the file access rights should also be restricted. Although the certificate and the key are stored in one file, only the certificate is sent to a client.

​	在这种情况下，文件访问权限也应受到限制。尽管证书和私钥存储在一个文件中，但只有证书会被发送给客户端。

The directives [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) and [ssl_ciphers](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers) can be used to limit connections to include only the strong versions and ciphers of SSL/TLS. By default nginx uses “`ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3`” and “`ssl_ciphers HIGH:!aNULL:!MD5`”, so configuring them explicitly is generally not needed. Note that default values of these directives were [changed](https://nginx.org/en/docs/http/configuring_https_servers.html#compatibility) several times.

​	指令 [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) 和 [ssl_ciphers](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers) 可以用于限制连接，只包括强版本和密码。默认情况下，nginx 使用“`ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3`”和“`ssl_ciphers HIGH:!aNULL:!MD5`”，因此通常不需要显式配置它们。请注意，这些指令的默认值已经[多次更改过](https://nginx.org/en/docs/http/configuring_https_servers.html#compatibility)。



## HTTPS 服务器优化 - HTTPS server optimization

SSL operations consume extra CPU resources. On multi-processor systems several [worker processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes) should be run, no less than the number of available CPU cores. The most CPU-intensive operation is the SSL handshake. There are two ways to minimize the number of these operations per client: the first is by enabling [keepalive](https://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_timeout) connections to send several requests via one connection and the second is to reuse SSL session parameters to avoid SSL handshakes for parallel and subsequent connections. The sessions are stored in an SSL session cache shared between workers and configured by the [ssl_session_cache](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache) directive. One megabyte of the cache contains about 4000 sessions. The default cache timeout is 5 minutes. It can be increased by using the [ssl_session_timeout](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_timeout) directive. Here is a sample configuration optimized for a multi-core system with 10 megabyte shared session cache:

​	SSL 操作会消耗额外的 CPU 资源。在多处理器系统上，应运行多个 [worker processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)，至少不少于可用的 CPU 核心数。最消耗 CPU 的操作是 SSL 握手。有两种方式可以最小化每个客户端的这些操作次数：一是通过启用 [keepalive](https://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_timeout) 连接来通过一个连接发送多个请求，二是重用 SSL 会话参数以避免并行和后续连接的 SSL 握手。这些会话存储在在 worker 之间共享的 SSL 会话缓存中，由 [ssl_session_cache](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache) 指令进行配置。一个缓存中大约包含 4000 个会话的数据。默认缓存超时为 5 分钟。可以通过使用 [ssl_session_timeout](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_timeout) 指令进行增加。以下是针对具有 10 兆字节共享会话缓存的多核系统优化的示例配置：

```
worker_processes auto;

http {
 ssl_session_cache   shared:SSL:10m;
 ssl_session_timeout 10m;

 server {
     listen              443 ssl;
     server_name         www.example.com;
     keepalive_timeout   70;

     ssl_certificate     www.example.com.crt;
     ssl_certificate_key www.example.com.key;
     ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
     ssl_ciphers         HIGH:!aNULL:!MD5;
     ...
```





## SSL 证书链 - SSL certificate chains

Some browsers may complain about a certificate signed by a well-known certificate authority, while other browsers may accept the certificate without issues. This occurs because the issuing authority has signed the server certificate using an intermediate certificate that is not present in the certificate base of well-known trusted certificate authorities which is distributed with a particular browser. In this case the authority provides a bundle of chained certificates which should be concatenated to the signed server certificate. The server certificate must appear before the chained certificates in the combined file:

​	一些浏览器可能会对由知名证书颁发机构签名的证书发出警告，而其他浏览器可能会接受该证书而没有问题。这是因为颁发机构已使用中间证书签署了服务器证书，而该中间证书在与特定浏览器分发的众所周知的受信任证书颁发机构的证书库中不存在。在这种情况下，颁发机构提供了一组链接的证书，应将其与已签名的服务器证书连接在一起。服务器证书必须出现在连接文件中链接的证书之前：

```
$ cat www.example.com.crt bundle.crt > www.example.com.chained.crt
```

The resulting file should be used in the [ssl_certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) directive:

​	结果文件应在 [ssl_certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) 指令中使用：

```
server {
 listen              443 ssl;
 server_name         www.example.com;
 ssl_certificate     www.example.com.chained.crt;
 ssl_certificate_key www.example.com.key;
 ...
}
```

If the server certificate and the bundle have been concatenated in the wrong order, nginx will fail to start and will display the error message:

​	如果服务器证书和链接的证书的连接顺序错误，nginx 将无法启动，并显示错误消息：

```
SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") failed
(SSL: error:0B080074:x509 certificate routines:
 X509_check_private_key:key values mismatch)
```

because nginx has tried to use the private key with the bundle’s first certificate instead of the server certificate.

因为 nginx 已尝试使用链接的 bundle 的第一个证书而不是服务器证书的私钥。

Browsers usually store intermediate certificates which they receive and which are signed by trusted authorities, so actively used browsers may already have the required intermediate certificates and may not complain about a certificate sent without a chained bundle. To ensure the server sends the complete certificate chain, the `openssl` command-line utility may be used, for example:

​	浏览器通常会存储中间证书，它们收到并由受信任的机构签名，因此经常使用的浏览器可能已经具有所需的中间证书，可能不会对发送没有链接的证书发出警告。为确保服务器发送完整的证书链，可以使用命令行实用程序 `openssl`，例如：

```
$ openssl s_client -connect www.godaddy.com:443
...
Certificate chain
0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
  /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
  /OU=MIS Department/CN=www.GoDaddy.com
  /serialNumber=0796928-7/2.5.4.15=V1.0, Clause 5.(b)
i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
  /OU=http://certificates.godaddy.com/repository
  /CN=Go Daddy Secure Certification Authority
  /serialNumber=07969287
1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
  /OU=http://certificates.godaddy.com/repository
  /CN=Go Daddy Secure Certification Authority
  /serialNumber=07969287
i:/C=US/O=The Go Daddy Group, Inc.
  /OU=Go Daddy Class 2 Certification Authority
2 s:/C=US/O=The Go Daddy Group, Inc.
  /OU=Go Daddy Class 2 Certification Authority
i:/L=ValiCert Validation Network/O=ValiCert, Inc.
  /OU=ValiCert Class 2 Policy Validation Authority
  /CN=http://www.valicert.com//emailAddress=info@valicert.com
...
```



> When testing configurations with [SNI](https://nginx.org/en/docs/http/configuring_https_servers.html#sni), it is important to specify the `-servername` option as `openssl` does not use SNI by default.
>
> ​	在测试配置时，使用 [SNI](https://nginx.org/en/docs/http/configuring_https_servers.html#sni) 时，重要的是要将 `-servername` 选项指定为 `openssl` 默认不使用 SNI。

In this example the subject (“*s*”) of the `www.GoDaddy.com` server certificate #0 is signed by an issuer (“*i*”) which itself is the subject of the certificate #1, which is signed by an issuer which itself is the subject of the certificate #2, which signed by the well-known issuer *ValiCert, Inc.* whose certificate is stored in the browsers’ built-in certificate base (that lay in the house that Jack built).

​	在此示例中，`www.GoDaddy.com` 服务器证书 #0 的主题 ("*s*") 是由签发者 ("*i*") 签署的，签发者本身是证书 #1 的主题，证书 #2 的签发者本身是证书 #1 的主题，这是证书 #2 的签发者签署的，这是知名签发者 *ValiCert, Inc.* 的证书，其证书存储在浏览器内置的证书库中。

If a certificate bundle has not been added, only the server certificate #0 will be shown.

​	如果证书 bundle 没有被添加，只会显示服务器证书 #0。



## 单一的 HTTP/HTTPS 服务器 - A single HTTP/HTTPS server

It is possible to configure a single server that handles both HTTP and HTTPS requests:

​	可以配置一个处理 HTTP 和 HTTPS 请求的单一服务器：

```
server {
 listen              80;
 listen              443 ssl;
 server_name         www.example.com;
 ssl_certificate     www.example.com.crt;
 ssl_certificate_key www.example.com.key;
 ...
}
```



> Prior to 0.7.14 SSL could not be enabled selectively for individual listening sockets, as shown above. SSL could only be enabled for the entire server using the [ssl](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl) directive, making it impossible to set up a single HTTP/HTTPS server. The `ssl` parameter of the [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive was added to solve this issue. The use of the [ssl](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl) directive in modern versions is thus discouraged.
>
> ​	在版本 0.7.14 之前，不能选择性地为单个监听 socket 启用 SSL，如上所示。只能使用 [ssl](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl) 指令为整个服务器启用 SSL，从而无法设置单一的 HTTP/HTTPS 服务器。在现代版本中使用 [ssl](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl) 指令因此被不建议。





## 基于名称的 HTTPS 服务器 - Name-based HTTPS servers

A common issue arises when configuring two or more HTTPS servers listening on a single IP address:

​	在配置在单个 IP 地址上监听的两个或多个 HTTPS 服务器时，通常会出现一个常见的问题：

```
server {
 listen          443 ssl;
 server_name     www.example.com;
 ssl_certificate www.example.com.crt;
 ...
}

server {
 listen          443 ssl;
 server_name     www.example.org;
 ssl_certificate www.example.org.crt;
 ...
}
```

With this configuration a browser receives the default server’s certificate, i.e. `www.example.com` regardless of the requested server name. This is caused by SSL protocol behaviour. The SSL connection is established before the browser sends an HTTP request and nginx does not know the name of the requested server. Therefore, it may only offer the default server’s certificate.

​	使用此配置，无论请求的服务器名称如何，浏览器都会接收到默认服务器的证书，即使请求的服务器名称与之不同。这是由于 SSL 协议的行为造成的。SSL 连接在浏览器发送 HTTP 请求之前建立，nginx 不知道请求的服务器名称。因此，它只能提供默认服务器的证书。

The oldest and most robust method to resolve the issue is to assign a separate IP address for every HTTPS server:

​	解决这个问题的最古老和最健壮的方法是为每个 HTTPS 服务器分配一个单独的 IP 地址：

```
server {
 listen          192.168.1.1:443 ssl;
 server_name     www.example.com;
 ssl_certificate www.example.com.crt;
 ...
}

server {
 listen          192.168.1.2:443 ssl;
 server_name     www.example.org;
 ssl_certificate www.example.org.crt;
 ...
}
```





## 具有多个名称的 SSL 证书 - An SSL certificate with several names

There are other ways that allow sharing a single IP address between several HTTPS servers. However, all of them have their drawbacks. One way is to use a certificate with several names in the SubjectAltName certificate field, for example, `www.example.com` and `www.example.org`. However, the SubjectAltName field length is limited.

​	还有其他方法可以在多个 HTTPS 服务器之间共享单个 IP 地址。然而，它们都有各自的缺点。一种方法是使用具有多个名称的证书放在 SubjectAltName 证书字段中，例如 `www.example.com` 和 `www.example.org`。但是，SubjectAltName 字段的长度是有限的。

Another way is to use a certificate with a wildcard name, for example, `*.example.org`. A wildcard certificate secures all subdomains of the specified domain, but only on one level. This certificate matches `www.example.org`, but does not match `example.org` and `www.sub.example.org`. These two methods can also be combined. A certificate may contain exact and wildcard names in the SubjectAltName field, for example, `example.org` and `*.example.org`.

​	另一种方法是使用具有通配符名称的证书，例如 `*.example.org`。通配符证书可保护指定域的所有子域，但仅在一个级别上有效。这个证书匹配 `www.example.org`，但不匹配 `example.org` 和 `www.sub.example.org`。这两种方法也可以结合使用。证书可以在 SubjectAltName 字段中包含确切的和通配符名称，例如 `example.org` 和 `*.example.org`。

It is better to place a certificate file with several names and its private key file at the *http* level of configuration to inherit their single memory copy in all servers:

​	最好将具有多个名称的证书文件和其私钥文件放置在配置的 *http* 级别中，以在所有服务器中继承它们的单一内存副本：

```
ssl_certificate     common.crt;
ssl_certificate_key common.key;

server {
 listen          443 ssl;
 server_name     www.example.com;
 ...
}

server {
 listen          443 ssl;
 server_name     www.example.org;
 ...
}
```





## 服务器名称指示 - Server Name Indication

A more generic solution for running several HTTPS servers on a single IP address is [TLS Server Name Indication extension](http://en.wikipedia.org/wiki/Server_Name_Indication) (SNI, RFC 6066), which allows a browser to pass a requested server name during the SSL handshake and, therefore, the server will know which certificate it should use for the connection. SNI is currently [supported](http://en.wikipedia.org/wiki/Server_Name_Indication#Support) by most modern browsers, though may not be used by some old or special clients.

​	在单个 IP 地址上运行多个 HTTPS 服务器的更通用的解决方案是 [TLS Server Name Indication 扩展](http://en.wikipedia.org/wiki/Server_Name_Indication) (SNI, RFC 6066)，它允许浏览器在 SSL 握手期间传递请求的服务器名称，因此服务器将知道它应该使用哪个证书进行连接。SNI 目前由大多数现代浏览器[支持](http://en.wikipedia.org/wiki/Server_Name_Indication#Support)，尽管某些旧版本或特殊客户端可能不使用。

> Only domain names can be passed in SNI, however some browsers may erroneously pass an IP address of the server as its name if a request includes literal IP address. One should not rely on this.
>
> ​	只能传递域名到 SNI 中，但是如果请求包括字面 IP 地址，一些浏览器可能会错误地将服务器的 IP 地址作为其名称传递。不能依赖这一点。



In order to use SNI in nginx, it must be supported in both the OpenSSL library with which the nginx binary has been built as well as the library to which it is being dynamically linked at run time. OpenSSL supports SNI since 0.9.8f version if it was built with config option “--enable-tlsext”. Since OpenSSL 0.9.8j this option is enabled by default. If nginx was built with SNI support, then nginx will show this when run with the “-V” switch:

​	为了在 nginx 中使用 SNI，必须同时在构建 nginx 二进制文件的 OpenSSL 库中支持它，并且在运行时动态链接到的库中也支持它。从 0.9.8f 版本开始，OpenSSL 支持 SNI，如果使用了配置选项 “--enable-tlsext” 进行构建。自从 OpenSSL 0.9.8j 版本开始，默认启用此选项。如果 nginx 使用了 SNI 支持，则在使用 “-V” 选项运行 nginx 时会显示出来：

```
$ nginx -V
...
TLS SNI support enabled
...
```

However, if the SNI-enabled nginx is linked dynamically to an OpenSSL library without SNI support, nginx displays the warning:

​	然而，如果启用了 SNI 支持的 nginx 在运行时动态链接到不支持 SNI 的 OpenSSL 库，nginx 将显示警告：

```
nginx was built with SNI support, however, now it is linked
dynamically to an OpenSSL library which has no tlsext support,
therefore SNI is not available
```





## 兼容性 Compatibility



- The SNI support status has been shown by the “-V” switch since 0.8.21 and 0.7.62.
- The `ssl` parameter of the [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive has been supported since 0.7.14. Prior to 0.8.21 it could only be specified along with the `default` parameter.
- SNI has been supported since 0.5.23.
- The shared SSL session cache has been supported since 0.5.6.

- Version 1.23.4 and later: the default SSL protocols are TLSv1, TLSv1.1, TLSv1.2, and TLSv1.3 (if supported by the OpenSSL library).
- Version 1.9.1 and later: the default SSL protocols are TLSv1, TLSv1.1, and TLSv1.2 (if supported by the OpenSSL library).
- Version 0.7.65, 0.8.19 and later: the default SSL protocols are SSLv3, TLSv1, TLSv1.1, and TLSv1.2 (if supported by the OpenSSL library).
- Version 0.7.64, 0.8.18 and earlier: the default SSL protocols are SSLv2, SSLv3, and TLSv1.

- Version 1.0.5 and later: the default SSL ciphers are “`HIGH:!aNULL:!MD5`”.
- Version 0.7.65, 0.8.20 and later: the default SSL ciphers are “`HIGH:!ADH:!MD5`”.
- Version 0.8.19: the default SSL ciphers are “`ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM`”.
- Version 0.7.64, 0.8.18 and earlier: the default SSL ciphers are
  “`ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP`”.
- SNI 支持状态自 0.8.21 和 0.7.62 版本开始通过 “-V” 开关显示。
- [监听](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令的 `ssl` 参数自 0.7.14 版本开始支持。在 0.8.21 版本之前，它只能与 `default` 参数一起使用。
- SNI 自 0.5.23 版本开始支持。
- 共享的 SSL 会话缓存自 0.5.6 版本开始支持。
- 版本 1.23.4 及以后：默认的 SSL 协议为 TLSv1、TLSv1.1、TLSv1.2 和 TLSv1.3（如果 OpenSSL 库支持）。
- 版本 1.9.1 及以后：默认的 SSL 协议为 TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 库支持）。
- 版本 0.7.65、0.8.19 及以后：默认的 SSL 协议为 SSLv3、TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 库支持）。
- 版本 0.7.64、0.8.18 及之前：默认的 SSL 协议为 SSLv2、SSLv3 和 TLSv1。
- 版本 1.0.5 及以后：默认的 SSL 密码为 “`HIGH:!aNULL:!MD5`”。
- 版本 0.7.65、0.8.20 及以后：默认的 SSL 密码为 “`HIGH:!ADH:!MD5`”。
- 版本 0.8.19：默认的 SSL 密码为 “`ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM`”。
- 版本 0.7.64、0.8.18 及之前：默认的 SSL 密码为 “`ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP`”。



written by Igor Sysoev edited by Brian Mercer 