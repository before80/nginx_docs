+++
title = "设置哈希表"
date = 2023-08-14T16:51:08+08:00
weight = 70
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Setting up hashes - 设置哈希表

https://nginx.org/en/docs/hash.html

To quickly process static sets of data such as server names, [map]({{< ref "/mod_ref/ngx_http_map_module#map">}}) directive’s values, MIME types, names of request header strings, nginx uses hash tables. During the start and each re-configuration nginx selects the minimum possible sizes of hash tables such that the bucket size that stores keys with identical hash values does not exceed the configured parameter (hash bucket size). The size of a table is expressed in buckets. The adjustment is continued until the table size exceeds the hash max size parameter. Most hashes have the corresponding directives that allow changing these parameters, for example, for the server names hash they are [server_names_hash_max_size]({{< ref "/mod_ref/ngx_http_core_module#server_names_hash_max_size">}}) and [server_names_hash_bucket_size]({{< ref "/mod_ref/ngx_http_core_module#server_names_hash_bucket_size">}}).

​	为了快速处理静态数据集，例如服务器名称、[map]({{< ref "/mod_ref/ngx_http_map_module#map">}}) 指令的值、MIME 类型、请求头字符串的名称，nginx 使用哈希表。在启动和每次重新配置期间，nginx 会选择最小可能的哈希表大小，以确保存储具有相同哈希值的键的桶大小不超过配置参数（哈希桶大小）。表的大小以桶为单位表示。此调整将继续，直到表的大小超过哈希最大大小参数。大多数哈希都有相应的指令，允许更改这些参数，例如，对于服务器名称哈希，它们是 [server_names_hash_max_size]({{< ref "/mod_ref/ngx_http_core_module#server_names_hash_max_size">}}) 和 [server_names_hash_bucket_size]({{< ref "/mod_ref/ngx_http_core_module#server_names_hash_bucket_size">}})。



The hash bucket size parameter is aligned to the size that is a multiple of the processor’s cache line size. This speeds up key search in a hash on modern processors by reducing the number of memory accesses. If hash bucket size is equal to one processor’s cache line size then the number of memory accesses during the key search will be two in the worst case — first to compute the bucket address, and second during the key search inside the bucket. Therefore, if nginx emits the message requesting to increase either hash max size or hash bucket size then the first parameter should first be increased.

​	哈希桶大小参数对齐到处理器缓存行大小的倍数。这通过减少内存访问次数来加速在现代处理器上的哈希键搜索。如果哈希桶大小等于一个处理器的缓存行大小，那么在键搜索期间的内存访问次数将在最坏情况下为两次 —— 第一次计算桶地址，第二次在桶内进行键搜索。因此，如果nginx发出要求增加哈希最大大小或哈希桶大小的消息，则应首先增加第一个参数。