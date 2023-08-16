+++
title = "ngx_http_js_module"
date = 2023-08-15T08:15:40+08:00
weight = 270
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_js_module

https://nginx.org/en/docs/http/ngx_http_js_module.html



The `ngx_http_js_module` module is used to implement location and variable handlers in [njs](https://nginx.org/en/docs/njs/index.html) — a subset of the JavaScript language.

Download and install instructions are available [here](https://nginx.org/en/docs/njs/install.html).



## Example Configuration

The example works since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0).

```
http {
    js_import http.js;

    js_set $foo     http.foo;
    js_set $summary http.summary;
    js_set $hash    http.hash;

    resolver 10.0.0.1;

    server {
        listen 8000;

        location / {
            add_header X-Foo $foo;
            js_content http.baz;
        }

        location = /summary {
            return 200 $summary;
        }

        location = /hello {
            js_content http.hello;
        }

        # since 0.7.0
        location = /fetch {
            js_content                   http.fetch;
            js_fetch_trusted_certificate /path/to/ISRG_Root_X1.pem;
        }

        # since 0.7.0
        location = /crypto {
            add_header Hash $hash;
            return     200;
        }
    }
}
```



The `http.js` file:

```
function foo(r) {
    r.log("hello from foo() handler");
    return "foo";
}

function summary(r) {
    var a, s, h;

    s = "JS summary\n\n";

    s += "Method: " + r.method + "\n";
    s += "HTTP version: " + r.httpVersion + "\n";
    s += "Host: " + r.headersIn.host + "\n";
    s += "Remote Address: " + r.remoteAddress + "\n";
    s += "URI: " + r.uri + "\n";

    s += "Headers:\n";
    for (h in r.headersIn) {
        s += "  header '" + h + "' is '" + r.headersIn[h] + "'\n";
    }

    s += "Args:\n";
    for (a in r.args) {
        s += "  arg '" + a + "' is '" + r.args[a] + "'\n";
    }

    return s;
}

function baz(r) {
    r.status = 200;
    r.headersOut.foo = 1234;
    r.headersOut['Content-Type'] = "text/plain; charset=utf-8";
    r.headersOut['Content-Length'] = 15;
    r.sendHeader();
    r.send("nginx");
    r.send("java");
    r.send("script");

    r.finish();
}

function hello(r) {
    r.return(200, "Hello world!");
}

// since 0.7.0
async function fetch(r) {
    let results = await Promise.all([ngx.fetch('https://nginx.org/'),
                                     ngx.fetch('https://nginx.org/en/')]);

    r.return(200, JSON.stringify(results, undefined, 4));
}

// since 0.7.0
async function hash(r) {
    let hash = await crypto.subtle.digest('SHA-512', r.headersIn.host);
    r.setReturnValue(Buffer.from(hash).toString('hex'));
}

export default {foo, summary, baz, hello, fetch, hash};
```





## Directives



### js_body_filter

  Syntax:`js_body_filter function | module.function [buffer_type=string | buffer];`

  Default: —

  Context: `location`, `if in location`, `limit_except`


This directive appeared in version 0.5.2.

Sets an njs function as a response body filter. The filter function is called for each data chunk of a response body with the following arguments:

- `r`

  the [HTTP request](https://nginx.org/en/docs/njs/reference.html#http) object

- `data`

  the incoming data chunk, may be a string or Buffer depending on the `buffer_type` value, by default is a string.

- `flags`

  an object with the following properties:`last`a boolean value, true if data is a last buffer.



The filter function can pass its own modified version of the input data chunk to the next body filter by calling [`r.sendBuffer()`](https://nginx.org/en/docs/njs/reference.html#r_sendbuffer). For example, to transform all the lowercase letters in the response body:

```
function filter(r, data, flags) {
    r.sendBuffer(data.toLowerCase(), flags);
}
```

To stop filtering (following data chunks will be passed to client without calling `js_body_filter`), [`r.done()`](https://nginx.org/en/docs/njs/reference.html#r_done) can be used.

If the filter function changes the length of the response body, then it is required to clear out the “Content-Length” response header (if any) in [`js_header_filter`]({{< ref "/mod_ref/ngx_http_js_module#js_header_filter">}}) to enforce chunked transfer encoding.



As the `js_body_filter` handler returns its result immediately, it supports only synchronous operations. Thus, asynchronous operations such as [r.subrequest()](https://nginx.org/en/docs/njs/reference.html#r_subrequest) or [setTimeout()](https://nginx.org/en/docs/njs/reference.html#settimeout) are not supported.





The directive can be specified inside the [if]({{< ref "/mod_ref/ngx_http_rewrite_module#if">}}) block since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_content

  Syntax:  `js_content function | module.function;`

  Default: —

  Context: `location`, `if in location`, `limit_except`


Sets an njs function as a location content handler. Since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0), a module function can be referenced.



The directive can be specified inside the [if]({{< ref "/mod_ref/ngx_http_rewrite_module#if">}}) block since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_fetch_buffer_size

  Syntax:`js_fetch_buffer_size size;`

  Default: `js_fetch_buffer_size 16k;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.4.

Sets the `size` of the buffer used for reading and writing with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_ciphers

  Syntax:  `js_fetch_ciphers ciphers;`

  Default: `js_fetch_ciphers HIGH:!aNULL:!MD5;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.0.

Specifies the enabled ciphers for HTTPS requests with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch). The ciphers are specified in the format understood by the OpenSSL library.

The full list can be viewed using the “`openssl ciphers`” command.



### js_fetch_max_response_buffer_size

  Syntax:`js_fetch_max_response_buffer_size size;`

  Default: `js_fetch_max_response_buffer_size 1m;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.4.

Sets the maximum `size` of the response received with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_protocols

  Syntax:`js_fetch_protocols [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];`

  Default: `js_fetch_protocols TLSv1 TLSv1.1 TLSv1.2;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.0.

Enables the specified protocols for HTTPS requests with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_timeout

  Syntax:  `js_fetch_timeout time;`

  Default: `js_fetch_timeout 60s;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.4.

Defines a timeout for reading and writing for [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch). The timeout is set only between two successive read/write operations, not for the whole response. If no data is transmitted within this time, the connection is closed.



### js_fetch_trusted_certificate

  Syntax:`js_fetch_trusted_certificate file;`

  Default: —

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.0.

Specifies a `file` with trusted CA certificates in the PEM format used to [verify](https://nginx.org/en/docs/njs/reference.html#fetch_verify) the HTTPS certificate with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_verify

  Syntax:  `js_fetch_verify on | off;`

  Default: `js_fetch_verify on;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.4.

Enables or disables verification of the HTTPS server certificate with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_fetch_verify_depth

  Syntax:`js_fetch_verify_depth number;`

  Default: `js_fetch_verify_depth 100;`

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.0.

Sets the verification depth in the HTTPS server certificates chain with [Fetch API](https://nginx.org/en/docs/njs/reference.html#ngx_fetch).



### js_header_filter

  Syntax:`js_header_filter function | module.function;`

  Default: —

  Context: `location`, `if in location`, `limit_except`


This directive appeared in version 0.5.1.

Sets an njs function as a response header filter. The directive allows changing arbitrary header fields of a response header.



As the `js_header_filter` handler returns its result immediately, it supports only synchronous operations. Thus, asynchronous operations such as [r.subrequest()](https://nginx.org/en/docs/njs/reference.html#r_subrequest) or [setTimeout()](https://nginx.org/en/docs/njs/reference.html#settimeout) are not supported.





The directive can be specified inside the [if]({{< ref "/mod_ref/ngx_http_rewrite_module#if">}}) block since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_import

  Syntax:`js_import module.js | export_name from module.js;`

  Default: —

  Context: `http`, `server`, `location`


This directive appeared in version 0.4.0.

Imports a module that implements location and variable handlers in njs. The `export_name` is used as a namespace to access module functions. If the `export_name` is not specified, the module name will be used as a namespace.

```
js_import http.js;
```

Here, the module name `http` is used as a namespace while accessing exports. If the imported module exports `foo()`, `http.foo` is used to refer to it.

Several `js_import` directives can be specified.



The directive can be specified on the `server` and `location` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_include

  Syntax:`js_include file;`

  Default: —

  Context: `http`


Specifies a file that implements location and variable handlers in njs:

```
nginx.conf:
js_include http.js;
location   /version {
    js_content version;
}

http.js:
function version(r) {
    r.return(200, njs.version);
}
```



The directive was made obsolete in version [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0) and was removed in version [0.7.1](https://nginx.org/en/docs/njs/changes.html#njs0.7.1). The [js_import]({{< ref "/mod_ref/ngx_http_js_module#js_import">}}) directive should be used instead.



### js_path

  Syntax:  `js_path path;`

  Default: —

  Context: `http`, `server`, `location`


This directive appeared in version 0.3.0.

Sets an additional path for njs modules.



The directive can be specified on the `server` and `location` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_preload_object

  Syntax:`js_preload_object name.json | name from file.json;`

  Default: —

  Context: `http`, `server`, `location`


This directive appeared in version 0.7.8.

Preloads an immutable object at configure time. The `name` is used a name of the global variable though which the object is available in njs code. If the `name` is not specified, the file name will be used instead.

```
js_preload_object map.json;
```

Here, the `map` is used as a name while accessing the preloaded object.

Several `js_preload_object` directives can be specified.



### js_set

  Syntax:`js_set $variable function | module.function;`

  Default: —

  Context: `http`, `server`, `location`


Sets an njs `function` for the specified `variable`. Since [0.4.0](https://nginx.org/en/docs/njs/changes.html#njs0.4.0), a module function can be referenced.

The function is called when the variable is referenced for the first time for a given request. The exact moment depends on a [phase](https://nginx.org/en/docs/dev/development_guide.html#http_phases) at which the variable is referenced. This can be used to perform some logic not related to variable evaluation. For example, if the variable is referenced only in the [log_format]({{< ref "/mod_ref/ngx_http_log_module#log_format">}}) directive, its handler will not be executed until the log phase. This handler can be used to do some cleanup right before the request is freed.



As the `js_set` handler returns its result immediately, it supports only synchronous operations. Thus, asynchronous operations such as [r.subrequest()](https://nginx.org/en/docs/njs/reference.html#r_subrequest) or [setTimeout()](https://nginx.org/en/docs/njs/reference.html#settimeout) are not supported.





The directive can be specified on the `server` and `location` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





### js_shared_dict_zone

  Syntax:`js_shared_dict_zone zone=name:size [timeout=time] [type=string|number] [evict];`

  Default: —

  Context: `http`


This directive appeared in version 0.8.0.

Sets the `name` and `size` of the shared memory zone that keeps the key-value dictionary shared between worker processes.

By default the shared dictionary uses a string as a key and a value. The optional `type` parameter allows redefining the value type to number.

The optional `timeout` parameter sets the time after which all shared dictionary entries are removed from the zone.

The optional `evict` parameter removes the oldest key-value pair when the zone storage is exhausted.

Examples:

```
example.conf:
    # Creates a 1Mb dictionary with string values,
    # removes key-value pairs after 60 seconds of inactivity:
    js_shared_dict_zone zone=foo:1M timeout=60s;

    # Creates a 512Kb dictionary with string values,
    # forcibly removes oldest key-value pairs when the zone is exhausted:
    js_shared_dict_zone zone=bar:512K timeout=30s evict;

    # Creates a 32Kb permanent dictionary with number values:
    js_shared_dict_zone zone=num:32k type=number;

example.js:
    function get(r) {
        r.return(200, ngx.shared.foo.get(r.args.key));
    }

    function set(r) {
        r.return(200, ngx.shared.foo.set(r.args.key, r.args.value));
    }

    function delete(r) {
        r.return(200, ngx.shared.bar.delete(r.args.key));
    }

    function increment(r) {
        r.return(200, ngx.shared.num.incr(r.args.key, 2));
    }
```





### js_var

  Syntax:  `js_var $variable [value];`

  Default: —

  Context: `http`, `server`, `location`


This directive appeared in version 0.5.3.

Declares a [writable](https://nginx.org/en/docs/njs/reference.html#r_variables) variable. The value can contain text, variables, and their combination. The variable is not overwritten after a redirect unlike variables created with the [set]({{< ref "/mod_ref/ngx_http_rewrite_module#set">}}) directive.



The directive can be specified on the `server` and `location` level since [0.7.7](https://nginx.org/en/docs/njs/changes.html#njs0.7.7).





Request Argument

Each HTTP njs handler receives one argument, a request [object](https://nginx.org/en/docs/njs/reference.html#http).