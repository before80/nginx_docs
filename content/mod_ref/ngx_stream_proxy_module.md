+++
title = "ngx_stream_proxy_module"
date = 2023-08-15T08:23:41+08:00
weight = 810
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_proxy_module

https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html



The `ngx_stream_proxy_module` module (1.9.0) allows proxying data streams over TCP, UDP (1.9.13), and UNIX-domain sockets.



## Example Configuration



```
server {
    listen 127.0.0.1:12345;
    proxy_pass 127.0.0.1:8080;
}

server {
    listen 12345;
    proxy_connect_timeout 1s;
    proxy_timeout 1m;
    proxy_pass example.com:12345;
}

server {
    listen 53 udp reuseport;
    proxy_timeout 20s;
    proxy_pass dns.example.com:53;
}

server {
    listen [::1]:12345;
    proxy_pass unix:/tmp/stream.socket;
}
```





## Directives



### proxy_bind

  Syntax:`proxy_bind address [transparent] | off;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.9.2.

Makes outgoing connections to a proxied server originate from the specified local IP `address`. Parameter value can contain variables (1.11.2). The special value `off` cancels the effect of the `proxy_bind` directive inherited from the previous configuration level, which allows the system to auto-assign the local IP address.



The `transparent` parameter (1.11.0) allows outgoing connections to a proxied server originate from a non-local IP address, for example, from a real IP address of a client:

```
proxy_bind $remote_addr transparent;
```

In order for this parameter to work, it is usually necessary to run nginx worker processes with the [superuser]({{< ref "/mod_ref/ngx_core_module#user">}}) privileges. On Linux it is not required (1.13.8) as if the `transparent` parameter is specified, worker processes inherit the `CAP_NET_RAW` capability from the master process. It is also necessary to configure kernel routing table to intercept network traffic from the proxied server.



### proxy_buffer_size

  Syntax:`proxy_buffer_size size;`

  Default: `proxy_buffer_size 16k;`

  Context: `stream`, `server`


This directive appeared in version 1.9.4.

Sets the `size` of the buffer used for reading data from the proxied server. Also sets the `size` of the buffer used for reading data from the client.



### proxy_connect_timeout

  Syntax:`proxy_connect_timeout time;`

  Default: `proxy_connect_timeout 60s;`

  Context: `stream`, `server`


Defines a timeout for establishing a connection with a proxied server.



### proxy_download_rate

  Syntax:`proxy_download_rate rate;`

  Default: `proxy_download_rate 0;`

  Context: `stream`, `server`


This directive appeared in version 1.9.3.

Limits the speed of reading the data from the proxied server. The `rate` is specified in bytes per second. The zero value disables rate limiting. The limit is set per a connection, so if nginx simultaneously opens two connections to the proxied server, the overall rate will be twice as much as the specified limit.

Parameter value can contain variables (1.17.0). It may be useful in cases where rate should be limited depending on a certain condition:

```
map $slow $rate {
    1     4k;
    2     8k;
}

