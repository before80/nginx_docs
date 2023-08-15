+++
title = "ngx_http_proxy_protocol_vendor_module"
date = 2023-08-15T08:17:29+08:00
weight = 380
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_proxy_protocol_vendor_module

https://nginx.org/en/docs/http/ngx_http_proxy_protocol_vendor_module.html



The `ngx_http_proxy_protocol_vendor_module` module (1.23.3) allows obtaining additional information about a connection in cloud platforms from application-specific TLVs of the [PROXY protocol](http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt) header.

Supported cloud platforms:

- Amazon Web Services
- Google Cloud Platform
- Microsoft Azure



The PROXY protocol must be previously enabled by setting the `proxy_protocol` parameter in the [listen](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive.



This module is available as part of our [commercial subscription](http://nginx.com/products/).





### Example Configuration



```
proxy_set_header X-Conn-ID $proxy_protocol_tlv_gcp_conn_id;

server {
    listen 80   proxy_protocol;
    listen 443  ssl proxy_protocol;
    ...
}
```





### Embedded Variables



- `$proxy_protocol_tlv_aws_vpce_id`

  TLV value from the PROXY Protocol header representing the [ID of AWS VPC endpoint](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#proxy-protocol)

- `$proxy_protocol_tlv_azure_pel_id`

  TLV value from the PROXY Protocol header representing the [LinkID of Azure private endpoint](https://learn.microsoft.com/en-us/azure/private-link/private-link-service-overview#getting-connection-information-using-tcp-proxy-v2)

- `$proxy_protocol_tlv_gcp_conn_id`

  TLV value from the PROXY Protocol header representing [Google Cloud PSC connection ID](https://cloud.google.com/vpc/docs/configure-private-service-connect-producer#proxy-protocol)