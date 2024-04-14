+++
title = "ngx_stream_ssl_module"
date = 2023-08-15T08:24:37+08:00
weight = 870
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_ssl_module

https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html



The `ngx_stream_ssl_module` module (1.9.0) provides the necessary support for a stream proxy server to work with the SSL/TLS protocol. This module is not built by default, it should be enabled with the `--with-stream_ssl_module` configuration parameter.



## Example Configuration

To reduce the processor load, it is recommended to

- set the number of [worker processes]({{< ref "/mod_ref/ngx_core_module#worker_processes">}}) equal to the number of processors,
- enable the [shared](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_session_cache_shared) session cache,
- disable the [built-in](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_session_cache_builtin) session cache,
- and possibly increase the session [lifetime](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_session_timeout) (by default, 5 minutes):



```
worker_processes auto;

stream {

    ...

    server {
        listen              12345 ssl;

        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
        ssl_certificate     /usr/local/nginx/conf/cert.pem;
        ssl_certificate_key /usr/local/nginx/conf/cert.key;
        ssl_session_cache   shared:SSL:10m;
        ssl_session_timeout 10m;

        ...
    }
```





## Directives



### ssl_alpn

  Syntax:`ssl_alpn protocol ...;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.21.4.

Specifies the list of supported [ALPN](https://datatracker.ietf.org/doc/html/rfc7301) protocols. One of the protocols must be [negotiated](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#var_ssl_alpn_protocol) if the client uses ALPN:

```
map $ssl_alpn_protocol $proxy {
    h2                127.0.0.1:8001;
    http/1.1          127.0.0.1:8002;
}

server {
    listen      12346;
    proxy_pass  $proxy;
    ssl_alpn    h2 http/1.1;
}
```





### ssl_certificate

  Syntax:`ssl_certificate file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with the certificate in the PEM format for the given server. If intermediate certificates should be specified in addition to a primary certificate, they should be specified in the same file in the following order: the primary certificate comes first, then the intermediate certificates. A secret key in the PEM format may be placed in the same file.

Since version 1.11.0, this directive can be specified multiple times to load certificates of different types, for example, RSA and ECDSA:

```
server {
    listen              12345 ssl;

    ssl_certificate     example.com.rsa.crt;
    ssl_certificate_key example.com.rsa.key;

    ssl_certificate     example.com.ecdsa.crt;
    ssl_certificate_key example.com.ecdsa.key;

    ...
}
```



Only OpenSSL 1.0.2 or higher supports separate certificate chains for different certificates. With older versions, only one certificate chain can be used.



Since version 1.15.9, variables can be used in the `file` name when using OpenSSL 1.0.2 or higher:

```
ssl_certificate     $ssl_server_name.crt;
ssl_certificate_key $ssl_server_name.key;
```

Note that using variables implies that a certificate will be loaded for each SSL handshake, and this may have a negative impact on performance.



The value `data`:`$variable` can be specified instead of the `file` (1.15.10), which loads a certificate from a variable without using intermediate files. Note that inappropriate use of this syntax may have its security implications, such as writing secret key data to [error log]({{< ref "/mod_ref/ngx_core_module#error_log">}}).



### ssl_certificate_key

  Syntax:`ssl_certificate_key file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with the secret key in the PEM format for the given server.

The value `engine`:`name`:`id` can be specified instead of the `file`, which loads a secret key with a specified `id` from the OpenSSL engine `name`.



The value `data`:`$variable` can be specified instead of the `file` (1.15.10), which loads a secret key from a variable without using intermediate files. Note that inappropriate use of this syntax may have its security implications, such as writing secret key data to [error log]({{< ref "/mod_ref/ngx_core_module#error_log">}}).

Since version 1.15.9, variables can be used in the `file` name when using OpenSSL 1.0.2 or higher.



### ssl_ciphers

  Syntax:  `ssl_ciphers ciphers;`

  Default: `ssl_ciphers HIGH:!aNULL:!MD5;`

  Context: `stream`, `server`


Specifies the enabled ciphers. The ciphers are specified in the format understood by the OpenSSL library, for example:

```
ssl_ciphers ALL:!aNULL:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
```



The full list can be viewed using the “`openssl ciphers`” command.



### ssl_client_certificate

  Syntax:`ssl_client_certificate file;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.11.8.

Specifies a `file` with trusted CA certificates in the PEM format used to [verify](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_verify_client) client certificates.

The list of certificates will be sent to clients. If this is not desired, the [ssl_trusted_certificate](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_trusted_certificate) directive can be used.



### ssl_conf_command

  Syntax:`ssl_conf_command name value;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.19.4.

Sets arbitrary OpenSSL configuration [commands](https://www.openssl.org/docs/man1.1.1/man3/SSL_CONF_cmd.html).

The directive is supported when using OpenSSL 1.0.2 or higher.



Several `ssl_conf_command` directives can be specified on the same level:

```
ssl_conf_command Options PrioritizeChaCha;
ssl_conf_command Ciphersuites TLS_CHACHA20_POLY1305_SHA256;
```

These directives are inherited from the previous configuration level if and only if there are no `ssl_conf_command` directives defined on the current level.



Note that configuring OpenSSL directly might result in unexpected behavior.





### ssl_crl

  Syntax:  `ssl_crl file;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.11.8.

