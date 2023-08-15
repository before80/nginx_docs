+++
title = "ngx_http_internal_redirect_module"
date = 2023-08-15T08:15:31+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_internal_redirect_module

https://nginx.org/en/docs/http/ngx_http_internal_redirect_module.html



The `ngx_http_internal_redirect_module` module (1.23.4) allows making an internal redirect. In contrast to [rewriting URIs](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html), the redirection is made after checking [request](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) and [connection](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) processing limits, and [access](https://nginx.org/en/docs/http/ngx_http_access_module.html) limits.



> This module is available as part of our [commercial subscription](http://nginx.com/products/).





## Example Configuration



> ```
> limit_req_zone $jwt_claim_sub zone=jwt_sub:10m rate=1r/s;
> 
> server {
>     location / {
>         auth_jwt          "realm";
>         auth_jwt_key_file key.jwk;
> 
>         internal_redirect @rate_limited;
>     }
> 
>     location @rate_limited {
>         internal;
> 
>         limit_req  zone=jwt_sub burst=10;
>         proxy_pass http://backend;
>     }
> }
> ```

The example implements [per-user](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.2) [rate limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html). Implementation without [internal_redirect](https://nginx.org/en/docs/http/ngx_http_internal_redirect_module.html#internal_redirect) is vulnerable to DoS attacks by unsigned JWTs, as normally the [limit_req](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req) check is performed [before](https://nginx.org/en/docs/dev/development_guide.html#http_phases) [auth_jwt](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt) check. Using [internal_redirect](https://nginx.org/en/docs/http/ngx_http_internal_redirect_module.html#internal_redirect) allows reordering these checks.



## Directives



### internal_redirect

| Syntax:  | `internal_redirect uri;` |
| :------- | ------------------------ |
| Default: | —                        |
| Context: | `server`, `location`     |

Sets the URI for internal redirection of the request. It is also possible to use a [named location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location_named) instead of the URI. The `uri` value can contain variables. If the `uri` value is empty, then the redirect will not be made.