+++
title = "ngx_http_auth_jwt_module"
date = 2023-08-15T08:12:27+08:00
weight = 60
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Module ngx_http_auth_jwt_module

https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html



The `ngx_http_auth_jwt_module` module (1.11.3) implements client authorization by validating the provided [JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519) (JWT) using the specified keys. The module supports [JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515) (JWS), [JSON Web Encryption](https://datatracker.ietf.org/doc/html/rfc7516) (JWE) (1.19.7), and Nested JWT (1.21.0). The module can be used for [OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) authentication.

​	`ngx_http_auth_jwt_module` 模块（1.11.3 版本）通过验证提供的 [JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)（JWT）来实现客户端授权，使用指定的密钥。该模块支持 [JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515)（JWS）、[JSON Web Encryption](https://datatracker.ietf.org/doc/html/rfc7516)（JWE）（1.19.7 版本）和嵌套 JWT（1.21.0 版本）。该模块可以用于 [OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) 身份验证。

The module may be combined with other access modules, such as [ngx_http_access_module](../ngx_http_access_module), [ngx_http_auth_basic_module](../ngx_http_auth_basic_module), and [ngx_http_auth_request_module](../ngx_http_auth_request_module), via the [satisfy]({{< ref "/mod_ref/ngx_http_core_module#satisfy">}}) directive.

​	该模块可以与其他访问模块（如 [ngx_http_access_module](../ngx_http_access_module)、[ngx_http_auth_basic_module](../ngx_http_auth_basic_module) 和 [ngx_http_auth_request_module](../ngx_http_auth_request_module)）结合使用，通过 [satisfy]({{< ref "/mod_ref/ngx_http_core_module#satisfy">}}) 指令。

This module is available as part of our [commercial subscription](http://nginx.com/products/).

​	此模块作为我们的[商业订阅](http://nginx.com/products/)的一部分提供。



## Supported Algorithms

The module supports the following JSON Web [Algorithms](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms).

​	该模块支持以下JSON Web [Algorithms](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms)。

JWS algorithms:

​	JWS 算法：

- HS256, HS384, HS512
- RS256, RS384, RS512
- ES256, ES384, ES512
- EdDSA (Ed25519 and Ed448 signatures) (1.15.7)



Prior to version 1.13.7, only HS256, RS256, ES256 algorithms were supported.

​	在1.13.7版本之前，只支持 HS256、RS256、ES256 算法。

JWE content encryption algorithms (1.19.7):

​	JWE 内容加密算法（1.19.7 版本）：

- A128CBC-HS256, A192CBC-HS384, A256CBC-HS512
- A128GCM, A192GCM, A256GCM



JWE key management algorithms (1.19.9):

​	JWE 密钥管理算法（1.19.9 版本）：

- A128KW, A192KW, A256KW
- A128GCMKW, A192GCMKW, A256GCMKW
- dir - direct use of a shared symmetric key as the content encryption key
- dir - 直接使用共享对称密钥作为内容加密密钥
- RSA-OAEP, RSA-OAEP-256, RSA-OAEP-384, RSA-OAEP-512 (1.21.0)
- RSA-OAEP、RSA-OAEP-256、RSA-OAEP-384、RSA-OAEP-512（1.21.0 版本）





## 示例配置 Example Configuration



```
location / {
    auth_jwt          "closed site";
    auth_jwt_key_file conf/keys.json;
}
```





## 指令 Directives



### auth_jwt

  Syntax:  `auth_jwt string [token=$variable] | off;`

  Default: `auth_jwt off;`

  Context: `http`, `server`, `location`, `limit_except`

Enables validation of JSON Web Token. The specified `string` is used as a `realm`. Parameter value can contain variables.

​	启用 JSON Web Token 的验证。指定的 `string` 用作 `realm`。参数值可以包含变量。

The optional `token` parameter specifies a variable that contains JSON Web Token. By default, JWT is passed in the “Authorization” header as a [Bearer Token](https://datatracker.ietf.org/doc/html/rfc6750). JWT may be also passed as a cookie or a part of a query string:

​	可选的 `token` 参数指定包含 JSON Web Token 的变量。默认情况下，JWT 作为 [Bearer Token](https://datatracker.ietf.org/doc/html/rfc6750) 在 “Authorization” 标头中传递。JWT 也可以作为 cookie 或查询字符串的一部分传递：

```
auth_jwt "closed site" token=$cookie_auth_token;
```

The special value `off` cancels the effect of the `auth_jwt` directive inherited from the previous configuration level.

​	特殊值 `off` 取消从先前配置级别继承的 `auth_jwt` 指令的效果。

### auth_jwt_claim_set

  Syntax:`auth_jwt_claim_set $variable name ...;`

  Default: —

  Context: `http`

This directive appeared in version 1.11.10.

​	此指令出现在 1.11.10 版本中。

Sets the `variable` to a JWT claim parameter identified by key names. Name matching starts from the top level of the JSON tree. For arrays, the variable keeps a list of array elements separated by commas.

​	将 `variable` 设置为由键名标识的 JWT 声明参数。名称匹配从 JSON 树的顶层开始。对于数组，变量保持由逗号分隔的数组元素列表。

```
auth_jwt_claim_set $email info e-mail;
auth_jwt_claim_set $job info "job title";
```

Prior to version 1.13.7, only one key name could be specified, and the result was undefined for arrays.

​	在1.13.7版本之前，只能指定一个键名，对于数组，结果是未定义的。

Variable values for tokens encrypted with JWE are available only after decryption which occurs during the [Access](https://nginx.org/en/docs/dev/development_guide.html#http_phases) phase.

​	JWE 加密的令牌的变量值仅在解密之后才可用，解密发生在[访问](https://nginx.org/en/docs/dev/development_guide.html#http_phases)阶段。



### auth_jwt_header_set

  Syntax:`auth_jwt_header_set $variable name ...;`

  Default: —

  Context: `http`

This directive appeared in version 1.11.10.

​	此指令出现在 1.11.10 版本中。

Sets the `variable` to a JOSE header parameter identified by key names. Name matching starts from the top level of the JSON tree. For arrays, the variable keeps a list of array elements separated by commas.

​	将 `variable` 设置为由键名标识的 JOSE 标头参数。名称匹配从 JSON 树的顶层开始。对于数组，变量保持由逗号分隔的数组元素列表。

Prior to version 1.13.7, only one key name could be specified, and the result was undefined for arrays.

​	在1.13.7版本之前，只能指定一个键名，对于数组，结果是未定义的。



### auth_jwt_key_cache

  Syntax:  `auth_jwt_key_cache time;`

  Default: `auth_jwt_key_cache 0;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.21.4.

​	此指令出现在 1.21.4 版本中。

Enables or disables caching of keys obtained from a [file]({{< ref "/mod_ref/ngx_http_auth_jwt_module#auth_jwt_key_file">}}) or from a [subrequest]({{< ref "/mod_ref/ngx_http_auth_jwt_module#auth_jwt_key_request">}}), and sets caching time for them. Caching of keys obtained from variables is not supported. By default, caching of keys is disabled.

​	启用或禁用从 [file]({{< ref "/mod_ref/ngx_http_auth_jwt_module#auth_jwt_key_file">}}) 或 [subrequest]({{< ref "/mod_ref/ngx_http_auth_jwt_module#auth_jwt_key_request">}}) 获取的密钥的缓存，并为它们设置缓存时间。不支持从变量获取密钥的缓存。默认情况下，禁用密钥的缓存。



### auth_jwt_key_file

  Syntax:  `auth_jwt_key_file file;`

  Default: —

  Context: `http`, `server`, `location`, `limit_except`

Specifies a `file` in [JSON Web Key Set](https://datatracker.ietf.org/doc/html/rfc7517#section-5) format for validating JWT signature. Parameter value can contain variables.

​	指定用于验证 JWT 签名的 [JSON Web Key Set](https://datatracker.ietf.org/doc/html/rfc7517#section-5) 格式的 `file`。参数值可以包含变量。

Several `auth_jwt_key_file` directives can be specified on the same level (1.21.1):

​	在同一级别上可以指定多个 `auth_jwt_key_file` 指令（1.21.1 版本）：

```
auth_jwt_key_file conf/keys.json;
auth_jwt_key_file conf/key.jwk;
```

If at least one of the specified keys cannot be loaded or processed, nginx will return the 500 (Internal Server Error) error.

​	如果至少有一个指定的密钥无法加载或处理，nginx 将返回 500（Internal Server Error）错误。

### auth_jwt_key_request

  Syntax:  `auth_jwt_key_request uri;`

  Default: —

  Context: `http`, `server`, `location`, `limit_except`

This directive appeared in version 1.15.6.

​	此指令出现在 1.15.6 版本中。

Allows retrieving a [JSON Web Key Set](https://datatracker.ietf.org/doc/html/rfc7517#section-5) file from a subrequest for validating JWT signature and sets the URI where the subrequest will be sent to. Parameter value can contain variables. To avoid validation overhead, it is recommended to cache the key file:

​	允许从子请求中检索 [JSON Web Key Set](https://datatracker.ietf.org/doc/html/rfc7517#section-5) 文件以验证 JWT 签名，并设置将发送子请求的 URI。参数值可以包含变量。为了避免验证开销，建议缓存密钥文件：

```
proxy_cache_path /data/nginx/cache levels=1 keys_zone=foo:10m;

server {
    ...

    location / {
        auth_jwt             "closed site";
        auth_jwt_key_request /jwks_uri;
    }

    location = /jwks_uri {
        internal;
        proxy_cache foo;
        proxy_pass  http://idp.example.com/keys;
    }
}
```

Several `auth_jwt_key_request` directives can be specified on the same level (1.21.1):

​	在同一级别上可以指定多个 `auth_jwt_key_request` 指令（1.21.1 版本）：

```
auth_jwt_key_request /jwks_uri;
auth_jwt_key_request /jwks2_uri;
```

If at least one of the specified keys cannot be loaded or processed, nginx will return the 500 (Internal Server Error) error.

​	如果至少有一个指定的密钥无法加载或处理，nginx 将返回 500（Internal Server Error）错误。

### auth_jwt_leeway

  Syntax:  `auth_jwt_leeway time;`

  Default: `auth_jwt_leeway 0s;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.13.10.

​	此指令出现在 1.13.10 版本中。

Sets the maximum allowable leeway to compensate clock skew when verifying the [exp](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.4) and [nbf](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.5) JWT claims.

​	设置验证 [exp](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.4) 和 [nbf](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.5) JWT 声明时的最大允许偏差以补偿时钟偏差。

### auth_jwt_type

  Syntax:`auth_jwt_type signed | encrypted | nested;`

  Default: `auth_jwt_type signed;`

  Context: `http`, `server`, `location`, `limit_except`

This directive appeared in version 1.19.7.

​	此指令出现在 1.19.7 版本中。

Specifies which type of JSON Web Token to expect: JWS (`signed`), JWE (`encrypted`), or signed and then encrypted Nested JWT (`nested`) (1.21.0).

​	指定期望的 JSON Web Token 类型：JWS（`signed`）、JWE（`encrypted`）或先签名再加密的嵌套 JWT（`nested`）（1.21.0 版本）。

### auth_jwt_require

  Syntax:`auth_jwt_require $value ... [error=401 | 403] ;`

  Default: —

  Context: `http`, `server`, `location`, `limit_except`

This directive appeared in version 1.21.2.

​	此指令出现在 1.21.2 版本中。

Specifies additional checks for JWT validation. The value can contain text, variables, and their combination, and must start with a variable (1.21.7). The authentication will succeed only if all the values are not empty and are not equal to “0”.

​	指定 JWT 验证的附加检查。值可以包含文本、变量及其组合，必须以变量（1.21.7 版本）开头。只有在所有值都不为空且不等于“0”时，认证才会成功。

```
map $jwt_claim_iss $valid_jwt_iss {
    "good" 1;
}
...

auth_jwt_require $valid_jwt_iss;
```

If any of the checks fails, the `401` error code is returned. The optional `error` parameter (1.21.7) allows redefining the error code to `403`.

​	如果任何检查失败，将返回 `401` 错误代码。可选的 `error` 参数（1.21.7 版本）允许重新定义错误代码为 `403`。

## 嵌入式变量 Embedded Variables

The `ngx_http_auth_jwt_module` module supports embedded variables:

​	`ngx_http_auth_jwt_module` 模块支持嵌入式变量：

- `$jwt_header_name`

  returns the value of a specified [JOSE header](https://datatracker.ietf.org/doc/html/rfc7515#section-4)

  返回指定的 [JOSE header](https://datatracker.ietf.org/doc/html/rfc7515#section-4) 值

- `$jwt_claim_name`

  returns the value of a specified [JWT claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4)For nested claims and claims including a dot (“.”), the value of the variable cannot be evaluated; the [auth_jwt_claim_set]({{< ref "/mod_ref/ngx_http_auth_jwt_module#auth_jwt_claim_set">}}) directive should be used instead.Variable values for tokens encrypted with JWE are available only after decryption which occurs during the [Access](https://nginx.org/en/docs/dev/development_guide.html#http_phases) phase.

  返回指定的 [JWT claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4) 值。对于嵌套的声明和包含点（“.”）的声明，变量的值无法评估；应使用 [auth_jwt_claim_set]({{< ref "/mod_ref/ngx_http_auth_jwt_module#auth_jwt_claim_set">}}) 指令。JWE 加密的令牌的变量值仅在解密后才可用，解密发生在[访问](https://nginx.org/en/docs/dev/development_guide.html#http_phases)阶段。

- `$jwt_payload`

  returns the decrypted top-level payload of `nested` or `encrypted` tokens (1.21.2). For nested tokens returns the enclosed JWS token. For encrypted tokens returns JSON with claims.
  
  返回 `nested` 或 `encrypted` 令牌（1.21.2 版本）的解密顶层 payload。对于嵌套令牌，返回封装的 JWS 令牌。对于加密令牌，返回带有声明的 JSON。