Specifies a `file` with revoked certificates (CRL) in the PEM format used to [verify](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_verify_client) client certificates.



### ssl_dhparam

  Syntax:`ssl_dhparam file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with DH parameters for DHE ciphers.

By default no parameters are set, and therefore DHE ciphers will not be used.

Prior to version 1.11.0, builtin parameters were used by default.





### ssl_ecdh_curve

  Syntax:`ssl_ecdh_curve curve;`

  Default: `ssl_ecdh_curve auto;`

  Context: `stream`, `server`


Specifies a `curve` for ECDHE ciphers.

When using OpenSSL 1.0.2 or higher, it is possible to specify multiple curves (1.11.0), for example:

```
ssl_ecdh_curve prime256v1:secp384r1;
```



The special value `auto` (1.11.0) instructs nginx to use a list built into the OpenSSL library when using OpenSSL 1.0.2 or higher, or `prime256v1` with older versions.



Prior to version 1.11.0, the `prime256v1` curve was used by default.





When using OpenSSL 1.0.2 or higher, this directive sets the list of curves supported by the server. Thus, in order for ECDSA certificates to work, it is important to include the curves used in the certificates.





### ssl_handshake_timeout

  Syntax:`ssl_handshake_timeout time;`

  Default: `ssl_handshake_timeout 60s;`

  Context: `stream`, `server`


Specifies a timeout for the SSL handshake to complete.



### ssl_password_file

  Syntax:`ssl_password_file file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with passphrases for [secret keys](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_certificate_key) where each passphrase is specified on a separate line. Passphrases are tried in turn when loading the key.

Example:

```
stream {
    ssl_password_file /etc/keys/global.pass;
    ...

    server {
        listen 127.0.0.1:12345;
        ssl_certificate_key /etc/keys/first.key;
    }

    server {
        listen 127.0.0.1:12346;

        # named pipe can also be used instead of a file
        ssl_password_file /etc/keys/fifo;
        ssl_certificate_key /etc/keys/second.key;
    }
}
```





### ssl_prefer_server_ciphers

  Syntax:`ssl_prefer_server_ciphers on | off;`

  Default: `ssl_prefer_server_ciphers off;`

  Context: `stream`, `server`


Specifies that server ciphers should be preferred over client ciphers when the SSLv3 and TLS protocols are used.



### ssl_protocols

  Syntax:`ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];`

  Default: `ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;`

  Context: `stream`, `server`


Enables the specified protocols.

The `TLSv1.1` and `TLSv1.2` parameters work only when OpenSSL 1.0.1 or higher is used.



The `TLSv1.3` parameter (1.13.0) works only when OpenSSL 1.1.1 or higher is used.



The `TLSv1.3` parameter is used by default since 1.23.4.





### ssl_session_cache

  Syntax:`ssl_session_cache off | none | [builtin[:size]] [shared:name:size];`

  Default: `ssl_session_cache none;`

  Context: `stream`, `server`


Sets the types and sizes of caches that store session parameters. A cache can be of any of the following types:

- `off`

  the use of a session cache is strictly prohibited: nginx explicitly tells a client that sessions may not be reused.

- `none`

  the use of a session cache is gently disallowed: nginx tells a client that sessions may be reused, but does not actually store session parameters in the cache.

- `builtin`

  a cache built in OpenSSL; used by one worker process only. The cache size is specified in sessions. If size is not given, it is equal to 20480 sessions. Use of the built-in cache can cause memory fragmentation.

- `shared`

  a cache shared between all worker processes. The cache size is specified in bytes; one megabyte can store about 4000 sessions. Each shared cache should have an arbitrary name. A cache with the same name can be used in several servers. It is also used to automatically generate, store, and periodically rotate TLS session ticket keys (1.23.2) unless configured explicitly using the [ssl_session_ticket_key](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_session_ticket_key) directive.



Both cache types can be used simultaneously, for example:

```
ssl_session_cache builtin:1000 shared:SSL:10m;
```

but using only shared cache without the built-in cache should be more efficient.



### ssl_session_ticket_key

  Syntax:`ssl_session_ticket_key file;`

  Default: —

  Context: `stream`, `server`


Sets a `file` with the secret key used to encrypt and decrypt TLS session tickets. The directive is necessary if the same key has to be shared between multiple servers. By default, a randomly generated key is used.

If several keys are specified, only the first key is used to encrypt TLS session tickets. This allows configuring key rotation, for example:

```
ssl_session_ticket_key current.key;
ssl_session_ticket_key previous.key;
```



The `file` must contain 80 or 48 bytes of random data and can be created using the following command:

```
openssl rand 80 > ticket.key
```