proxy_download_rate $rate;
```





### proxy_half_close

  Syntax:`proxy_half_close on | off;`

  Default: `proxy_half_close off;`

  Context: `stream`, `server`


This directive appeared in version 1.21.4.

Enables or disables closing each direction of a TCP connection independently (“TCP half-close”). If enabled, proxying over TCP will be kept until both sides close the connection.



### proxy_next_upstream

  Syntax:`proxy_next_upstream on | off;`

  Default: `proxy_next_upstream on;`

  Context: `stream`, `server`


When a connection to the proxied server cannot be established, determines whether a client connection will be passed to the next server.

Passing a connection to the next server can be limited by [the number of tries](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_next_upstream_tries) and by [time](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_next_upstream_timeout).



### proxy_next_upstream_timeout

  Syntax:`proxy_next_upstream_timeout time;`

  Default: `proxy_next_upstream_timeout 0;`

  Context: `stream`, `server`


Limits the time allowed to pass a connection to the [next server](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_next_upstream). The `0` value turns off this limitation.



### proxy_next_upstream_tries

  Syntax:`proxy_next_upstream_tries number;`

  Default: `proxy_next_upstream_tries 0;`

  Context: `stream`, `server`


Limits the number of possible tries for passing a connection to the [next server](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_next_upstream). The `0` value turns off this limitation.



### proxy_pass

  Syntax:`proxy_pass address;`

  Default: —

  Context: `server`


Sets the address of a proxied server. The address can be specified as a domain name or IP address, and a port:

```
proxy_pass localhost:12345;
```

or as a UNIX-domain socket path:

```
proxy_pass unix:/tmp/stream.socket;
```



If a domain name resolves to several addresses, all of them will be used in a round-robin fashion. In addition, an address can be specified as a [server group](https://nginx.org/en/docs/stream/ngx_stream_upstream_module.html).

The address can also be specified using variables (1.11.3):

```
proxy_pass $upstream;
```

In this case, the server name is searched among the described [server groups](https://nginx.org/en/docs/stream/ngx_stream_upstream_module.html), and, if not found, is determined using a [resolver](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#resolver).



### proxy_protocol

  Syntax:`proxy_protocol on | off;`

  Default: `proxy_protocol off;`

  Context: `stream`, `server`


This directive appeared in version 1.9.2.

Enables the [PROXY protocol](http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt) for connections to a proxied server.



### proxy_requests

  Syntax:`proxy_requests number;`

  Default: `proxy_requests 0;`

  Context: `stream`, `server`


This directive appeared in version 1.15.7.

Sets the number of client datagrams at which binding between a client and existing UDP stream session is dropped. After receiving the specified number of datagrams, next datagram from the same client starts a new session. The session terminates when all client datagrams are transmitted to a proxied server and the expected number of [responses](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_responses) is received, or when it reaches a [timeout](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_timeout).



### proxy_responses

  Syntax:`proxy_responses number;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.9.13.

