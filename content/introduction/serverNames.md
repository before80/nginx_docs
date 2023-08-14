+++
title = "Server names"
date = 2023-08-14T16:55:07+08:00
weight = 150
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Server names

https://nginx.org/en/docs/http/server_names.html



Server names are defined using the [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) directive and determine which [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block is used for a given request. See also “[How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html)”. They may be defined using exact names, wildcard names, or regular expressions:

​	服务器名称使用 [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令进行定义，用于确定哪个 [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块用于给定的请求。也可以查看 “[How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html)” 进行更多了解。服务器名称可以使用精确名称、通配符名称或正则表达式进行定义：

```
server {
 listen       80;
 server_name  example.org  www.example.org;
 ...
}

server {
 listen       80;
 server_name  *.example.org;
 ...
}

server {
 listen       80;
 server_name  mail.*;
 ...
}

server {
 listen       80;
 server_name  ~^(?<user>.+)\.example\.net$;
 ...
}
```



When searching for a virtual server by name, if name matches more than one of the specified variants, e.g. both wildcard name and regular expression match, the first matching variant will be chosen, in the following order of precedence:

​	当按名称搜索虚拟服务器时，如果名称匹配多个指定的变体，例如，通配符名称和正则表达式都匹配，将选择第一个匹配的变体，顺序如下：

1. exact name
2. longest wildcard name starting with an asterisk, e.g. “`*.example.org`”
3. longest wildcard name ending with an asterisk, e.g. “`mail.*`”
4. first matching regular expression (in order of appearance in a configuration file)
5. 精确名称
6. 以星号开头的最长通配符名称，例如“`*.example.org`”
7. 以星号结尾的最长通配符名称，例如“`mail.*`”
8. 第一个匹配的正则表达式（按出现在配置文件中的顺序）





## 通配符名称 - Wildcard names

A wildcard name may contain an asterisk only on the name’s start or end, and only on a dot border. The names “`www.*.example.org`” and “`w*.example.org`” are invalid. However, these names can be specified using regular expressions, for example, “`~^www\..+\.example\.org$`” and “`~^w.*\.example\.org$`”. An asterisk can match several name parts. The name “`*.example.org`” matches not only `www.example.org` but `www.sub.example.org` as well.

​	通配符名称只能在名称的开头或末尾以及点边界上包含一个星号。名称 “`www.*.example.org`” 和 “`w*.example.org`” 是无效的。但是，可以使用正则表达式指定这些名称，例如“`~^www\..+\.example\.org$`” 和 “`~^w.*\.example\.org$`”。星号可以匹配多个名称部分。名称 “`*.example.org`” 不仅匹配 `www.example.org`，还匹配 `www.sub.example.org`。

A special wildcard name in the form “`.example.org`” can be used to match both the exact name “`example.org`” and the wildcard name “`*.example.org`”.

​	特殊的通配符名称形式 “`.example.org`” 可用于同时匹配精确名称 “`example.org`” 和通配符名称 “`*.example.org`”。



## 正则表达式名称 - Regular expressions names

The regular expressions used by nginx are compatible with those used by the Perl programming language (PCRE). To use a regular expression, the server name must start with the tilde character:

​	nginx 使用的正则表达式与 Perl 编程语言（PCRE）兼容。要使用正则表达式，服务器名称必须以波浪号字符开头：

```
server_name  ~^www\d+\.example\.net$;
```

otherwise it will be treated as an exact name, or if the expression contains an asterisk, as a wildcard name (and most likely as an invalid one). Do not forget to set “`^`” and “`$`” anchors. They are not required syntactically, but logically. Also note that domain name dots should be escaped with a backslash. A regular expression containing the characters “`{`” and “`}`” should be quoted:

​	否则，它将被视为精确名称，或者如果表达式包含星号，则视为通配符名称（很可能是无效的）。不要忘记设置 “`^`” 和 “`$`” 锚点。从语法上讲，它们不是必需的，但在逻辑上是必需的。还要注意，域名中的点应该使用反斜杠进行转义。包含字符 “`{`” 和 “`}`” 的正则表达式应该用引号括起来：

```
server_name  "~^(?<name>\w\d{1,3}+)\.example\.net$";
```

otherwise nginx will fail to start and display the error message:

​	否则，nginx 将无法启动并显示错误消息：

```
directive "server_name" is not terminated by ";" in ...
```

A named regular expression capture can be used later as a variable:

​	以命名的正则表达式捕获可以稍后用作变量：

```
server {
 server_name   ~^(www\.)?(?<domain>.+)$;

 location / {
     root   /sites/$domain;
 }
}
```

The PCRE library supports named captures using the following syntax:

​	PCRE 库支持使用以下语法进行命名捕获：

> | `?<*name*>`  | Perl 5.10 compatible syntax, supported since PCRE-7.0 与 Perl 5.10 兼容的语法，自 PCRE-7.0 起支持 |
> | ------------ | ------------------------------------------------------------ |
> | `?'*name*'`  | Perl 5.10 compatible syntax, supported since PCRE-7.0 与 Perl 5.10 兼容的语法，自 PCRE-7.0 起支持 |
> | `?P<*name*>` | Python compatible syntax, supported since PCRE-4.0 与 Python 兼容的语法，自 PCRE-4.0 起支持 |

If nginx fails to start and displays the error message:

​	如果 nginx 在启动时无法启动并显示错误消息：

```
pcre_compile() failed: unrecognized character after (?< in ...
```

this means that the PCRE library is old and the syntax “`?P<*name*>`” should be tried instead. The captures can also be used in digital form:

这意味着 PCRE 库太旧，应尝试使用语法 “`?P<*name*>`” 替代。捕获还可以以数字形式使用：

```
server {
 server_name   ~^(www\.)?(.+)$;

 location / {
     root   /sites/$2;
 }
}
```

However, such usage should be limited to simple cases (like the above), since the digital references can easily be overwritten.

​	然而，应该将这种用法限制在简单的情况下（如上所示），因为数字引用很容易被覆盖。

## 其他名称 - Miscellaneous names

There are some server names that are treated specially.

​	有一些特殊处理的服务器名称。

If it is required to process requests without the “Host” header field in a [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block which is not the default, an empty name should be specified:

​	如果需要在不是默认的 [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块中处理不带 “Host” 标头字段的请求，应指定一个空名称：

```
server {
 listen       80;
 server_name  example.org  www.example.org  "";
 ...
}
```

If no [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) is defined in a [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block then nginx uses the empty name as the server name.

​	如果在 [server](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块中没有定义 [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name)，则 nginx 将使用空名称作为服务器名称。

> nginx versions up to 0.8.48 used the machine’s hostname as the server name in this case.
>
> ​	在 0.8.48 版本之前的 nginx 版本中，此情况下使用的是主机名作为服务器名称。

If a server name is defined as “`$hostname`” (0.9.4), the machine’s hostname is used.

​	如果服务器名称定义为 “`$hostname`”（0.9.4），则使用主机名。

If someone makes a request using an IP address instead of a server name, the “Host” request header field will contain the IP address and the request can be handled using the IP address as the server name:

​	如果有人使用 IP 地址而不是服务器名称发出请求，则“Host” 请求标头字段将包含 IP 地址，请求可以使用 IP 地址作为服务器名称进行处理：

```
server {
 listen       80;
 server_name  example.org
              www.example.org
              ""
              192.168.1.1
              ;
 ...
}
```

In catch-all server examples the strange name “`_`” can be seen:

​	在通用服务器示例中，可以看到奇怪的名称 “`_`”：

```
server {
 listen       80  default_server;
 server_name  _;
 return       444;
}
```

There is nothing special about this name, it is just one of a myriad of invalid domain names which never intersect with any real name. Other invalid names like “`--`” and “`!@#`” may equally be used.

​	这个名称没有什么特别之处，它只是众多无效域名中的一个，与任何真实名称都不相交。其他无效名称，如 “`--`” 和 “`!@#`” 也可以被使用。

nginx versions up to 0.6.25 supported the special name “`*`” which was erroneously interpreted to be a catch-all name. It never functioned as a catch-all or wildcard server name. Instead, it supplied the functionality that is now provided by the [server_name_in_redirect](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) directive. The special name “`*`” is now deprecated and the [server_name_in_redirect](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) directive should be used. Note that there is no way to specify the catch-all name or the default server using the [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) directive. This is a property of the [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive and not of the [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) directive. See also “[How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html)”. It is possible to define servers listening on ports *:80 and *:8080, and direct that one will be the default server for port `*:8080`, while the other will be the default for port `*:80`:

​	0.6.25 版本之前的 nginx 版本支持特殊名称 “`*`”，这个名称被错误地解释为通配符名称。它从未作为通配符服务器名称或通配符名称。相反，它提供了现在由 [server_name_in_redirect](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) 指令提供的功能。特殊名称 “`*`” 现已被弃用，应该使用 [server_name_in_redirect](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) 指令。请注意，没有办法使用 [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令指定通配符名称或默认服务器。这是 [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令的属性，而不是 [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令的属性。也可以查看 “[How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html)” 进行更多了解。可以在 *:80 和 *:8080 端口上定义服务器，指定其中一个将是端口 `*:8080` 的默认服务器，而另一个将是端口 `*:80` 的默认服务器：

```
server {
 listen       80;
 listen       8080  default_server;
 server_name  example.net;
 ...
}

server {
 listen       80  default_server;
 listen       8080;
 server_name  example.org;
 ...
}
```





## 国际化名称 - Internationalized names

Internationalized domain names ([IDNs](https://en.wikipedia.org/wiki/Internationalized_domain_name)) should be specified using an ASCII (Punycode) representation in the [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) directive:

​	国际化域名（[IDNs](https://en.wikipedia.org/wiki/Internationalized_domain_name)）应使用 ASCII（Punycode）表示法在 [server_name](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令中指定：

```
server {
 listen       80;
 server_name  xn--e1afmkfd.xn--80akhbyknj4f;  # пример.испытание
 ...
}
```



## 虚拟服务器选择 - Virtual server selection

First, a connection is created in a default server context. Then, the server name can be determined in the following request processing stages, each involved in server configuration selection:

​	首先，在默认的服务器上下文中创建连接。然后，可以在以下请求处理阶段中确定服务器名称，每个阶段都涉及服务器配置的选择：

- during SSL handshake, in advance, according to [SNI](https://nginx.org/en/docs/http/configuring_https_servers.html#sni)
- after processing the request line
- after processing the `Host` header field
- if the server name was not determined after processing the request line or from the `Host` header field, nginx will use the empty name as the server name.
- 在 SSL 握手期间，根据 [SNI](https://nginx.org/en/docs/http/configuring_https_servers.html#sni) 提前确定
- 在处理请求行之后
- 在处理 `Host` 标头字段之后
- 如果在处理请求行或 `Host` 标头字段后没有确定服务器名称，nginx 将使用空名称作为服务器名称。

At each of these stages, different server configurations can be applied. As such, certain directives should be specified with caution:

​	在这些阶段中，可以应用不同的服务器配置。因此，某些指令应谨慎指定：

- in case of the [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) directive, the protocol list is set by the OpenSSL library before the server configuration could be applied according to the name requested through SNI, thus, protocols should be specified only for a default server;
- 对于 [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) 指令，在 OpenSSL 库设置服务器配置之前，协议列表由 OpenSSL 库设置，根据通过 SNI 请求的名称进行设置，因此，应仅为默认服务器指定协议；
- the [client_header_buffer_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_header_buffer_size) and [merge_slashes](https://nginx.org/en/docs/http/ngx_http_core_module.html#merge_slashes) directives are involved before reading the request line, thus, such directives use a default server configuration or the server configuration chosen by SNI;
- [client_header_buffer_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_header_buffer_size) 和 [merge_slashes](https://nginx.org/en/docs/http/ngx_http_core_module.html#merge_slashes) 指令在读取请求行之前进行处理，因此，这些指令使用默认服务器配置或由 SNI 选择的服务器配置；
- in case of the [ignore_invalid_headers](https://nginx.org/en/docs/http/ngx_http_core_module.html#ignore_invalid_headers), [large_client_header_buffers](https://nginx.org/en/docs/http/ngx_http_core_module.html#large_client_header_buffers), and [underscores_in_headers](https://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers) directives involved in processing request header fields, it additionally depends whether the server configuration was updated according to the request line or the `Host` header field;
- 在处理请求标头字段时，涉及 [ignore_invalid_headers](https://nginx.org/en/docs/http/ngx_http_core_module.html#ignore_invalid_headers)、[large_client_header_buffers](https://nginx.org/en/docs/http/ngx_http_core_module.html#large_client_header_buffers) 和 [underscores_in_headers](https://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers) 指令，还取决于服务器配置是否根据请求行或 `Host` 标头字段进行了更新；
- an error response will be handled with the [error_page](https://nginx.org/en/docs/http/ngx_http_core_module.html#error_page) directive in the server that currently fulfills the request.
- 错误响应将使用 [error_page](https://nginx.org/en/docs/http/ngx_http_core_module.html#error_page) 指令在当前满足请求的服务器中处理。



## 优化 Optimization

Exact names, wildcard names starting with an asterisk, and wildcard names ending with an asterisk are stored in three hash tables bound to the listen ports. The sizes of hash tables are optimized at the configuration phase so that a name can be found with the fewest CPU cache misses. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).

​	精确名称、以星号开头的通配符名称和以星号结尾的通配符名称存储在绑定到监听端口的三个哈希表中。哈希表的大小在配置阶段进行了优化，以便可以以最少的 CPU 缓存未命中找到名称。设置哈希表的详细信息在单独的 [document](https://nginx.org/en/docs/hash.html) 中提供。

The exact names hash table is searched first. If a name is not found, the hash table with wildcard names starting with an asterisk is searched. If the name is not found there, the hash table with wildcard names ending with an asterisk is searched.

​	首先搜索精确名称哈希表。如果找不到名称，则搜索以星号开头的通配符名称的哈希表。如果在那里找不到名称，则搜索以星号结尾的通配符名称的哈希表。

Searching wildcard names hash table is slower than searching exact names hash table because names are searched by domain parts. Note that the special wildcard form “`.example.org`” is stored in a wildcard names hash table and not in an exact names hash table.

​	搜索通配符名称哈希表要比搜索精确名称哈希表慢，因为名称是按域部分搜索的。请注意，特殊的通配符形式 “`.example.org`” 存储在通配符名称哈希表中，而不是精确名称哈希表中。

Regular expressions are tested sequentially and therefore are the slowest method and are non-scalable.

​	正则表达式会依次进行测试，因此是最慢的方法，也是不可扩展的。

For these reasons, it is better to use exact names where possible. For example, if the most frequently requested names of a server are `example.org` and `www.example.org`, it is more efficient to define them explicitly:

​	因此，在可能的情况下最好使用精确名称。例如，如果服务器最常请求的名称是 `example.org` 和 `www.example.org`，则明确定义它们更有效：

```
server {
 listen       80;
 server_name  example.org  www.example.org  *.example.org;
 ...
}
```

than to use the simplified form:

而不是使用简化形式：

```
server {
 listen       80;
 server_name  .example.org;
 ...
}
```



If a large number of server names are defined, or unusually long server names are defined, tuning the [server_names_hash_max_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) and [server_names_hash_bucket_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) directives at the *http* level may become necessary. The default value of the [server_names_hash_bucket_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) directive may be equal to 32, or 64, or another value, depending on CPU cache line size. If the default value is 32 and server name is defined as “`too.long.server.name.example.org`” then nginx will fail to start and display the error message:

​	如果定义了大量的服务器名称，或者定义了异常长的服务器名称，则可能需要调整 *http* 级别上的 [server_names_hash_max_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) 和 [server_names_hash_bucket_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) 指令。[server_names_hash_bucket_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) 指令的默认值可能等于 32，或者等于 64，或者其他值，具体取决于 CPU 缓存线大小。如果默认值是 32，而服务器名称定义为 “`too.long.server.name.example.org`”，则 nginx 将无法启动并显示错误消息：

```
could not build the server_names_hash,
you should increase server_names_hash_bucket_size: 32
```

In this case, the directive value should be increased to the next power of two:

​	在这种情况下，应将指令值增加到下一个二次幂：

```
http {
 server_names_hash_bucket_size  64;
 ...
```

If a large number of server names are defined, another error message will appear:

​	如果定义了大量的服务器名称，还会出现另一个错误消息：

```
could not build the server_names_hash,
you should increase either server_names_hash_max_size: 512
or server_names_hash_bucket_size: 32
```

In such a case, first try to set [server_names_hash_max_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) to a number close to the number of server names. Only if this does not help, or if nginx’s start time is unacceptably long, try to increase [server_names_hash_bucket_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size).

​	在这种情况下，首先尝试将 [server_names_hash_max_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) 设置为接近服务器名称数量的数字。只有在这不起作用，或者 nginx 的启动时间不可接受地长时，才尝试增加 [server_names_hash_bucket_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size)。

If a server is the only server for a listen port, then nginx will not test server names at all (and will not build the hash tables for the listen port). However, there is one exception. If a server name is a regular expression with captures, then nginx has to execute the expression to get the captures.

​	如果服务器是监听端口的唯一服务器，那么 nginx 将根本不会测试服务器名称（也不会为监听端口构建哈希表）。然而，有一个例外情况。如果服务器名称是带有捕获的正则表达式，则 nginx 必须执行表达式以获取捕获。

## 兼容性 Compatibility

- The special server name “`$hostname`” has been supported since 0.9.4.
- A default server name value is an empty name “” since 0.8.48.
- Named regular expression server name captures have been supported since 0.8.25.
- Regular expression server name captures have been supported since 0.7.40.
- An empty server name “” has been supported since 0.7.12.
- A wildcard server name or regular expression has been supported for use as the first server name since 0.6.25.
- Regular expression server names have been supported since 0.6.7.
- Wildcard form `example.*` has been supported since 0.6.0.
- The special form `.example.org` has been supported since 0.3.18.
- Wildcard form `*.example.org` has been supported since 0.1.13.
- 特殊的服务器名称 “`$hostname`” 自 0.9.4 版本开始支持。
- 默认服务器名称值自 0.8.48 版本开始为空名称 “”。
- 命名的正则表达式服务器名称捕获自 0.8.25 版本开始支持。
- 正则表达式服务器名称捕获自 0.7.40 版本开始支持。
- 空服务器名称 “” 自 0.7.12 版本开始支持。
- 自 0.6.25 版本开始支持将通配符服务器名称或正则表达式用作第一个服务器名称。
- 正则表达式服务器名称自 0.6.7 版本开始支持。
- 通配符形式 `example.*` 自 0.6.0 版本开始支持。
- 特殊形式 `.example.org` 自 0.3.18 版本开始支持。
- 通配符形式 `*.example.org` 自 0.1.13 版本开始支持。



written by Igor Sysoev edited by Brian Mercer 