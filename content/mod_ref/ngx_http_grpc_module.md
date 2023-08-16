+++
title = "ngx_http_grpc_module"
date = 2023-08-15T08:14:14+08:00
weight = 180
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_grpc_module

https://nginx.org/en/docs/http/ngx_http_grpc_module.html



The `ngx_http_grpc_module` module allows passing requests to a gRPC server (1.13.10). The module requires the [ngx_http_v2_module](https://nginx.org/en/docs/http/ngx_http_v2_module.html) module.



## Example Configuration



```
server {
    listen 9000;

    http2 on;

    location / {
        grpc_pass 127.0.0.1:9000;
    }
}
```





## Directives



### grpc_bind

  Syntax:`grpc_bind address [transparent ] | off;`

  Default: —

  Context: `http`, `server`, `location`


Makes outgoing connections to a gRPC server originate from the specified local IP address with an optional port. Parameter value can contain variables. The special value `off` cancels the effect of the `grpc_bind` directive inherited from the previous configuration level, which allows the system to auto-assign the local IP address and port.



The `transparent` parameter allows outgoing connections to a gRPC server originate from a non-local IP address, for example, from a real IP address of a client:

```
grpc_bind $remote_addr transparent;
```

In order for this parameter to work, it is usually necessary to run nginx worker processes with the [superuser]({{< ref "/mod_ref/ngx_core_module#user">}}) privileges. On Linux it is not required as if the `transparent` parameter is specified, worker processes inherit the `CAP_NET_RAW` capability from the master process. It is also necessary to configure kernel routing table to intercept network traffic from the gRPC server.



### grpc_buffer_size

  Syntax:  `grpc_buffer_size size;`

  Default: `grpc_buffer_size 4k|8k;`

  Context: `http`, `server`, `location`


Sets the `size` of the buffer used for reading the response received from the gRPC server. The response is passed to the client synchronously, as soon as it is received.



### grpc_connect_timeout

  Syntax:`grpc_connect_timeout time;`

  Default: `grpc_connect_timeout 60s;`

  Context: `http`, `server`, `location`


Defines a timeout for establishing a connection with a gRPC server. It should be noted that this timeout cannot usually exceed 75 seconds.



### grpc_hide_header

  Syntax:  `grpc_hide_header field;`

  Default: —

  Context: `http`, `server`, `location`


By default, nginx does not pass the header fields “Date”, “Server”, and “X-Accel-...” from the response of a gRPC server to a client. The `grpc_hide_header` directive sets additional fields that will not be passed. If, on the contrary, the passing of fields needs to be permitted, the [grpc_pass_header]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_pass_header">}}) directive can be used.



### grpc_ignore_headers

  Syntax:`grpc_ignore_headers field ...;`

  Default: —

  Context: `http`, `server`, `location`


Disables processing of certain response header fields from the gRPC server. The following fields can be ignored: “X-Accel-Redirect” and “X-Accel-Charset”.

If not disabled, processing of these header fields has the following effect:

- “X-Accel-Redirect” performs an [internal redirect]({{< ref "/mod_ref/ngx_http_core_module#internal">}}) to the specified URI;
- “X-Accel-Charset” sets the desired [charset]({{< ref "/mod_ref/ngx_http_charset_module#charset">}}) of a response.





### grpc_intercept_errors

  Syntax:`grpc_intercept_errors on | off;`

  Default: `grpc_intercept_errors off;`

  Context: `http`, `server`, `location`


Determines whether gRPC server responses with codes greater than or equal to 300 should be passed to a client or be intercepted and redirected to nginx for processing with the [error_page]({{< ref "/mod_ref/ngx_http_core_module#error_page">}}) directive.



### grpc_next_upstream

  Syntax:`grpc_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | http_429 | non_idempotent | off ...;`

  Default: `grpc_next_upstream error timeout;`

  Context: `http`, `server`, `location`


Specifies in which cases a request should be passed to the next server:

- `error`

  an error occurred while establishing a connection with the server, passing a request to it, or reading the response header;

- `timeout`

  a timeout has occurred while establishing a connection with the server, passing a request to it, or reading the response header;

- `invalid_header`

  a server returned an empty or invalid response;

- `http_500`

  a server returned a response with the code 500;

- `http_502`

  a server returned a response with the code 502;

- `http_503`

  a server returned a response with the code 503;

- `http_504`

  a server returned a response with the code 504;

- `http_403`

  a server returned a response with the code 403;

- `http_404`

  a server returned a response with the code 404;

- `http_429`

  a server returned a response with the code 429;

- `non_idempotent`

  normally, requests with a [non-idempotent](https://datatracker.ietf.org/doc/html/rfc7231#section-4.2.2) method (`POST`, `LOCK`, `PATCH`) are not passed to the next server if a request has been sent to an upstream server; enabling this option explicitly allows retrying such requests;

- `off`

  disables passing a request to the next server.



One should bear in mind that passing a request to the next server is only possible if nothing has been sent to a client yet. That is, if an error or timeout occurs in the middle of the transferring of a response, fixing this is impossible.

The directive also defines what is considered an [unsuccessful attempt]({{< ref "/mod_ref/ngx_http_upstream_module#max_fails">}}) of communication with a server. The cases of `error`, `timeout` and `invalid_header` are always considered unsuccessful attempts, even if they are not specified in the directive. The cases of `http_500`, `http_502`, `http_503`, `http_504`, and `http_429` are considered unsuccessful attempts only if they are specified in the directive. The cases of `http_403` and `http_404` are never considered unsuccessful attempts.

Passing a request to the next server can be limited by [the number of tries]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_next_upstream_tries">}}) and by [time]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_next_upstream_timeout">}}).



