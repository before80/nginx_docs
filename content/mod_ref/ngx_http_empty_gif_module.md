+++
title = "ngx_http_empty_gif_module"
date = 2023-08-15T08:13:20+08:00
weight = 120
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_empty_gif_module

https://nginx.org/en/docs/http/ngx_http_empty_gif_module.html



The `ngx_http_empty_gif_module` module emits single-pixel transparent GIF.



## Example Configuration



```
location = /_.gif {
    empty_gif;
}
```





## Directives



### empty_gif

  Syntax:`empty_gif;`

  Default: —

  Context: `location`


Turns on module processing in a surrounding location.