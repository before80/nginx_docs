+++
title = "ngx_otel_module"
date = 2023-08-15T08:25:32+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_otel_module

https://nginx.org/en/docs/ngx_otel_module.html



The `ngx_otel_module` module (1.23.4) provides [OpenTelemetry](https://opentelemetry.io/) distributed tracing support. The module supports [W3C](https://w3c.github.io/trace-context) context propagation and OTLP/gRPC export protocol.



> This module is available as part of our [commercial subscription](http://nginx.com/products/) in `nginx-plus-module-otel` package. After installation, the module can be loaded [dynamically](https://nginx.org/en/docs/ngx_core_module.html#load_module).





## Example Configuration



> ```
> load_module modules/ngx_otel_module.so;
> 
> events {
> }
> 
> http {
> 
>     otel_exporter {
>         endpoint localhost:4317;
>     }
> 
>     server {
>         listen 127.0.0.1:8080;
> 
>         location / {
>             otel_trace         on;
>             otel_trace_context inject;
> 
>             proxy_pass http://backend;
>         }
>     }
> }
> ```





## Directives



### otel_exporter

| Syntax:  | `otel_exporter { ... }` |
| :------- | ----------------------- |
| Default: | —                       |
| Context: | `http`                  |

Specifies OTel data export parameters:

- `endpoint`

  the address of OTLP/gRPC endpoint that will accept telemetry data.

- `interval`

  the maximum interval between two exports, by default is `5` seconds.

- `batch_size`

  the maximum number of spans to be sent in one batch per worker, by default is `512`.

- `batch_count`

  the number of pending batches per worker, spans exceeding the limit are dropped, by default is `4`.

Example:

> ```
> otel_exporter {
>     endpoint    localhost:4317;
>     interval    5s;
>     batch_size  512;
>     batch_count 4;
> }
> ```





### otel_service_name

| Syntax:  | `otel_service_name name;`                  |
| :------- | ------------------------------------------ |
| Default: | `otel_service_name unknown_service:nginx;` |
| Context: | `http`                                     |

Sets the “[`service.name`](https://opentelemetry.io/docs/reference/specification/resource/semantic_conventions/#service)” attribute of the OTel resource.



### otel_trace

| Syntax:  | `otel_trace on | off | $variable;` |
| :------- | ---------------------------------- |
| Default: | `otel_trace off;`                  |
| Context: | `http`, `server`, `location`       |

Enables or disables OpenTelemetry tracing. The directive can also be enabled by specifying a variable:

> ```
> split_clients "$otel_trace_id" $ratio_sampler {
>               10%              on;
>               *                off;
> }
> 
> server {
>     location / {
>         otel_trace         $ratio_sampler;
>         otel_trace_context inject;
>         proxy_pass         http://backend;
>     }
> }
> ```





### otel_trace_context

| Syntax:  | `otel_trace_context extract | inject | propagate | ignore;` |
| :------- | ----------------------------------------------------------- |
| Default: | `otel_trace_context ignore;`                                |
| Context: | `http`, `server`, `location`                                |

Specifies how to propagate [traceparent/tracestate](https://www.w3.org/TR/trace-context/#design-overview) headers:

- `extract`

  uses an existing trace context from the request, so that the identifiers of a [trace](https://nginx.org/en/docs/ngx_otel_module.html#var_otel_trace_id) and the [parent span](https://nginx.org/en/docs/ngx_otel_module.html#var_otel_parent_id) are inherited from the incoming request.

- `inject`

  adds a new context to the request, overwriting existing headers, if any.

- `propagate`

  updates the existing context (combines [extract](https://nginx.org/en/docs/ngx_otel_module.html#extract) and [inject](https://nginx.org/en/docs/ngx_otel_module.html#inject)).

- `ignore`

  skips context headers processing.





### otel_span_name

| Syntax:  | `otel_span_name name;`       |
| :------- | ---------------------------- |
| Default: | —                            |
| Context: | `http`, `server`, `location` |

Defines the name of the OTel [span](https://opentelemetry.io/docs/concepts/observability-primer/#spans). By default, it is a name of the location for a request. The name can contain variables.



### otel_span_attr

| Syntax:  | `otel_span_attr name value;` |
| :------- | ---------------------------- |
| Default: | —                            |
| Context: | `http`, `server`, `location` |

Adds a custom OTel span attribute. The value can contain variables.



## Default span attributes

The following [span attributes](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/semantic_conventions/http.md) are added automatically:

- `http.method`
- `http.target`
- `http.route`
- `http.scheme`
- `http.flavor`
- `http.user_agent`
- `http.request_content_length`
- `http.response_content_length`
- `http.status_code`
- `net.host.name`
- `net.host.port`
- `net.sock.peer.addr`
- `net.sock.peer.port`





## Embedded Variables



- `$otel_trace_id`

  the identifier of the trace the current span belongs to, for example, `56552bc4daa3bf39c08362527e1dd6c4`

- `$otel_span_id`

  the identifier of the current span, for example, `4c0b8531ec38ca59`

- `$otel_parent_id`

  the identifier of the parent span, for example, `dc94d281b0f884ea`

- `$otel_parent_sampled`

  the “`sampled`” flag of the parent span, can be “`1`” or “`0`”