+++
title = "ngx_stream_js_module"
date = 2023-08-15T08:22:39+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_js_module

https://nginx.org/en/docs/stream/ngx_stream_js_module.html



The `ngx_stream_js_module` module is used to implement handlers in [njs](https://nginx.org/en/docs/njs/index.html) — a subset of the JavaScript language.

Download and install instructions are available [here](https://nginx.org/en/docs/njs/install.html).



## Example Configuration

The example works since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0).

> ```
> stream {
>     js_import stream.js;
> 
>     js_set $bar stream.bar;
>     js_set $req_line stream.req_line;
> 
>     server {
>         listen 12345;
> 
>         js_preread stream.preread;
>         return     $req_line;
>     }
> 
>     server {
>         listen 12346;
> 
>         js_access  stream.access;
>         proxy_pass 127.0.0.1:8000;
>         js_filter  stream.header_inject;
>     }
> }
> 
> http {
>     server {
>         listen 8000;
>         location / {
>             return 200 $http_foo\n;
>         }
>     }
> }
> ```



The `stream.js` file:

> ```
> var line = '';
> 
> function bar(s) {
>     var v = s.variables;
>     s.log("hello from bar() handler!");
>     return "bar-var" + v.remote_port + "; pid=" + v.pid;
> }
> 
> function preread(s) {
>     s.on('upload', function (data, flags) {
>         var n = data.indexOf('\n');
>         if (n != -1) {
>             line = data.substr(0, n);
>             s.done();
>         }
>     });
> }
> 
> function req_line(s) {
>     return line;
> }
> 
> // Read HTTP request line.
> // Collect bytes in 'req' until
> // request line is read.
> // Injects HTTP header into a client's request
> 
> var my_header =  'Foo: foo';
> function header_inject(s) {
>     var req = '';
>     s.on('upload', function(data, flags) {
>         req += data;
>         var n = req.search('\n');
>         if (n != -1) {
>             var rest = req.substr(n + 1);
>             req = req.substr(0, n + 1);
>             s.send(req + my_header + '\r\n' + rest, flags);
>             s.off('upload');
>         }
>     });
> }
> 
> function access(s) {
>     if (s.remoteAddress.match('^192.*')) {
>         s.deny();
>         return;
>     }
> 
>     s.allow();
> }
> 
> export default {bar, preread, req_line, header_inject, access};
> ```





## Directives



### js_access

| Syntax:  | `js_access function | module.function;` |
| :------- | --------------------------------------- |
| Default: | —                                       |
| Context: | `stream`, `server`                      |

Sets an njs function which will be called at the [access](https://nginx.org/en/docs/stream/stream_processing.html#access_phase) phase. Since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0), a module function can be referenced.

The function is called once at the moment when the stream session reaches the [access](https://nginx.org/en/docs/stream/stream_processing.html#access_phase) phase for the first time. The function is called with the following arguments:

- `s`

  the [Stream Session](https://nginx.org/en/docs/njs/reference.html#stream) object



At this phase, it is possible to perform initialization or register a callback with the [`s.on()`](https://nginx.org/en/docs/njs/reference.html#s_on) method for each incoming data chunk until one of the following methods are called: [`s.allow()`](https://nginx.org/en/docs/njs/reference.html#s_allow), [`s.decline()`](https://nginx.org/en/docs/njs/reference.html#s_decline), [`s.done()`](https://nginx.org/en/docs/njs/reference.html#s_done). As soon as one of these methods is called, the stream session processing switches to the [next phase](https://nginx.org/en/docs/stream/stream_processing.html) and all current [`s.on()`](https://nginx.org/en/docs/njs/reference.html#s_on) callbacks are dropped.



### js_fetch_buffer_size

| Syntax:  | `js_fetch_buffer_size size;` |
| :------- | ---------------------------- |
| Default: | `js_fetch_buffer_size 16k;`  |
| Context: | `stream`, `server`           |

This directive appeared in version 0.7.4.

Sets the `size` of the buffer used for reading and writing with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_ciphers

| Syntax:  | `js_fetch_ciphers ciphers;`          |
| :------- | ------------------------------------ |
| Default: | `js_fetch_ciphers HIGH:!aNULL:!MD5;` |
| Context: | `stream`, `server`                   |

This directive appeared in version 0.7.0.

Specifies the enabled ciphers for HTTPS connections with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch). The ciphers are specified in the format understood by the OpenSSL library.

The full list can be viewed using the “`openssl ciphers`” command.



### js_fetch_max_response_buffer_size

| Syntax:  | `js_fetch_max_response_buffer_size size;` |
| :------- | ----------------------------------------- |
| Default: | `js_fetch_max_response_buffer_size 1m;`   |
| Context: | `stream`, `server`                        |

This directive appeared in version 0.7.4.

Sets the maximum `size` of the response received with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_protocols

| Syntax:  | `js_fetch_protocols [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];` |
| :------- | ----------------------------------------------------------- |
| Default: | `js_fetch_protocols TLSv1 TLSv1.1 TLSv1.2;`                 |
| Context: | `stream`, `server`                                          |

This directive appeared in version 0.7.0.

Enables the specified protocols for HTTPS connections with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_timeout

| Syntax:  | `js_fetch_timeout time;` |
| :------- | ------------------------ |
| Default: | `js_fetch_timeout 60s;`  |
| Context: | `stream`, `server`       |

This directive appeared in version 0.7.4.

Defines a timeout for reading and writing for [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch). The timeout is set only between two successive read/write operations, not for the whole response. If no data is transmitted within this time, the connection is closed.



### js_fetch_trusted_certificate

| Syntax:  | `js_fetch_trusted_certificate file;` |
| :------- | ------------------------------------ |
| Default: | —                                    |
| Context: | `stream`, `server`                   |

This directive appeared in version 0.7.0.

Specifies a `file` with trusted CA certificates in the PEM format used to [verify](https://nginx.org/en/docs/njs/reference.html#fetch_verify) the HTTPS certificate with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_verify

| Syntax:  | `js_fetch_verify on | off;` |
| :------- | --------------------------- |
| Default: | `js_fetch_verify on;`       |
| Context: | `stream`, `server`          |

This directive appeared in version 0.7.4.

Enables or disables verification of the HTTPS server certificate with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_verify_depth

| Syntax:  | `js_fetch_verify_depth number;` |
| :------- | ------------------------------- |
| Default: | `js_fetch_verify_depth 100;`    |
| Context: | `stream`, `server`              |

This directive appeared in version 0.7.0.

Sets the verification depth in the HTTPS server certificates chain with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_filter

| Syntax:  | `js_filter function | module.function;` |
| :------- | --------------------------------------- |
| Default: | —                                       |
| Context: | `stream`, `server`                      |

Sets a data filter. Since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0), a module function can be referenced. The filter function is called once at the moment when the stream session reaches the [content](https://nginx.org/en/docs/stream/stream_processing.html#content_phase) phase.

The filter function is called with the following arguments:

- `s`

  the [Stream Session](https://nginx.org/en/docs/njs/reference.html#stream) object



At this phase, it is possible to perform initialization or register a callback with the [`s.on()`](https://nginx.org/en/docs/njs/reference.html#s_on) method for each incoming data chunk. The [`s.off()`](https://nginx.org/en/docs/njs/reference.html#s_off) method may be used to unregister a callback and stop filtering.



> As the `js_filter` handler returns its result immediately, it supports only synchronous operations. Thus, asynchronous operations such as [`ngx.fetch()`](https://nginx.org/en/docs/njs/reference.html#ngx_fetch) or [`setTimeout()`](https://nginx.org/en/docs/njs/reference.html#settimeout) are not supported.





### js_import

| Syntax:  | `js_import module.js | export_name from module.js;` |
| :------- | --------------------------------------------------- |
| Default: | —                                                   |
| Context: | `stream`, `server`                                  |

This directive appeared in version 0.4.0.

Imports a module that implements location and variable handlers in njs. The `export_name` is used as a namespace to access module functions. If the `export_name` is not specified, the module name will be used as a namespace.

> ```
> js_import stream.js;
> ```

Here, the module name `stream` is used as a namespace while accessing exports. If the imported module exports `foo()`, `stream.foo` is used to refer to it.

Several `js_import` directives can be specified.



> The directive can be specified on the `server` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_include

| Syntax:  | `js_include file;` |
| :------- | ------------------ |
| Default: | —                  |
| Context: | `stream`           |

Specifies a file that implements server and variable handlers in njs:

> ```
> nginx.conf:
> js_include stream.js;
> js_set     $js_addr address;
> server {
>     listen 127.0.0.1:12345;
>     return $js_addr;
> }
> 
> stream.js:
> function address(s) {
>     return s.remoteAddress;
> }
> ```



The directive was made obsolete in version [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0) and was removed in version [0.7.1](https://nginx.org/en/docs/njs/changes.html#njs0.7.1). The [js_import](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_import) directive should be used instead.



### js_path

| Syntax:  | `js_path path;`    |
| :------- | ------------------ |
| Default: | —                  |
| Context: | `stream`, `server` |

This directive appeared in version 0.3.0.

Sets an additional path for njs modules.



> The directive can be specified on the `server` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_preload_object

| Syntax:  | `js_preload_object name.json | name from file.json;` |
| :------- | ---------------------------------------------------- |
| Default: | —                                                    |
| Context: | `stream`, `server`                                   |

This directive appeared in version 0.7.8.

Preloads an immutable object at configure time. The `name` is used a name of the global variable though which the object is available in njs code. If the `name` is not specified, the file name will be used instead.

> ```
> js_preload_object map.json;
> ```

Here, the `map` is used as a name while accessing the preloaded object.

Several `js_preload_object` directives can be specified.



### js_preread

| Syntax:  | `js_preread function | module.function;` |
| :------- | ---------------------------------------- |
| Default: | —                                        |
| Context: | `stream`, `server`                       |

Sets an njs function which will be called at the [preread](https://nginx.org/en/docs/stream/stream_processing.html#preread_phase) phase. Since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0), a module function can be referenced.

The function is called once at the moment when the stream session reaches the [preread](https://nginx.org/en/docs/stream/stream_processing.html#preread_phase) phase for the first time. The function is called with the following arguments:

- `s`

  the [Stream Session](https://nginx.org/en/docs/njs/reference.html#stream) object



At this phase, it is possible to perform initialization or register a callback with the [`s.on()`](https://nginx.org/en/docs/njs/reference.html#s_on) method for each incoming data chunk until one of the following methods are called: [`s.allow()`](https://nginx.org/en/docs/njs/reference.html#s_allow), [`s.decline()`](https://nginx.org/en/docs/njs/reference.html#s_decline), [`s.done()`](https://nginx.org/en/docs/njs/reference.html#s_done). When one of these methods is called, the stream session switches to the [next phase](https://nginx.org/en/docs/stream/stream_processing.html) and all current [`s.on()`](https://nginx.org/en/docs/njs/reference.html#s_on) callbacks are dropped.



> As the `js_preread` handler returns its result immediately, it supports only synchronous callbacks. Thus, asynchronous callbacks such as [`ngx.fetch()`](https://nginx.org/en/docs/njs/reference.html#ngx_fetch) or [`setTimeout()`](https://nginx.org/en/docs/njs/reference.html#settimeout) are not supported. Nevertheless, asynchronous operations are supported in [`s.on()`](https://nginx.org/en/docs/njs/reference.html#s_on) callbacks in the [preread](https://nginx.org/en/docs/stream/stream_processing.html#preread_phase) phase. See [this example](https://github.com/nginx/njs-examples#authorizing-connections-using-ngx-fetch-as-auth-request-stream-auth-request) for more information.





### js_set

| Syntax:  | `js_set $variable function | module.function;` |
| :------- | ---------------------------------------------- |
| Default: | —                                              |
| Context: | `stream`, `server`                             |

Sets an njs `function` for the specified `variable`. Since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0), a module function can be referenced.

The function is called when the variable is referenced for the first time for a given request. The exact moment depends on a [phase](https://nginx.org/en/docs/stream/stream_processing.html) at which the variable is referenced. This can be used to perform some logic not related to variable evaluation. For example, if the variable is referenced only in the [log_format](https://nginx.org/en/docs/stream/ngx_stream_log_module.html#log_format) directive, its handler will not be executed until the log phase. This handler can be used to do some cleanup right before the request is freed.



> As the `js_set` handler returns its result immediately, it supports only synchronous callbacks. Thus, asynchronous callbacks such as [ngx.fetch()](https://nginx.org/en/docs/njs/reference.html#ngx_fetch) or [setTimeout()](https://nginx.org/en/docs/njs/reference.html#settimeout) are not supported.





> The directive can be specified on the `server` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_shared_dict_zone

| Syntax:  | `js_shared_dict_zone zone=name:size [timeout=time] [type=string|number] [evict];` |
| :------- | ------------------------------------------------------------ |
| Default: | —                                                            |
| Context: | `stream`                                                     |

This directive appeared in version 0.8.0.

Sets the `name` and `size` of the shared memory zone that keeps the key-value dictionary shared between worker processes.

By default the shared dictionary uses a string as a key and a value. The optional `type` parameter allows redefining the value type to number.

The optional `timeout` parameter sets the time after which all shared dictionary entries are removed from the zone.

The optional `evict` parameter removes the oldest key-value pair when the zone storage is exhausted.

Examples:

> ```
> example.conf:
>     # Creates a 1Mb dictionary with string values,
>     # removes key-value pairs after 60 seconds of inactivity:
>     js_shared_dict_zone zone=foo:1M timeout=60s;
> 
>     # Creates a 512Kb dictionary with string values,
>     # forcibly removes oldest key-value pairs when the zone is exhausted:
>     js_shared_dict_zone zone=bar:512K timeout=30s evict;
> 
>     # Creates a 32Kb permanent dictionary with number values:
>     js_shared_dict_zone zone=num:32k type=number;
> 
> example.js:
>     function get(r) {
>         r.return(200, ngx.shared.foo.get(r.args.key));
>     }
> 
>     function set(r) {
>         r.return(200, ngx.shared.foo.set(r.args.key, r.args.value));
>     }
> 
>     function delete(r) {
>         r.return(200, ngx.shared.bar.delete(r.args.key));
>     }
> 
>     function increment(r) {
>         r.return(200, ngx.shared.num.incr(r.args.key, 2));
>     }
> ```





### js_var

| Syntax:  | `js_var $variable [value];` |
| :------- | --------------------------- |
| Default: | —                           |
| Context: | `stream`, `server`          |

This directive appeared in version 0.5.3.

Declares a [writable](https://nginx.org/en/docs/njs/reference.html#r_variables) variable. The value can contain text, variables, and their combination.



> The directive can be specified on the `server` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





## Session Object Properties

Each stream njs handler receives one argument, a stream session [object](https://nginx.org/en/docs/njs/reference.html#stream).