### grpc_next_upstream_timeout

  Syntax:`grpc_next_upstream_timeout time;`

  Default: `grpc_next_upstream_timeout 0;`

  Context: `http`, `server`, `location`


Limits the time during which a request can be passed to the [next server]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_next_upstream">}}). The `0` value turns off this limitation.



### grpc_next_upstream_tries

  Syntax:`grpc_next_upstream_tries number;`

  Default: `grpc_next_upstream_tries 0;`

  Context: `http`, `server`, `location`


Limits the number of possible tries for passing a request to the [next server]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_next_upstream">}}). The `0` value turns off this limitation.



### grpc_pass

  Syntax:  `grpc_pass address;`

  Default: —

  Context: `location`, `if in location`


Sets the gRPC server address. The address can be specified as a domain name or IP address, and a port:

```
grpc_pass localhost:9000;
```

or as a UNIX-domain socket path:

```
grpc_pass unix:/tmp/grpc.socket;
```

Alternatively, the “`grpc://`” scheme can be used:

```
grpc_pass grpc://127.0.0.1:9000;
```

To use gRPC over SSL, the “`grpcs://`” scheme should be used:

```
grpc_pass grpcs://127.0.0.1:443;
```



If a domain name resolves to several addresses, all of them will be used in a round-robin fashion. In addition, an address can be specified as a [server group](https://nginx.org/en/docs/http/ngx_http_upstream_module.html).

Parameter value can contain variables (1.17.8). In this case, if an address is specified as a domain name, the name is searched among the described [server groups](https://nginx.org/en/docs/http/ngx_http_upstream_module.html), and, if not found, is determined using a [resolver]({{< ref "/mod_ref/ngx_http_core_module#resolver">}}).



### grpc_pass_header

  Syntax:  `grpc_pass_header field;`

  Default: —

  Context: `http`, `server`, `location`


Permits passing [otherwise disabled]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_hide_header">}}) header fields from a gRPC server to a client.



### grpc_read_timeout

  Syntax:  `grpc_read_timeout time;`

  Default: `grpc_read_timeout 60s;`

  Context: `http`, `server`, `location`


Defines a timeout for reading a response from the gRPC server. The timeout is set only between two successive read operations, not for the transmission of the whole response. If the gRPC server does not transmit anything within this time, the connection is closed.



### grpc_send_timeout

  Syntax:  `grpc_send_timeout time;`

  Default: `grpc_send_timeout 60s;`

  Context: `http`, `server`, `location`


Sets a timeout for transmitting a request to the gRPC server. The timeout is set only between two successive write operations, not for the transmission of the whole request. If the gRPC server does not receive anything within this time, the connection is closed.



### grpc_set_header

  Syntax:  `grpc_set_header field value;`

  Default: `grpc_set_header Content-Length $content_length;`

  Context: `http`, `server`, `location`


Allows redefining or appending fields to the request header [passed]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_pass_request_headers">}}) to the gRPC server. The `value` can contain text, variables, and their combinations. These directives are inherited from the previous configuration level if and only if there are no `grpc_set_header` directives defined on the current level.

If the value of a header field is an empty string then this field will not be passed to a gRPC server:

```
grpc_set_header Accept-Encoding "";
```





### grpc_socket_keepalive

  Syntax:`grpc_socket_keepalive on | off;`

  Default: `grpc_socket_keepalive off;`

  Context: `http`, `server`, `location`


This directive appeared in version 1.15.6.

Configures the “TCP keepalive” behavior for outgoing connections to a gRPC server. By default, the operating system’s settings are in effect for the socket. If the directive is set to the value “`on`”, the `SO_KEEPALIVE` socket option is turned on for the socket.



### grpc_ssl_certificate

  Syntax:`grpc_ssl_certificate file;`

  Default: —

  Context: `http`, `server`, `location`


Specifies a `file` with the certificate in the PEM format used for authentication to a gRPC SSL server.

Since version 1.21.0, variables can be used in the `file` name.



### grpc_ssl_certificate_key

  Syntax:`grpc_ssl_certificate_key file;`

  Default: —

  Context: `http`, `server`, `location`


Specifies a `file` with the secret key in the PEM format used for authentication to a gRPC SSL server.

The value `engine`:`name`:`id` can be specified instead of the `file`, which loads a secret key with a specified `id` from the OpenSSL engine `name`.

Since version 1.21.0, variables can be used in the `file` name.



### grpc_ssl_ciphers

  Syntax:  `grpc_ssl_ciphers ciphers;`

  Default: `grpc_ssl_ciphers DEFAULT;`

  Context: `http`, `server`, `location`


Specifies the enabled ciphers for requests to a gRPC SSL server. The ciphers are specified in the format understood by the OpenSSL library.

The full list can be viewed using the “`openssl ciphers`” command.



### grpc_ssl_conf_command

  Syntax:`grpc_ssl_conf_command name value;`

  Default: —

  Context: `http`, `server`, `location`


This directive appeared in version 1.19.4.

Sets arbitrary OpenSSL configuration [commands](https://www.openssl.org/docs/man1.1.1/man3/SSL_CONF_cmd.html) when establishing a connection with the gRPC SSL server.

The directive is supported when using OpenSSL 1.0.2 or higher.



Several `grpc_ssl_conf_command` directives can be specified on the same level. These directives are inherited from the previous configuration level if and only if there are no `grpc_ssl_conf_command` directives defined on the current level.



Note that configuring OpenSSL directly might result in unexpected behavior.





### grpc_ssl_crl

  Syntax:  `grpc_ssl_crl file;`

  Default: —

  Context: `http`, `server`, `location`


Specifies a `file` with revoked certificates (CRL) in the PEM format used to [verify]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_ssl_verify">}}) the certificate of the gRPC SSL server.



### grpc_ssl_name

  Syntax:  `grpc_ssl_name name;`

  Default: `grpc_ssl_name host from grpc_pass;`

  Context: `http`, `server`, `location`


Allows overriding the server name used to [verify]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_ssl_verify">}}) the certificate of the gRPC SSL server and to be [passed through SNI]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_ssl_server_name">}}) when establishing a connection with the gRPC SSL server.

