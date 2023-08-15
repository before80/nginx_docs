+++
title = "ngx_http_f4f_module"
date = 2023-08-15T08:13:28+08:00
weight = 130
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_f4f_module

https://nginx.org/en/docs/http/ngx_http_f4f_module.html



The `ngx_http_f4f_module` module provides server-side support for Adobe HTTP Dynamic Streaming (HDS).

This module implements handling of HTTP Dynamic Streaming requests in the “`/videoSeg1-Frag1`” form — extracting the needed fragment from the `videoSeg1.f4f` file using the `videoSeg1.f4x` index file. This module is an alternative to the Adobe’s f4f module (HTTP Origin Module) for Apache.

Usual pre-processing with Adobe’s f4fpackager is required, see relevant documentation for details.



This module is available as part of our [commercial subscription](http://nginx.com/products/).





## Example Configuration



```
location /video/ {
    f4f;
    ...
}
```





## Directives



### f4f

  Syntax:  `f4f;`

  Default: —

  Context: `location`


Turns on module processing in the surrounding location.



### f4f_buffer_size

  Syntax:  `f4f_buffer_size size;`

  Default: `f4f_buffer_size 512k;`

  Context: `http`, `server`, `location`


Sets the `size` of the buffer used for reading the `.f4x` index file.