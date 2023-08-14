+++
title = "njs 脚本语言"
date = 2023-08-14T16:57:04+08:00
weight = 190
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# njs scripting language - njs 脚本语言

https://nginx.org/en/docs/njs/index.html



njs is a subset of the JavaScript language that allows extending nginx functionality. njs is created in compliance with [ECMAScript 5.1](http://www.ecma-international.org/ecma-262/5.1/) (strict mode) with some [ECMAScript 6](http://www.ecma-international.org/ecma-262/6.0/) and later extensions. The compliance is still [evolving](https://nginx.org/en/docs/njs/compatibility.html).

​	njs 是 JavaScript 语言的一个子集，它允许扩展 nginx 的功能。njs 是按照 [ECMAScript 5.1](http://www.ecma-international.org/ecma-262/5.1/)（严格模式）创建的，带有一些 [ECMAScript 6](http://www.ecma-international.org/ecma-262/6.0/) 和后续的扩展。遵循程度仍在不断地[发展中](https://nginx.org/en/docs/njs/compatibility.html)。

- [Download and install](https://nginx.org/en/docs/njs/install.html)
- [Changes](https://nginx.org/en/docs/njs/changes.html)
- [Reference](https://nginx.org/en/docs/njs/reference.html)
- [Examples](https://github.com/nginx/njs-examples/)
- [Security](https://nginx.org/en/docs/njs/security.html)
- [Compatibility](https://nginx.org/en/docs/njs/compatibility.html)
- [Command-line interface](https://nginx.org/en/docs/njs/cli.html)
- [Understanding preloaded objects](https://nginx.org/en/docs/njs/preload_objects.html)
- [Tested OS and platforms](https://nginx.org/en/docs/njs/index.html#tested_os_and_platforms)

- [ngx_http_js_module](https://nginx.org/en/docs/http/ngx_http_js_module.html)
- [ngx_stream_js_module](https://nginx.org/en/docs/stream/ngx_stream_js_module.html)

- [Writing njs code using TypeScript definition files](https://nginx.org/en/docs/njs/typescript.html)
- [Using node modules with njs](https://nginx.org/en/docs/njs/node_modules.html)
- [下载和安装](https://nginx.org/en/docs/njs/install.html)
- [变更](https://nginx.org/en/docs/njs/changes.html)
- [参考](https://nginx.org/en/docs/njs/reference.html)
- [示例](https://github.com/nginx/njs-examples/)
- [安全性](https://nginx.org/en/docs/njs/security.html)
- [兼容性](https://nginx.org/en/docs/njs/compatibility.html)
- [命令行界面](https://nginx.org/en/docs/njs/cli.html)
- [理解预加载对象](https://nginx.org/en/docs/njs/preload_objects.html)
- [经过测试的操作系统和平台](https://nginx.org/en/docs/njs/index.html#tested_os_and_platforms)
- [ngx_http_js_module](https://nginx.org/en/docs/http/ngx_http_js_module.html)
- [ngx_stream_js_module](https://nginx.org/en/docs/stream/ngx_stream_js_module.html)
- [使用 TypeScript 定义文件编写 njs 代码](https://nginx.org/en/docs/njs/typescript.html)
- [在 njs 中使用 Node 模块](https://nginx.org/en/docs/njs/node_modules.html)





## 使用案例 - Use cases



- Complex access control and security checks in njs before a request reaches an upstream server
- 在 njs 中进行复杂的访问控制和安全性检查，以防止请求达到上游服务器之前
- Manipulating response headers
- 操纵响应头
- Writing flexible asynchronous content handlers and filters
- 编写灵活的异步内容处理程序和过滤器

See [examples](https://github.com/nginx/njs-examples/) and [blog posts](https://www.nginx.com/blog/tag/nginx-javascript-module/) for more njs use cases.

​	有关更多 njs 使用案例，请参见 [示例](https://github.com/nginx/njs-examples/) 和 [博客文章](https://www.nginx.com/blog/tag/nginx-javascript-module/)。



## 基本的 HTTP 示例 - Basic HTTP Example

To use njs in nginx:

​	要在 nginx 中使用 njs：

- [install](https://nginx.org/en/docs/njs/install.html) njs scripting language

- [安装](https://nginx.org/en/docs/njs/install.html) njs 脚本语言

- create an njs script file, for example, `http.js`. See [Reference](https://nginx.org/en/docs/njs/reference.html) for the list of njs properties and methods.

- 创建一个 njs 脚本文件，例如 `http.js`。请参阅 [参考](https://nginx.org/en/docs/njs/reference.html) 获取 njs 属性和方法的列表。

  ```
  function hello(r) {
   r.return(200, "Hello world!");
  }
  
  export default {hello};
  ```

  

- in the `nginx.conf` file, enable [ngx_http_js_module](https://nginx.org/en/docs/http/ngx_http_js_module.html) module and specify the [js_import](https://nginx.org/en/docs/http/ngx_http_js_module.html#js_import) directive with the `http.js` script file:

- 在 `nginx.conf` 文件中，启用 [ngx_http_js_module](https://nginx.org/en/docs/http/ngx_http_js_module.html) 模块，并使用 `http.js` 脚本文件指定 [js_import](https://nginx.org/en/docs/http/ngx_http_js_module.html#js_import) 指令：

  ```
  load_module modules/ngx_http_js_module.so;
  
  events {}
  
  http {
   js_import http.js;
  
   server {
       listen 8000;
  
       location / {
           js_content http.hello;
       }
   }
  }
  ```

  

There is also a standalone [command line](https://nginx.org/en/docs/njs/cli.html) utility that can be used independently of nginx for njs development and debugging.

​	还有一个独立的 [命令行](https://nginx.org/en/docs/njs/cli.html) 实用程序，可用于独立于 nginx 进行 njs 的开发和调试。



Tested OS and platforms

​	经过测试的操作系统和平台

- FreeBSD / amd64;
- Linux / x86, amd64, arm64, ppc64el;
- Solaris 11 / amd64;
- macOS / x86_64;



## 在 nginx.conf 2018 上的演示 - Presentation at nginx.conf 2018



<iframe type="text/html" src="https://www.youtube.com/embed/Jc_L6UffFOs?modestbranding=1&amp;rel=0&amp;showinfo=0&amp;color=white" frameborder="0" allowfullscreen="1" style="top: 0px; left: 0px; width: 780.802px; height: 439.198px;"></iframe>

