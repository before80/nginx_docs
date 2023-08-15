+++
title = "ngx_stream_mqtt_preread_module"
date = 2023-08-15T08:23:24+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_stream_mqtt_preread_module

https://nginx.org/en/docs/stream/ngx_stream_mqtt_preread_module.html



The `ngx_stream_mqtt_preread_module` module (1.23.4) allows extracting information from the CONNECT message of the Message Queuing Telemetry Transport protocol (MQTT) versions [3.1.1](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html) and [5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html), for example, a username or a client ID.



> This module is available as part of our [commercial subscription](http://nginx.com/products/).





## Example Configuration



> ```
> mqtt_preread on;
> return       $mqtt_preread_clientid;
> ```





## Directives



### mqtt_preread

| Syntax:  | `mqtt_preread on | off;` |
| :------- | ------------------------ |
| Default: | `mqtt_preread off;`      |
| Context: | `stream`, `server`       |

Enables extracting information from the MQTT CONNECT message at the [preread](https://nginx.org/en/docs/stream/stream_processing.html#preread_phase) phase.



## Embedded Variables



- `$mqtt_preread_clientid`

  the `clientid` value from the CONNECT message

- `$mqtt_preread_username`

  the `username` value from the CONNECT message