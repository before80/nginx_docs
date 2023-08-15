+++
title = "ngx_http_auth_basic_module"
date = 2023-08-15T08:12:18+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

## Module ngx_http_auth_basic_module

https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html



The `ngx_http_auth_basic_module` module allows limiting access to resources by validating the user name and password using the “HTTP Basic Authentication” protocol.

Access can also be limited by [address](https://nginx.org/en/docs/http/ngx_http_access_module.html), by the [result of subrequest](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html), or by [JWT](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html). Simultaneous limitation of access by address and by password is controlled by the [satisfy](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) directive.



## Example Configuration



> ```
> location / {
>     auth_basic           "closed site";
>     auth_basic_user_file conf/htpasswd;
> }
> ```





## Directives



### auth_basic

| Syntax:  | `auth_basic string | off;`                   |
| :------- | -------------------------------------------- |
| Default: | `auth_basic off;`                            |
| Context: | `http`, `server`, `location`, `limit_except` |

Enables validation of user name and password using the “HTTP Basic Authentication” protocol. The specified parameter is used as a `realm`. Parameter value can contain variables (1.3.10, 1.2.7). The special value `off` cancels the effect of the `auth_basic` directive inherited from the previous configuration level.



### auth_basic_user_file

| Syntax:  | `auth_basic_user_file file;`                 |
| :------- | -------------------------------------------- |
| Default: | —                                            |
| Context: | `http`, `server`, `location`, `limit_except` |

Specifies a file that keeps user names and passwords, in the following format:

> ```
> # comment
> name1:password1
> name2:password2:comment
> name3:password3
> ```

- The `file` name can contain variables.

  The following password types are supported:

  - encrypted with the `crypt()` function; can be generated using the “`htpasswd`” utility from the Apache HTTP Server distribution or the “`openssl passwd`” command;
  - hashed with the Apache variant of the MD5-based password algorithm (apr1); can be generated with the same tools;
  - specified by the “`{scheme}data`” syntax (1.0.3+) as described in [RFC 2307](https://datatracker.ietf.org/doc/html/rfc2307#section-5.3); currently implemented schemes include `PLAIN` (an example one, should not be used), `SHA` (1.3.13) (plain SHA-1 hashing, should not be used) and `SSHA` (salted SHA-1 hashing, used by some software packages, notably OpenLDAP and Dovecot).

  > Support for `SHA` scheme was added only to aid in migration from other web servers. It should not be used for new passwords, since unsalted SHA-1 hashing that it employs is vulnerable to [rainbow table](http://en.wikipedia.org/wiki/Rainbow_attack) attacks.