+++
title = "ngx_stream_core_module"
date = 2023-08-15T08:22:03+08:00
weight = 710
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_core_module

https://nginx.org/en/docs/stream/ngx_stream_core_module.html



The `ngx_stream_core_module` module is available since version 1.9.0. This module is not built by default, it should be enabled with the `--with-stream` configuration parameter.



## Example Configuration



```
worker_processes auto;

error_log /var/log/nginx/error.log info;

events {
    worker_connections  1024;
}

stream {
    upstream backend {
        hash $remote_addr consistent;

        server backend1.example.com:12345 weight=5;
        server 127.0.0.1:12345            max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;
    }

    upstream dns {
       server 192.168.0.1:53535;
       server dns.example.com:53;
    }

    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

    server {
        listen 127.0.0.1:53 udp reuseport;
        proxy_timeout 20s;
        proxy_pass dns;
    }

    server {
        listen [::1]:12345;
        proxy_pass unix:/tmp/stream.socket;
    }
}
```





## Directives



### listen

  Syntax:`listen address:port [ssl] [udp] [proxy_protocol] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];`

| Default: | —                                                            |
  Context: `server`


Sets the `address` and `port` for the socket on which the server will accept connections. It is possible to specify just the port. The address can also be a hostname, for example:

```
listen 127.0.0.1:12345;
listen *:12345;
listen 12345;     # same as *:12345
listen localhost:12345;
```

IPv6 addresses are specified in square brackets:

```
listen [::1]:12345;
listen [::]:12345;
```

UNIX-domain sockets are specified with the “`unix:`” prefix:

```
listen unix:/var/run/nginx.sock;
```





Port ranges (1.15.10) are specified with the first and last port separated by a hyphen:

```
listen 127.0.0.1:12345-12399;
listen 12345-12399;
```



The `ssl` parameter allows specifying that all connections accepted on this port should work in SSL mode.



