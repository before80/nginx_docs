+++
title = "ngx_http_charset_module"
date = 2023-08-15T08:13:03+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_charset_module

https://nginx.org/en/docs/http/ngx_http_charset_module.html



The `ngx_http_charset_module` module adds the specified charset to the “Content-Type” response header field. In addition, the module can convert data from one charset to another, with some limitations:

​	`ngx_http_charset_module` 模块将指定的字符集添加到“Content-Type”响应头字段中。此外，该模块可以将数据从一个字符集转换为另一个字符集，但有一些限制：

- conversion is performed one way — from server to client,
- only single-byte charsets can be converted
- or single-byte charsets to/from UTF-8.
- 转换是单向的，从服务器到客户端
- 只能转换单字节字符集
- 或者单字节字符集与UTF-8之间的转换





## 示例配置 Example Configuration



```
include        conf/koi-win;

charset        windows-1251;
source_charset koi8-r;
```





## 指令 Directives



### charset

  Syntax:  `charset charset | off;`

  Default: `charset off;`

  Context: `http`, `server`, `location`, `if in location`

Adds the specified charset to the “Content-Type” response header field. If this charset is different from the charset specified in the [source_charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#source_charset) directive, a conversion is performed.

​	将指定的字符集添加到“Content-Type”响应头字段中。如果此字符集与[source_charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#source_charset)指令中指定的字符集不同，则进行转换。

The parameter `off` cancels the addition of charset to the “Content-Type” response header field.

​	参数`off`取消将字符集添加到“Content-Type”响应头字段中。

A charset can be defined with a variable:

​	字符集可以使用变量定义：

```
charset $charset;
```

In such a case, all possible values of a variable need to be present in the configuration at least once in the form of the [charset_map](https://nginx.org/en/docs/http/ngx_http_charset_module.html#charset_map), [charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#charset), or [source_charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#source_charset) directives. For `utf-8`, `windows-1251`, and `koi8-r` charsets, it is sufficient to include the files `conf/koi-win`, `conf/koi-utf`, and `conf/win-utf` into configuration. For other charsets, simply making a fictitious conversion table works, for example:

​	在这种情况下，变量的所有可能值都需要至少以[charset_map](https://nginx.org/en/docs/http/ngx_http_charset_module.html#charset_map)、[charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#charset)或[source_charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#source_charset)指令的形式出现在配置中。对于`utf-8`、`windows-1251`和`koi8-r`字符集，只需将文件`conf/koi-win`、`conf/koi-utf`和`conf/win-utf`包含在配置中即可。对于其他字符集，只需创建一个虚构的转换表，例如：

```
charset_map iso-8859-5 _ { }
```

In addition, a charset can be set in the “X-Accel-Charset” response header field. This capability can be disabled using the [proxy_ignore_headers](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_headers), [fastcgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_ignore_headers), [uwsgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_ignore_headers), [scgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_ignore_headers), and [grpc_ignore_headers](https://nginx.org/en/docs/http/ngx_http_grpc_module.html#grpc_ignore_headers) directives.

​	此外，字符集可以在“X-Accel-Charset”响应头字段中设置。可以使用[proxy_ignore_headers](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_headers)、[fastcgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_ignore_headers)、[uwsgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_ignore_headers)、[scgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_ignore_headers)和[grpc_ignore_headers](https://nginx.org/en/docs/http/ngx_http_grpc_module.html#grpc_ignore_headers)指令禁用此功能。



### charset_map

  Syntax:`charset_map charset1 charset2 { ... }`

  Default: —

  Context: `http`

Describes the conversion table from one charset to another. A reverse conversion table is built using the same data. Character codes are given in hexadecimal. Missing characters in the range 80-FF are replaced with “`?`”. When converting from UTF-8, characters missing in a one-byte charset are replaced with “`&#XXXX;`”.

​	描述从一个字符集到另一个字符集的转换表。使用相同的数据构建反向转换表。字符代码以十六进制给出。范围80-FF中缺少的字符将被替换为“`?`”。从UTF-8转换时，缺少在单字节字符集中的字符将被替换为“`&#XXXX;`”。

Example:

​	示例：

```
charset_map koi8-r windows-1251 {
    C0 FE ; # small yu
    C1 E0 ; # small a
    C2 E1 ; # small b
    C3 F6 ; # small ts
    ...
}
```



When describing a conversion table to UTF-8, codes for the UTF-8 charset should be given in the second column, for example:

​	当描述从转换表到UTF-8时，UTF-8字符集的代码应在第二列中给出，例如：

```
charset_map koi8-r utf-8 {
    C0 D18E ; # small yu
    C1 D0B0 ; # small a
    C2 D0B1 ; # small b
    C3 D186 ; # small ts
    ...
}
```

Full conversion tables from `koi8-r` to `windows-1251`, and from `koi8-r` and `windows-1251` to `utf-8` are provided in the distribution files `conf/koi-win`, `conf/koi-utf`, and `conf/win-utf`.

​	从`koi8-r`到`windows-1251`以及从`koi8-r`和`windows-1251`到`utf-8`的完整转换表包含在分发文件`conf/koi-win`、`conf/koi-utf`和`conf/win-utf`中。

### charset_types

  	Syntax:  `charset_types mime-type ...;`

 Default: `charset_types text/html text/xml text/plain text/vnd.wap.wml application/javascript application/rss+xml;`

 	 Context: `http`, `server`, `location`

This directive appeared in version 0.7.9.

​	此指令出现在0.7.9版本中。

Enables module processing in responses with the specified MIME types in addition to “`text/html`”. The special value “`*`” matches any MIME type (0.8.29).

​	除“`text/html`”外，启用在响应中处理指定MIME类型的模块处理。特殊值“`*`”匹配任何MIME类型（0.8.29 版本）。

Until version 1.5.4, “`application/x-javascript`” was used as the default MIME type instead of “`application/javascript`”.

​	在1.5.4版本之前，默认MIME类型为“`application/x-javascript`”。



### override_charset

  Syntax:  `override_charset on | off;`

  Default: `override_charset off;`

  Context: `http`, `server`, `location`, `if in location`

Determines whether a conversion should be performed for answers received from a proxied or a FastCGI/uwsgi/SCGI/gRPC server when the answers already carry a charset in the “Content-Type” response header field. If conversion is enabled, a charset specified in the received response is used as a source charset.

​	确定是否应对从代理服务器或FastCGI/uwsgi/SCGI/gRPC服务器接收的带有“Content-Type”响应头字段中的字符集的答案执行转换。如果启用转换，则使用接收到的响应中指定的字符集作为源字符集。

It should be noted that if a response is received in a subrequest then the conversion from the response charset to the main request charset is always performed, regardless of the `override_charset` directive setting.

​	应注意，如果在子请求中接收到响应，则始终会执行从响应字符集到主请求字符集的转换，不管`override_charset`指令设置如何。



### source_charset

  Syntax:  `source_charset charset;`

  Default: —

  Context: `http`, `server`, `location`, `if in location`

Defines the source charset of a response. If this charset is different from the charset specified in the [charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#charset) directive, a conversion is performed.

​	定义响应的源字符集。如果此字符集与[charset](https://nginx.org/en/docs/http/ngx_http_charset_module.html#charset)指令中指定的字符集不同，则进行转换。