Depending on the file size either AES256 (for 80-byte keys, 1.11.8) or AES128 (for 48-byte keys) is used for encryption.



### ssl_session_tickets

  Syntax:`ssl_session_tickets on | off;`

  Default: `ssl_session_tickets on;`

  Context: `stream`, `server`


Enables or disables session resumption through [TLS session tickets](https://datatracker.ietf.org/doc/html/rfc5077).



### ssl_session_timeout

  Syntax:`ssl_session_timeout time;`

  Default: `ssl_session_timeout 5m;`

  Context: `stream`, `server`


Specifies a time during which a client may reuse the session parameters.



### ssl_trusted_certificate

  Syntax:`ssl_trusted_certificate file;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.11.8.

Specifies a `file` with trusted CA certificates in the PEM format used to [verify](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_verify_client) client certificates.

In contrast to the certificate set by [ssl_client_certificate](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_client_certificate), the list of these certificates will not be sent to clients.



### ssl_verify_client

  Syntax:`ssl_verify_client on | off | optional | optional_no_ca;`

  Default: `ssl_verify_client off;`

  Context: `stream`, `server`


This directive appeared in version 1.11.8.

Enables verification of client certificates. The verification result is stored in the [$ssl_client_verify](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#var_ssl_client_verify) variable. If an error has occurred during the client certificate verification or a client has not presented the required certificate, the connection is closed.

The `optional` parameter requests the client certificate and verifies it if the certificate is present.

The `optional_no_ca` parameter requests the client certificate but does not require it to be signed by a trusted CA certificate. This is intended for the use in cases when a service that is external to nginx performs the actual certificate verification. The contents of the certificate is accessible through the [$ssl_client_cert](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#var_ssl_client_cert) variable.



### ssl_verify_depth

  Syntax:`ssl_verify_depth number;`

  Default: `ssl_verify_depth 1;`

  Context: `stream`, `server`


This directive appeared in version 1.11.8.

Sets the verification depth in the client certificates chain.



## Embedded Variables

The `ngx_stream_ssl_module` module supports variables since 1.11.2.

### `$ssl_alpn_protocol`

  returns the protocol selected by ALPN during the SSL handshake, or an empty string otherwise (1.21.4);

### `$ssl_cipher`

  returns the name of the cipher used for an established SSL connection;

### `$ssl_ciphers`

  returns the list of ciphers supported by the client (1.11.7). Known ciphers are listed by names, unknown are shown in hexadecimal, for example:`AES128-SHA:AES256-SHA:0x00ff `The variable is fully supported only when using OpenSSL version 1.0.2 or higher. With older versions, the variable is available only for new sessions and lists only known ciphers.

### `$ssl_client_cert`

  returns the client certificate in the PEM format for an established SSL connection, with each line except the first prepended with the tab character (1.11.8);

### `$ssl_client_fingerprint`

  returns the SHA1 fingerprint of the client certificate for an established SSL connection (1.11.8);

### `$ssl_client_i_dn`

  returns the “issuer DN” string of the client certificate for an established SSL connection according to [RFC 2253](https://datatracker.ietf.org/doc/html/rfc2253) (1.11.8);

### `$ssl_client_raw_cert`

  returns the client certificate in the PEM format for an established SSL connection (1.11.8);

### `$ssl_client_s_dn`

  returns the “subject DN” string of the client certificate for an established SSL connection according to [RFC 2253](https://datatracker.ietf.org/doc/html/rfc2253) (1.11.8);

### `$ssl_client_serial`

  returns the serial number of the client certificate for an established SSL connection (1.11.8);

### `$ssl_client_v_end`

  returns the end date of the client certificate (1.11.8);

### `$ssl_client_v_remain`

  returns the number of days until the client certificate expires (1.11.8);

### `$ssl_client_v_start`

  returns the start date of the client certificate (1.11.8);

### `$ssl_client_verify`

  returns the result of client certificate verification (1.11.8): “`SUCCESS`”, “`FAILED:` `reason`”, and “`NONE`” if a certificate was not present;

### `$ssl_curve`

  returns the negotiated curve used for SSL handshake key exchange process (1.21.5). Known curves are listed by names, unknown are shown in hexadecimal, for example:`prime256v1 `The variable is supported only when using OpenSSL version 3.0 or higher. With older versions, the variable value will be an empty string.

### `$ssl_curves`

  returns the list of curves supported by the client (1.11.7). Known curves are listed by names, unknown are shown in hexadecimal, for example:`0x001d:prime256v1:secp521r1:secp384r1 `The variable is supported only when using OpenSSL version 1.0.2 or higher. With older versions, the variable value will be an empty string.The variable is available only for new sessions.

### `$ssl_protocol`

  returns the protocol of an established SSL connection;

### `$ssl_server_name`

  returns the server name requested through [SNI](http://en.wikipedia.org/wiki/Server_Name_Indication);

### `$ssl_session_id`

  returns the session identifier of an established SSL connection;

### `$ssl_session_reused`

  returns “`r`” if an SSL session was reused, or “`.`” otherwise.