Sets the number of datagrams expected from the proxied server in response to a client datagram if the [UDP](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#udp) protocol is used. The number serves as a hint for session termination. By default, the number of datagrams is not limited.

If zero value is specified, no response is expected. However, if a response is received and the session is still not finished, the response will be handled.



### proxy_session_drop

  Syntax:`proxy_session_drop on | off;`

  Default: `proxy_session_drop off;`

  Context: `stream`, `server`


This directive appeared in version 1.15.8.

Enables terminating all sessions to a proxied server after it was removed from the group or marked as permanently unavailable. This can occur because of [re-resolve](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#resolver) or with the API [`DELETE`]({{< ref "/mod_ref/ngx_http_api_module#deleteStreamUpstreamServer">}}) command. A server can be marked as permanently unavailable if it is considered [unhealthy](https://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html#health_check) or with the API [`PATCH`]({{< ref "/mod_ref/ngx_http_api_module#patchStreamUpstreamServer">}}) command. Each session is terminated when the next read or write event is processed for the client or proxied server.



This directive is available as part of our [commercial subscription](http://nginx.com/products/).





### proxy_socket_keepalive

  Syntax:`proxy_socket_keepalive on | off;`

  Default: `proxy_socket_keepalive off;`

  Context: `stream`, `server`


This directive appeared in version 1.15.6.

Configures the “TCP keepalive” behavior for outgoing connections to a proxied server. By default, the operating system’s settings are in effect for the socket. If the directive is set to the value “`on`”, the `SO_KEEPALIVE` socket option is turned on for the socket.



### proxy_ssl

  Syntax:`proxy_ssl on | off;`

  Default: `proxy_ssl off;`

  Context: `stream`, `server`


Enables the SSL/TLS protocol for connections to a proxied server.



### proxy_ssl_certificate

  Syntax:`proxy_ssl_certificate file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with the certificate in the PEM format used for authentication to a proxied server.

Since version 1.21.0, variables can be used in the `file` name.



### proxy_ssl_certificate_key

  Syntax:`proxy_ssl_certificate_key file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with the secret key in the PEM format used for authentication to a proxied server.

Since version 1.21.0, variables can be used in the `file` name.



### proxy_ssl_ciphers

  Syntax:`proxy_ssl_ciphers ciphers;`

  Default: `proxy_ssl_ciphers DEFAULT;`

  Context: `stream`, `server`


Specifies the enabled ciphers for connections to a proxied server. The ciphers are specified in the format understood by the OpenSSL library.

The full list can be viewed using the “`openssl ciphers`” command.



### proxy_ssl_conf_command

  Syntax:`proxy_ssl_conf_command name value;`

  Default: —

  Context: `stream`, `server`


This directive appeared in version 1.19.4.

Sets arbitrary OpenSSL configuration [commands](https://www.openssl.org/docs/man1.1.1/man3/SSL_CONF_cmd.html) when establishing a connection with the proxied server.

The directive is supported when using OpenSSL 1.0.2 or higher.



Several `proxy_ssl_conf_command` directives can be specified on the same level. These directives are inherited from the previous configuration level if and only if there are no `proxy_ssl_conf_command` directives defined on the current level.



Note that configuring OpenSSL directly might result in unexpected behavior.





### proxy_ssl_crl

  Syntax:`proxy_ssl_crl file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with revoked certificates (CRL) in the PEM format used to [verify](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_ssl_verify) the certificate of the proxied server.



### proxy_ssl_name

  Syntax:  `proxy_ssl_name name;`

  Default: `proxy_ssl_name host from proxy_pass;`

  Context: `stream`, `server`


Allows overriding the server name used to [verify](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_ssl_verify) the certificate of the proxied server and to be [passed through SNI](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_ssl_server_name) when establishing a connection with the proxied server. The server name can also be specified using variables (1.11.3).

By default, the host part of the [proxy_pass](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_pass) address is used.



### proxy_ssl_password_file

  Syntax:`proxy_ssl_password_file file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with passphrases for [secret keys](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_ssl_certificate_key) where each passphrase is specified on a separate line. Passphrases are tried in turn when loading the key.



### proxy_ssl_protocols

  Syntax:`proxy_ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];`

  Default: `proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;`

  Context: `stream`, `server`


Enables the specified protocols for connections to a proxied server.



The `TLSv1.3` parameter is used by default since 1.23.4.





### proxy_ssl_server_name

  Syntax:`proxy_ssl_server_name on | off;`

  Default: `proxy_ssl_server_name off;`

  Context: `stream`, `server`


Enables or disables passing of the server name through [TLS Server Name Indication extension](http://en.wikipedia.org/wiki/Server_Name_Indication) (SNI, RFC 6066) when establishing a connection with the proxied server.



### proxy_ssl_session_reuse

  Syntax:`proxy_ssl_session_reuse on | off;`

  Default: `proxy_ssl_session_reuse on;`

  Context: `stream`, `server`


Determines whether SSL sessions can be reused when working with the proxied server. If the errors “`SSL3_GET_FINISHED:digest check failed`” appear in the logs, try disabling session reuse.



### proxy_ssl_trusted_certificate

  Syntax:`proxy_ssl_trusted_certificate file;`

  Default: —

  Context: `stream`, `server`


Specifies a `file` with trusted CA certificates in the PEM format used to [verify](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_ssl_verify) the certificate of the proxied server.



### proxy_ssl_verify

  Syntax:`proxy_ssl_verify on | off;`

  Default: `proxy_ssl_verify off;`

  Context: `stream`, `server`


Enables or disables verification of the proxied server certificate.



### proxy_ssl_verify_depth

  Syntax:`proxy_ssl_verify_depth number;`

  Default: `proxy_ssl_verify_depth 1;`

  Context: `stream`, `server`


Sets the verification depth in the proxied server certificates chain.



### proxy_timeout

  Syntax:`proxy_timeout timeout;`

  Default: `proxy_timeout 10m;`

  Context: `stream`, `server`


Sets the `timeout` between two successive read or write operations on client or proxied server connections. If no data is transmitted within this time, the connection is closed.



### proxy_upload_rate

  Syntax:`proxy_upload_rate rate;`

  Default: `proxy_upload_rate 0;`

  Context: `stream`, `server`


This directive appeared in version 1.9.3.

Limits the speed of reading the data from the client. The `rate` is specified in bytes per second. The zero value disables rate limiting. The limit is set per a connection, so if the client simultaneously opens two connections, the overall rate will be twice as much as the specified limit.

Parameter value can contain variables (1.17.0). It may be useful in cases where rate should be limited depending on a certain condition:

```
map $slow $rate {
    1     4k;
    2     8k;
}

proxy_upload_rate $rate;
```