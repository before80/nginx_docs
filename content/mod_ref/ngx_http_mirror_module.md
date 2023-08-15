+++
title = "ngx_http_mirror_module"
date = 2023-08-15T08:16:39+08:00
weight = 340
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_mirror_module

https://nginx.org/en/docs/http/ngx_http_mirror_module.html



The `ngx_http_mirror_module` module (1.13.4) implements mirroring of an original request by creating background mirror subrequests. Responses to mirror subrequests are ignored.



## Example Configuration



```
location / {
    mirror /mirror;
    proxy_pass http://backend;
}

location = /mirror {
    internal;
    proxy_pass http://test_backend$request_uri;
}
```





## Directives



### mirror

  Syntax:  `mirror uri | off;`

  Default: `mirror off;`

  Context: `http`, `server`, `location`


Sets the URI to which an original request will be mirrored. Several mirrors can be specified on the same configuration level.



### mirror_request_body

  Syntax:`mirror_request_body on | off;`

  Default: `mirror_request_body on;`

  Context: `http`, `server`, `location`


Indicates whether the client request body is mirrored. When enabled, the client request body will be read prior to creating mirror subrequests. In this case, unbuffered client request body proxying set by the [proxy_request_buffering](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_request_buffering), [fastcgi_request_buffering](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_request_buffering), [scgi_request_buffering](https://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_request_buffering), and [uwsgi_request_buffering](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_request_buffering) directives will be disabled.

```
location / {
    mirror /mirror;
    mirror_request_body off;
    proxy_pass http://backend;
}

location = /mirror {
    internal;
    proxy_pass http://log_backend;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
    proxy_set_header X-Original-URI $request_uri;
}
```