The `udp` parameter configures a listening socket for working with datagrams (1.9.13). In order to handle packets from the same address and port in the same session, the [`reuseport`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#reuseport) parameter should also be specified.



The `proxy_protocol` parameter (1.11.4) allows specifying that all connections accepted on this port should use the [PROXY protocol](http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt).

The PROXY protocol version 2 is supported since version 1.13.11.



The `listen` directive can have several additional parameters specific to socket-related system calls.

- `fastopen`=`number`

  enables “[TCP Fast Open](http://en.wikipedia.org/wiki/TCP_Fast_Open)” for the listening socket (1.21.0) and [limits](https://datatracker.ietf.org/doc/html/rfc7413#section-5.1) the maximum length for the queue of connections that have not yet completed the three-way handshake.Do not enable this feature unless the server can handle receiving the [same SYN packet with data](https://datatracker.ietf.org/doc/html/rfc7413#section-6.1) more than once.

- `backlog`=`number`

  sets the `backlog` parameter in the `listen()` call that limits the maximum length for the queue of pending connections (1.9.2). By default, `backlog` is set to -1 on FreeBSD, DragonFly BSD, and macOS, and to 511 on other platforms.

- `rcvbuf`=`size`

  sets the receive buffer size (the `SO_RCVBUF` option) for the listening socket (1.11.13).

- `sndbuf`=`size`

  sets the send buffer size (the `SO_SNDBUF` option) for the listening socket (1.11.13).

- `bind`

  this parameter instructs to make a separate `bind()` call for a given address:port pair. The fact is that if there are several `listen` directives with the same port but different addresses, and one of the `listen` directives listens on all addresses for the given port (`*:port`), nginx will `bind()` only to `*:port`. It should be noted that the `getsockname()` system call will be made in this case to determine the address that accepted the connection. If the `backlog`, `rcvbuf`, `sndbuf`, `ipv6only`, `reuseport`, or `so_keepalive` parameters are used then for a given `address`:`port` pair a separate `bind()` call will always be made.

- `ipv6only`=`on`|`off`

  this parameter determines (via the `IPV6_V6ONLY` socket option) whether an IPv6 socket listening on a wildcard address `[::]` will accept only IPv6 connections or both IPv6 and IPv4 connections. This parameter is turned on by default. It can only be set once on start.

- `reuseport`

  this parameter (1.9.1) instructs to create an individual listening socket for each worker process (using the `SO_REUSEPORT` socket option on Linux 3.9+ and DragonFly BSD, or `SO_REUSEPORT_LB` on FreeBSD 12+), allowing a kernel to distribute incoming connections between worker processes. This currently works only on Linux 3.9+, DragonFly BSD, and FreeBSD 12+ (1.15.1).Inappropriate use of this option may have its security [implications](http://man7.org/linux/man-pages/man7/socket.7.html).

- `so_keepalive`=`on`|`off`|[`keepidle`]:[`keepintvl`]:[`keepcnt`]

  this parameter configures the “TCP keepalive” behavior for the listening socket. If this parameter is omitted then the operating system’s settings will be in effect for the socket. If it is set to the value “`on`”, the `SO_KEEPALIVE` option is turned on for the socket. If it is set to the value “`off`”, the `SO_KEEPALIVE` option is turned off for the socket. Some operating systems support setting of TCP keepalive parameters on a per-socket basis using the `TCP_KEEPIDLE`, `TCP_KEEPINTVL`, and `TCP_KEEPCNT` socket options. On such systems (currently, Linux 2.4+, NetBSD 5+, and FreeBSD 9.0-STABLE), they can be configured using the `keepidle`, `keepintvl`, and `keepcnt` parameters. One or two parameters may be omitted, in which case the system default setting for the corresponding socket option will be in effect. For example,`so_keepalive=30m::10`will set the idle timeout (`TCP_KEEPIDLE`) to 30 minutes, leave the probe interval (`TCP_KEEPINTVL`) at its system default, and set the probes count (`TCP_KEEPCNT`) to 10 probes.



Different servers must listen on different `address`:`port` pairs.



### preread_buffer_size

  Syntax:`preread_buffer_size size;`

  Default: `preread_buffer_size 16k;`

  Context: `stream`, `server`


This directive appeared in version 1.11.5.

Specifies a `size` of the [preread](https://nginx.org/en/docs/stream/stream_processing.html#preread_phase) buffer.



### preread_timeout

  Syntax:`preread_timeout timeout;`

  Default: `preread_timeout 30s;`

  Context: `stream`, `server`


This directive appeared in version 1.11.5.

Specifies a `timeout` of the [preread](https://nginx.org/en/docs/stream/stream_processing.html#preread_phase) phase.



### proxy_protocol_timeout

  Syntax:`proxy_protocol_timeout timeout;`

  Default: `proxy_protocol_timeout 30s;`

  Context: `stream`, `server`


This directive appeared in version 1.11.4.

Specifies a `timeout` for reading the PROXY protocol header to complete. If no entire header is transmitted within this time, the connection is closed.



### resolver

  Syntax:`resolver address ... [valid=time] [ipv4=on|off] [ipv6=on|off] [status_zone=zone];`

| Default: | —                                                            |
  Context: `stream`, `server`


This directive appeared in version 1.11.3.

Configures name servers used to resolve names of upstream servers into addresses, for example:

```
resolver 127.0.0.1 [::1]:5353;
```

The address can be specified as a domain name or IP address, with an optional port. If port is not specified, the port 53 is used. Name servers are queried in a round-robin fashion.



By default, nginx will look up both IPv4 and IPv6 addresses while resolving. If looking up of IPv4 or IPv6 addresses is not desired, the `ipv4=off` (1.23.1) or the `ipv6=off` parameter can be specified.



By default, nginx caches answers using the TTL value of a response. The optional `valid` parameter allows overriding it:

```
resolver 127.0.0.1 [::1]:5353 valid=30s;
```



To prevent DNS spoofing, it is recommended configuring DNS servers in a properly secured trusted local network.





The optional `status_zone` parameter (1.17.1) enables [collection](https://nginx.org/en/docs/http/ngx_http_api_module.html#resolvers_) of DNS server statistics of requests and responses in the specified `zone`. The parameter is available as part of our [commercial subscription](http://nginx.com/products/).



Before version 1.11.3, this directive was available as part of our [commercial subscription](http://nginx.com/products/).





### resolver_timeout

  Syntax:`resolver_timeout time;`

  Default: `resolver_timeout 30s;`

  Context: `stream`, `server`


This directive appeared in version 1.11.3.

Sets a timeout for name resolution, for example:

```
resolver_timeout 5s;
```



Before version 1.11.3, this directive was available as part of our [commercial subscription](http://nginx.com/products/).





### server

  Syntax:`server { ... }`

| Default: | —                |
  Context: `stream`


Sets the configuration for a server.



### stream

  Syntax:`stream { ... }`

| Default: | —                |
  Context: `main`


Provides the configuration file context in which the stream server directives are specified.



### tcp_nodelay

  Syntax:`tcp_nodelay on | off;`

  Default: `tcp_nodelay on;`

  Context: `stream`, `server`


This directive appeared in version 1.9.4.

Enables or disables the use of the `TCP_NODELAY` option. The option is enabled for both client and proxied server connections.



### variables_hash_bucket_size

  Syntax:`variables_hash_bucket_size size;`

  Default: `variables_hash_bucket_size 64;`

  Context: `stream`


This directive appeared in version 1.11.2.

Sets the bucket size for the variables hash table. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).



### variables_hash_max_size

  Syntax:`variables_hash_max_size size;`

  Default: `variables_hash_max_size 1024;`

  Context: `stream`


This directive appeared in version 1.11.2.

Sets the maximum `size` of the variables hash table. The details of setting up hash tables are provided in a separate [document](https://nginx.org/en/docs/hash.html).



## Embedded Variables

The `ngx_stream_core_module` module supports variables since 1.11.2.

- `$binary_remote_addr`

  client address in a binary form, value’s length is always 4 bytes for IPv4 addresses or 16 bytes for IPv6 addresses

- `$bytes_received`

  number of bytes received from a client (1.11.4)

- `$bytes_sent`

  number of bytes sent to a client

- `$connection`

  connection serial number

- `$hostname`

  host name

- `$msec`

  current time in seconds with the milliseconds resolution

- `$nginx_version`

  nginx version

- `$pid`

  PID of the worker process

- `$protocol`

  protocol used to communicate with the client: `TCP` or `UDP` (1.11.4)

- `$proxy_protocol_addr`

  client address from the PROXY protocol header (1.11.4)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive.

- `$proxy_protocol_port`

  client port from the PROXY protocol header (1.11.4)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive.

- `$proxy_protocol_server_addr`

  server address from the PROXY protocol header (1.17.6)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive.

- `$proxy_protocol_server_port`

  server port from the PROXY protocol header (1.17.6)The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive.

- `$proxy_protocol_tlv_``name`

  TLV from the PROXY Protocol header (1.23.2). The `name` can be a TLV type name or its numeric value. In the latter case, the value is hexadecimal and should be prefixed with `0x`:`$proxy_protocol_tlv_alpn $proxy_protocol_tlv_0x01 `SSL TLVs can also be accessed by TLV type name or its numeric value, both prefixed by `ssl_`:`$proxy_protocol_tlv_ssl_version $proxy_protocol_tlv_ssl_0x21 `The following TLV type names are supported:`alpn` (`0x01`) - upper layer protocol used over the connection`authority` (`0x02`) - host name value passed by the client`unique_id` (`0x05`) - unique connection id`netns` (`0x30`) - name of the namespace`ssl` (`0x20`) - binary SSL TLV structureThe following SSL TLV type names are supported:`ssl_version` (`0x21`) - SSL version used in client connection`ssl_cn` (`0x22`) - SSL certificate Common Name`ssl_cipher` (`0x23`) - name of the used cipher`ssl_sig_alg` (`0x24`) - algorithm used to sign the certificate`ssl_key_alg` (`0x25`) - public-key algorithmAlso, the following special SSL TLV type name is supported:`ssl_verify` - client SSL certificate verification result, zero if the client presented a certificate and it was successfully verified, and non-zero otherwiseThe PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive.

- `$remote_addr`

  client address

- `$remote_port`

  client port

- `$server_addr`

  an address of the server which accepted a connectionComputing a value of this variable usually requires one system call. To avoid a system call, the [listen](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directives must specify addresses and use the `bind` parameter.

- `$server_port`

  port of the server which accepted a connection

- `$session_time`

  session duration in seconds with a milliseconds resolution (1.11.4);

- `$status`

  session status (1.11.4), can be one of the following:`200`session completed successfully`400`client data could not be parsed, for example, the [PROXY protocol](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#proxy_protocol) header`403`access forbidden, for example, when access is limited for [certain client addresses](https://nginx.org/en/docs/stream/ngx_stream_access_module.html)`500`internal server error`502`bad gateway, for example, if an upstream server could not be selected or reached.`503`service unavailable, for example, when access is limited by the [number of connections](https://nginx.org/en/docs/stream/ngx_stream_limit_conn_module.html)

- `$time_iso8601`

  local time in the ISO 8601 standard format

- `$time_local`

  local time in the Common Log Format