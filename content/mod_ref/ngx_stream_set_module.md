+++
title = "ngx_stream_set_module"
date = 2023-08-15T08:24:17+08:00
weight = 850
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_set_module

https://nginx.org/en/docs/stream/ngx_stream_set_module.html



The `ngx_stream_set_module` module (1.19.3) allows setting a value for a variable.



## Example Configuration



```
server {
    listen 12345;
    set    $true 1;
}
```





## Directives



### set

  Syntax:`set $variable value;`

| Default: | —                      |
  Context: `server`


Sets a `value` for the specified `variable`. The `value` can contain text, variables, and their combination.