By default, the host part from [grpc_pass]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_pass">}}) is used.



### grpc_ssl_password_file

  Syntax:`grpc_ssl_password_file file;`

  Default: —

  Context: `http`, `server`, `location`


Specifies a `file` with passphrases for [secret keys]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_ssl_certificate_key">}}) where each passphrase is specified on a separate line. Passphrases are tried in turn when loading the key.



### grpc_ssl_protocols

  Syntax:`grpc_ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];`

  Default: `grpc_ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;`

  Context: `http`, `server`, `location`


Enables the specified protocols for requests to a gRPC SSL server.



The `TLSv1.3` parameter is used by default since 1.23.4.





### grpc_ssl_server_name

  Syntax:`grpc_ssl_server_name on | off;`

  Default: `grpc_ssl_server_name off;`

  Context: `http`, `server`, `location`


Enables or disables passing of the server name through [TLS Server Name Indication extension](http://en.wikipedia.org/wiki/Server_Name_Indication) (SNI, RFC 6066) when establishing a connection with the gRPC SSL server.



### grpc_ssl_session_reuse

  Syntax:`grpc_ssl_session_reuse on | off;`

  Default: `grpc_ssl_session_reuse on;`

  Context: `http`, `server`, `location`


Determines whether SSL sessions can be reused when working with the gRPC server. If the errors “`SSL3_GET_FINISHED:digest check failed`” appear in the logs, try disabling session reuse.



### grpc_ssl_trusted_certificate

  Syntax:`grpc_ssl_trusted_certificate file;`

  Default: —

  Context: `http`, `server`, `location`


Specifies a `file` with trusted CA certificates in the PEM format used to [verify]({{< ref "/mod_ref/ngx_http_grpc_module#grpc_ssl_verify">}}) the certificate of the gRPC SSL server.



### grpc_ssl_verify

  Syntax:  `grpc_ssl_verify on | off;`

  Default: `grpc_ssl_verify off;`

  Context: `http`, `server`, `location`


Enables or disables verification of the gRPC SSL server certificate.



### grpc_ssl_verify_depth

  Syntax:`grpc_ssl_verify_depth number;`

  Default: `grpc_ssl_verify_depth 1;`

  Context: `http`, `server`, `location`


Sets the verification depth in the gRPC SSL server certificates chain.