+++
title = "ngx_http_fastcgi_module"
date = 2023-08-15T08:13:37+08:00
weight = 140
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_fastcgi_module

https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html



The `ngx_http_fastcgi_module` module allows passing requests to a FastCGI server.

​	`ngx_http_fastcgi_module` 模块允许将请求传递给 FastCGI 服务器。

## 示例配置 Example Configuration



```
location / {
    fastcgi_pass  localhost:9000;
    fastcgi_index index.php;

    fastcgi_param SCRIPT_FILENAME /home/www/scripts/php$fastcgi_script_name;
    fastcgi_param QUERY_STRING    $query_string;
    fastcgi_param REQUEST_METHOD  $request_method;
    fastcgi_param CONTENT_TYPE    $content_type;
    fastcgi_param CONTENT_LENGTH  $content_length;
}
```





## 指令 Directives



### fastcgi_bind

  Syntax:`fastcgi_bind address [transparent] | off;`

  Default: —

  Context: `http`, `server`, `location`

This directive appeared in version 0.8.22.

​	此指令在版本0.8.22中出现。

Makes outgoing connections to a FastCGI server originate from the specified local IP address with an optional port (1.11.2). Parameter value can contain variables (1.3.12). The special value `off` (1.3.12) cancels the effect of the `fastcgi_bind` directive inherited from the previous configuration level, which allows the system to auto-assign the local IP address and port.

​	使发往FastCGI服务器的外发连接从指定的本地IP地址（可选端口）开始（1.11.2）。参数值可以包含变量（1.3.12）。特殊值`off`（1.3.12）取消从先前配置级别继承的`fastcgi_bind`指令的影响，这使得系统可以自动分配本地IP地址和端口。

The `transparent` parameter (1.11.0) allows outgoing connections to a FastCGI server originate from a non-local IP address, for example, from a real IP address of a client:

​	`transparent`参数（1.11.0）允许从非本地IP地址（例如客户端的真实IP地址）开始发往FastCGI服务器的外发连接：

```
fastcgi_bind $remote_addr transparent;
```

In order for this parameter to work, it is usually necessary to run nginx worker processes with the [superuser]({{< ref "/mod_ref/ngx_core_module#user">}}) privileges. On Linux it is not required (1.13.8) as if the `transparent` parameter is specified, worker processes inherit the `CAP_NET_RAW` capability from the master process. It is also necessary to configure kernel routing table to intercept network traffic from the FastCGI server.

​	为了使此参数起作用，通常需要以[超级用户]({{< ref "/mod_ref/ngx_core_module#user">}})特权运行nginx worker进程。在Linux上（1.13.8），不需要这样做，因为如果指定了`transparent`参数，则worker进程从主进程继承了`CAP_NET_RAW`能力。还需要配置内核路由表以拦截来自FastCGI服务器的网络流量。

### fastcgi_buffer_size

  Syntax:  `fastcgi_buffer_size size;`

  Default: `fastcgi_buffer_size 4k|8k;`

  Context: `http`, `server`, `location`


Sets the `size` of the buffer used for reading the first part of the response received from the FastCGI server. This part usually contains a small response header. By default, the buffer size is equal to one memory page. This is either 4K or 8K, depending on a platform. It can be made smaller, however.

​	设置用于从FastCGI服务器接收的响应的第一部分的缓冲区的大小。此部分通常包含一个小的响应头。默认情况下，缓冲区大小等于一个内存页。这取决于平台，可以是4K或8K。但也可以设置得更小。

### fastcgi_buffering

  Syntax:`fastcgi_buffering on | off;`

  Default: `fastcgi_buffering on;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.5.6.

​	此指令在版本1.5.6中出现。

Enables or disables buffering of responses from the FastCGI server.

​	启用或禁用从FastCGI服务器缓冲响应。

When buffering is enabled, nginx receives a response from the FastCGI server as soon as possible, saving it into the buffers set by the [fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}}) and [fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}}) directives. If the whole response does not fit into memory, a part of it can be saved to a [temporary file]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_path">}}) on the disk. Writing to temporary files is controlled by the [fastcgi_max_temp_file_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_max_temp_file_size">}}) and [fastcgi_temp_file_write_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_file_write_size">}}) directives.

​	当启用缓冲时，nginx尽快从FastCGI服务器接收响应，将其保存到由[fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}})和[fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}})指令设置的缓冲区中。如果整个响应无法放入内存中，则其中的一部分可以保存到磁盘上的[临时文件]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_path">}})中。写入临时文件由[fastcgi_max_temp_file_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_max_temp_file_size">}})和[fastcgi_temp_file_write_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_file_write_size">}})指令控制。

When buffering is disabled, the response is passed to a client synchronously, immediately as it is received. nginx will not try to read the whole response from the FastCGI server. The maximum size of the data that nginx can receive from the server at a time is set by the [fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}}) directive.

​	当禁用缓冲时，响应将同步传递给客户端，即立即在接收到响应时传递给客户端。nginx不会尝试从FastCGI服务器读取整个响应。nginx从服务器一次可以接收的数据的最大大小由[fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}})指令设置。

Buffering can also be enabled or disabled by passing “`yes`” or “`no`” in the “X-Accel-Buffering” response header field. This capability can be disabled using the [fastcgi_ignore_headers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_ignore_headers">}}) directive.

​	还可以通过在“X-Accel-Buffering”响应头字段中传递“yes”或“no”来启用或禁用缓冲。可以使用[fastcgi_ignore_headers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_ignore_headers">}})指令禁用此功能。

### fastcgi_buffers

  Syntax:`fastcgi_buffers number size;`

  Default: `fastcgi_buffers 8 4k|8k;`

  Context: `http`, `server`, `location`


Sets the `number` and `size` of the buffers used for reading a response from the FastCGI server, for a single connection. By default, the buffer size is equal to one memory page. This is either 4K or 8K, depending on a platform.

​	设置用于在单个连接中从FastCGI服务器读取响应的缓冲区的`number`和`size`。默认情况下，缓冲区大小等于一个内存页。这取决于平台，可以是4K或8K。

### fastcgi_busy_buffers_size

  Syntax:  `fastcgi_busy_buffers_size size;`

  Default: `fastcgi_busy_buffers_size 8k|16k;`

  Context: `http`, `server`, `location`


When [buffering]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}}) of responses from the FastCGI server is enabled, limits the total `size` of buffers that can be busy sending a response to the client while the response is not yet fully read. In the meantime, the rest of the buffers can be used for reading the response and, if needed, buffering part of the response to a temporary file. By default, `size` is limited by the size of two buffers set by the [fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}}) and [fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}}) directives.

​	当启用[缓冲]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}})从FastCGI服务器接收的响应时，限制了在响应尚未完全读取的情况下，可以忙于将响应发送给客户端的缓冲区的总大小。同时，其余的缓冲区可以用于读取响应并且在需要时将响应的一部分缓冲到临时文件中。默认情况下，`size`受到由[fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}})和[fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}})指令设置的两个缓冲区大小的限制。

### fastcgi_cache

  Syntax:  `fastcgi_cache zone | off;`

  Default: `fastcgi_cache off;`

  Context: `http`, `server`, `location`


Defines a shared memory zone used for caching. The same zone can be used in several places. Parameter value can contain variables (1.7.9). The `off` parameter disables caching inherited from the previous configuration level.

​	定义用于缓存的共享内存区域。同一区域可以在多个地方使用。参数值可以包含变量（1.7.9）。`off`参数取消从先前配置级别继承的缓存。

### fastcgi_cache_background_update

  Syntax:`fastcgi_cache_background_update on | off;`

  Default: `fastcgi_cache_background_update off;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.11.10.

​	此指令在版本1.11.10中出现。

Allows starting a background subrequest to update an expired cache item, while a stale cached response is returned to the client. Note that it is necessary to [allow]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_use_stale_updating">}}) the usage of a stale cached response when it is being updated.

​	允许启动后台子请求来更新已过期的缓存项目，同时向客户端返回过期的缓存响应。请注意，当正在更新时，需要[允许]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_use_stale_updating">}})使用已过时的缓存响应。

### fastcgi_cache_bypass

  Syntax:`fastcgi_cache_bypass string ...;`

  Default: —

  Context: `http`, `server`, `location`

Defines conditions under which the response will not be taken from a cache. If at least one value of the string parameters is not empty and is not equal to “0” then the response will not be taken from the cache:

​	定义响应不会从缓存中获取的条件。如果至少有一个字符串参数的值不为空且不等于“0”，则响应将不会从缓存中获取：

```
fastcgi_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
fastcgi_cache_bypass $http_pragma    $http_authorization;
```

Can be used along with the [fastcgi_no_cache]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_no_cache">}}) directive.

​	可以与[fastcgi_no_cache]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_no_cache">}})指令一起使用。

### fastcgi_cache_key

  Syntax:  `fastcgi_cache_key string;`

  Default: —

  Context: `http`, `server`, `location`

Defines a key for caching, for example

​	定义用于缓存的键，例如

```
fastcgi_cache_key localhost:9000$request_uri;
```





### fastcgi_cache_lock

  Syntax:`fastcgi_cache_lock on | off;`

  Default: `fastcgi_cache_lock off;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.1.12.

​	此指令在版本1.1.12中出现。

When enabled, only one request at a time will be allowed to populate a new cache element identified according to the [fastcgi_cache_key]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_key">}}) directive by passing a request to a FastCGI server. Other requests of the same cache element will either wait for a response to appear in the cache or the cache lock for this element to be released, up to the time set by the [fastcgi_cache_lock_timeout]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_lock_timeout">}}) directive.

​	启用后，只允许一个请求一次填充一个新的缓存元素，根据[fastcgi_cache_key]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_key">}})指令将请求传递给FastCGI服务器。同一缓存元素的其他请求要么等待缓存中出现响应，要么等待此元素的缓存锁释放，最多等待[fastcgi_cache_lock_timeout]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_lock_timeout">}})指令设置的时间。

### fastcgi_cache_lock_age

  Syntax:`fastcgi_cache_lock_age time;`

  Default: `fastcgi_cache_lock_age 5s;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.7.8.

​	此指令在版本1.7.8中出现。

If the last request passed to the FastCGI server for populating a new cache element has not completed for the specified `time`, one more request may be passed to the FastCGI server.

​	如果用于填充新的缓存元素的最后一个请求在指定的`time`内尚未完成，那么可以再次将一个请求传递给FastCGI服务器。

### fastcgi_cache_lock_timeout

  Syntax:`fastcgi_cache_lock_timeout time;`

  Default: `fastcgi_cache_lock_timeout 5s;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.1.12.

​	此指令在版本1.1.12中出现。

Sets a timeout for [fastcgi_cache_lock]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_lock">}}). When the `time` expires, the request will be passed to the FastCGI server, however, the response will not be cached.

​	为[fastcgi_cache_lock]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_lock">}})设置超时。当`time`到期时，请求将传递给FastCGI服务器，但不会缓存响应。

Before 1.7.8, the response could be cached.

​	在1.7.8之前，响应可能会被缓存。



### fastcgi_cache_max_range_offset

  Syntax:`fastcgi_cache_max_range_offset number;`

  Default: —

  Context: `http`, `server`, `location`

This directive appeared in version 1.11.6.

​	此指令在版本1.11.6中出现。

Sets an offset in bytes for byte-range requests. If the range is beyond the offset, the range request will be passed to the FastCGI server and the response will not be cached.

​	为字节范围请求设置偏移量。如果范围超出偏移量，则范围请求将传递给FastCGI服务器，响应不会被缓存。

### fastcgi_cache_methods

  Syntax:`fastcgi_cache_methods GET | HEAD | POST ...;`

  Default: `fastcgi_cache_methods GET HEAD;`

  Context: `http`, `server`, `location`

This directive appeared in version 0.7.59.

​	此指令在版本0.7.59中出现。

If the client request method is listed in this directive then the response will be cached. “`GET`” and “`HEAD`” methods are always added to the list, though it is recommended to specify them explicitly. See also the [fastcgi_no_cache]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_no_cache">}}) directive.

​	如果客户端请求方法列在此指令中，则响应将被缓存。“GET”和“HEAD”方法始终添加到列表中，尽管建议明确指定它们。还请参阅[fastcgi_no_cache]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_no_cache">}})指令。

### fastcgi_cache_min_uses

  Syntax:`fastcgi_cache_min_uses number;`

  Default: `fastcgi_cache_min_uses 1;`

  Context: `http`, `server`, `location`


Sets the `number` of requests after which the response will be cached.

​	设置请求的次数，超过该次数后响应将被缓存。

### fastcgi_cache_path

Syntax:`fastcgi_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [min_free=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];`

  Default: —

  Context: `http`

Sets the path and other parameters of a cache. Cache data are stored in files. Both the key and file name in a cache are a result of applying the MD5 function to the proxied URL. The `levels` parameter defines hierarchy levels of a cache: from 1 to 3, each level accepts values 1 or 2. For example, in the following configuration

​	设置缓存的路径和其他参数。缓存数据存储在文件中。缓存中的键和文件名都是将MD5函数应用于代理的URL得到的结果。`levels`参数定义缓存的层级：从1到3，每个层级可以接受值1或2。例如，在以下配置中：

```
fastcgi_cache_path /data/nginx/cache levels=1:2 keys_zone=one:10m;
```

file names in a cache will look like this:

缓存中的文件名将如下所示：

```
/data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c
```

A cached response is first written to a temporary file, and then the file is renamed. Starting from version 0.8.9, temporary files and the cache can be put on different file systems. However, be aware that in this case a file is copied across two file systems instead of the cheap renaming operation. It is thus recommended that for any given location both cache and a directory holding temporary files are put on the same file system. A directory for temporary files is set based on the `use_temp_path` parameter (1.7.10). If this parameter is omitted or set to the value `on`, the directory set by the [fastcgi_temp_path]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_path">}}) directive for the given location will be used. If the value is set to `off`, temporary files will be put directly in the cache directory.

​	缓存的响应首先被写入临时文件，然后文件被重命名。从0.8.9版本开始，临时文件和缓存可以位于不同的文件系统上。但是，请注意，在这种情况下，文件将在两个文件系统之间复制，而不是进行廉价的重命名操作。因此，建议对于给定的位置，缓存和包含临时文件的目录都放在同一个文件系统上。临时文件的目录是基于`use_temp_path`参数设置的（1.7.10）。如果省略此参数或将其设置为`on`，则将使用由给定位置的[fastcgi_temp_path]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_path">}})指令设置的目录。如果值设置为`off`，则临时文件将直接放在缓存目录中。

In addition, all active keys and information about data are stored in a shared memory zone, whose `name` and `size` are configured by the `keys_zone` parameter. One megabyte zone can store about 8 thousand keys.

​	此外，所有活动的键和关于数据的信息都存储在一个由`keys_zone`参数配置的共享内存区域中。一个兆字节的区域可以存储大约8千个键。

As part of [commercial subscription](http://nginx.com/products/), the shared memory zone also stores extended cache [information]({{< ref "/mod_ref/ngx_http_api_module#http_caches_">}}), thus, it is required to specify a larger zone size for the same number of keys. For example, one megabyte zone can store about 4 thousand keys.

​	作为[商业订阅](http://nginx.com/products/)的一部分，共享内存区域还存储扩展缓存[信息]({{< ref "/mod_ref/ngx_http_api_module#http_caches_">}})，因此，需要为相同数量的键指定更大的区域大小。例如，一个兆字节的区域可以存储大约4千个键。

Cached data that are not accessed during the time specified by the `inactive` parameter get removed from the cache regardless of their freshness. By default, `inactive` is set to 10 minutes.

​	未在`inactive`参数指定的时间内未访问的缓存数据将从缓存中删除，而不考虑其新鲜度。默认情况下，`inactive`设置为10分钟。

The special “cache manager” process monitors the maximum cache size set by the `max_size` parameter, and the minimum amount of free space set by the `min_free` (1.19.1) parameter on the file system with cache. When the size is exceeded or there is not enough free space, it removes the least recently used data. The data is removed in iterations configured by `manager_files`, `manager_threshold`, and `manager_sleep` parameters (1.11.5). During one iteration no more than `manager_files` items are deleted (by default, 100). The duration of one iteration is limited by the `manager_threshold` parameter (by default, 200 milliseconds). Between iterations, a pause configured by the `manager_sleep` parameter (by default, 50 milliseconds) is made.

​	特殊的“缓存管理器”进程监视由`max_size`参数设置的最大缓存大小，以及在具有缓存的文件系统上由`min_free`（1.19.1）参数设置的最小剩余空间。当超过大小或没有足够的剩余空间时，它会删除最近最少使用的数据。数据是在由`manager_files`、`manager_threshold`和`manager_sleep`参数（1.11.5）配置的迭代中删除的。在一个迭代中，不会删除超过`manager_files`项（默认为100）。一个迭代的持续时间由`manager_threshold`参数（默认为200毫秒）限制。在迭代之间，会进行由`manager_sleep`参数（默认为50毫秒）配置的暂停。

A minute after the start the special “cache loader” process is activated. It loads information about previously cached data stored on file system into a cache zone. The loading is also done in iterations. During one iteration no more than `loader_files` items are loaded (by default, 100). Besides, the duration of one iteration is limited by the `loader_threshold` parameter (by default, 200 milliseconds). Between iterations, a pause configured by the `loader_sleep` parameter (by default, 50 milliseconds) is made.

​	启动1分钟后，会激活特殊的“缓存加载器”进程。它将文件系统上存储的先前缓存数据的信息加载到缓存区域中。加载也是在迭代中进行的。在一个迭代中，不会加载超过`loader_files`项（默认为100）。此外，一个迭代的持续时间由`loader_threshold`参数（默认为200毫秒）限制。在迭代之间，会进行由`loader_sleep`参数（默认为50毫秒）配置的暂停。

Additionally, the following parameters are available as part of our [commercial subscription](http://nginx.com/products/):

​	此外，以下参数也可作为[商业订阅](http://nginx.com/products/)的一部分提供：

- `purger`=`on`|`off`

  Instructs whether cache entries that match a [wildcard key]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_purge">}}) will be removed from the disk by the cache purger (1.7.12). Setting the parameter to `on` (default is `off`) will activate the “cache purger” process that permanently iterates through all cache entries and deletes the entries that match the wildcard key.

  指示是否将与[通配符键]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_purge">}})匹配的缓存条目从磁盘中移除缓存清除程序（1.7.12）。将参数设置为`on`（默认为`off`）将激活“缓存清除程序”进程，该进程将永久迭代所有缓存条目并删除与通配符键匹配的条目。

- `purger_files`=`number`

  Sets the number of items that will be scanned during one iteration (1.7.12). By default, `purger_files` is set to 10.

  设置在一个迭代中将被扫描的项的数量（1.7.12）。默认情况下，`purger_files`设置为10。

- `purger_threshold`=`number`

  Sets the duration of one iteration (1.7.12). By default, `purger_threshold` is set to 50 milliseconds.

  设置一个迭代的持续时间（1.7.12）。默认情况下，`purger_threshold`设置为50毫秒。

- `purger_sleep`=`number`

  Sets a pause between iterations (1.7.12). By default, `purger_sleep` is set to 50 milliseconds.
  
  设置迭代之间的暂停（1.7.12）。默认情况下，`purger_sleep`设置为50毫秒。

In versions 1.7.3, 1.7.7, and 1.11.10 cache header format has been changed. Previously cached responses will be considered invalid after upgrading to a newer nginx version.

​	在版本1.7.3、1.7.7和1.11.10中，缓存头格式已更改。升级到新版本的nginx后，之前缓存的响应将被视为无效。



### fastcgi_cache_purge

  Syntax:`fastcgi_cache_purge string ...;`

  Default: —

  Context: `http`, `server`, `location`

This directive appeared in version 1.5.7.

​	此指令在版本1.5.7中出现。

Defines conditions under which the request will be considered a cache purge request. If at least one value of the string parameters is not empty and is not equal to “0” then the cache entry with a corresponding [cache key]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_key">}}) is removed. The result of successful operation is indicated by returning the 204 (No Content) response.

​	定义在何种条件下将请求视为缓存清除请求。如果至少有一个字符串参数的值不为空且不等于“0”，则将删除具有相应的[缓存键]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_key">}})的缓存条目。成功操作的结果由返回204（无内容）响应指示。

If the [cache key]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_key">}}) of a purge request ends with an asterisk (“`*`”), all cache entries matching the wildcard key will be removed from the cache. However, these entries will remain on the disk until they are deleted for either [inactivity]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_path">}}), or processed by the [cache purger]({{< ref "/mod_ref/ngx_http_fastcgi_module#purger">}}) (1.7.12), or a client attempts to access them.

​	如果缓存清除请求的[缓存键]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_key">}})以星号（“`*`”）结尾，则将从缓存中删除与通配符键匹配的所有缓存条目。但是，这些条目将保留在磁盘上，直到它们因[非活动]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_path">}})或由[缓存清除程序]({{< ref "/mod_ref/ngx_http_fastcgi_module#purger">}})（1.7.12）处理，或客户端试图访问它们而被删除。

Example configuration:

​	示例配置：

```
fastcgi_cache_path /data/nginx/cache keys_zone=cache_zone:10m;

map $request_method $purge_method {
    PURGE   1;
    default 0;
}

server {
    ...
    location / {
        fastcgi_pass        backend;
        fastcgi_cache       cache_zone;
        fastcgi_cache_key   $uri;
        fastcgi_cache_purge $purge_method;
    }
}
```

This functionality is available as part of our [commercial subscription](http://nginx.com/products/).

​	此功能作为[商业订阅](http://nginx.com/products/)的一部分提供。



### fastcgi_cache_revalidate

  Syntax:`fastcgi_cache_revalidate on | off;`

  Default: `fastcgi_cache_revalidate off;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.5.7.

​	此指令在版本1.5.7中出现。

Enables revalidation of expired cache items using conditional requests with the “If-Modified-Since” and “If-None-Match” header fields.

​	启用使用带有“If-Modified-Since”和“If-None-Match”头字段的条件请求来重新验证过期的缓存项。

### fastcgi_cache_use_stale

  Syntax:`fastcgi_cache_use_stale error | timeout | invalid_header | updating | http_500 | http_503 | http_403 | http_404 | http_429 | off ...;`

  Default: `fastcgi_cache_use_stale off;`

  Context: `http`, `server`, `location`

Determines in which cases a stale cached response can be used when an error occurs during communication with the FastCGI server. The directive’s parameters match the parameters of the [fastcgi_next_upstream]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}}) directive.

​	确定在与FastCGI服务器通信期间发生错误时可以在哪些情况下使用旧的缓存响应。指令的参数与[fastcgi_next_upstream]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}})指令的参数匹配。

The `error` parameter also permits using a stale cached response if a FastCGI server to process a request cannot be selected.

​	`error`参数还允许在无法选择FastCGI服务器来处理请求时使用旧的缓存响应。

Additionally, the `updating` parameter permits using a stale cached response if it is currently being updated. This allows minimizing the number of accesses to FastCGI servers when updating cached data.

​	此外，`updating`参数允许在当前正在更新缓存响应时使用旧的缓存响应。这允许在更新缓存数据时将对FastCGI服务器的访问次数最小化。

Using a stale cached response can also be enabled directly in the response header for a specified number of seconds after the response became stale (1.11.10). This has lower priority than using the directive parameters.

​	还可以直接在响应头中启用在响应变得陈旧后一定秒数内使用旧的缓存响应（1.11.10）。这优先级低于使用指令参数。 

- The “[stale-while-revalidate](https://datatracker.ietf.org/doc/html/rfc5861#section-3)” extension of the “Cache-Control” header field permits using a stale cached response if it is currently being updated.
- “Cache-Control”的“[stale-while-revalidate](https://datatracker.ietf.org/doc/html/rfc5861#section-3)”扩展允许在当前更新缓存响应时使用旧的缓存响应。
- The “[stale-if-error](https://datatracker.ietf.org/doc/html/rfc5861#section-4)” extension of the “Cache-Control” header field permits using a stale cached response in case of an error.
- “Cache-Control”的“[stale-if-error](https://datatracker.ietf.org/doc/html/rfc5861#section-4)”扩展允许在错误情况下使用旧的缓存响应。



To minimize the number of accesses to FastCGI servers when populating a new cache element, the [fastcgi_cache_lock]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_lock">}}) directive can be used.

​	为了在填充新的缓存元素时将对FastCGI服务器的访问次数最小化，可以使用[fastcgi_cache_lock]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_lock">}})指令。



### fastcgi_cache_valid

  Syntax:`fastcgi_cache_valid [code ...] time;`

  Default: —

  Context: `http`, `server`, `location`

Sets caching time for different response codes. For example, the following directives

​	设置不同响应代码的缓存时间。例如，以下指令：

```
fastcgi_cache_valid 200 302 10m;
fastcgi_cache_valid 404      1m;
```

set 10 minutes of caching for responses with codes 200 and 302 and 1 minute for responses with code 404.

为响应代码为200和302设置了10分钟的缓存时间，为响应代码为404设置了1分钟的缓存时间。

If only caching `time` is specified

​	如果只指定缓存`time`：

```
fastcgi_cache_valid 5m;
```

then only 200, 301, and 302 responses are cached.

则仅缓存200、301和302的响应。

In addition, the `any` parameter can be specified to cache any responses:

​	此外，还可以指定`any`参数来缓存任何响应：

```
fastcgi_cache_valid 200 302 10m;
fastcgi_cache_valid 301      1h;
fastcgi_cache_valid any      1m;
```

Parameters of caching can also be set directly in the response header. This has higher priority than setting of caching time using the directive.

缓存的参数也可以直接在响应头中设置。这比使用指令设置缓存时间具有更高的优先级。

- The “X-Accel-Expires” header field sets caching time of a response in seconds. The zero value disables caching for a response. If the value starts with the `@` prefix, it sets an absolute time in seconds since Epoch, up to which the response may be cached.
- “X-Accel-Expires”头字段以秒为单位设置响应的缓存时间。零值禁用响应的缓存。如果值以`@`前缀开头，则它设置了自纪元以来的绝对时间，直到此时间响应可能会被缓存。
- If the header does not include the “X-Accel-Expires” field, parameters of caching may be set in the header fields “Expires” or “Cache-Control”.
- 如果头字段中不包含“X-Accel-Expires”字段，则可以在“Expires”或“Cache-Control”头字段中设置缓存的参数。
- If the header includes the “Set-Cookie” field, such a response will not be cached.
- 如果头字段包含“Set-Cookie”字段，则不会缓存此响应。
- If the header includes the “Vary” field with the special value “`*`”, such a response will not be cached (1.7.7). If the header includes the “Vary” field with another value, such a response will be cached taking into account the corresponding request header fields (1.7.7).
- 如果头字段包含带有特殊值“`*`”的“Vary”字段，则不会缓存此响应（1.7.7）。如果头字段包含其他值的“Vary”字段，则会缓存此响应，并考虑相应的请求头字段（1.7.7）。

Processing of one or more of these response header fields can be disabled using the [fastcgi_ignore_headers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_ignore_headers">}}) directive.

​	可以使用[fastcgi_ignore_headers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_ignore_headers">}})指令禁用这些响应头字段中的一个或多个的处理。

### fastcgi_catch_stderr

  Syntax:`fastcgi_catch_stderr string;`

  Default: —

  Context: `http`, `server`, `location`

Sets a string to search for in the error stream of a response received from a FastCGI server. If the `string` is found then it is considered that the FastCGI server has returned an [invalid response]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}}). This allows handling application errors in nginx, for example:

​	设置要在从FastCGI服务器接收的响应的错误流中搜索的字符串。如果找到`string`，则认为FastCGI服务器返回了[无效响应]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}})。这允许在nginx中处理应用程序错误，例如：

```
location /php/ {
    fastcgi_pass backend:9000;
    ...
    fastcgi_catch_stderr "PHP Fatal error";
    fastcgi_next_upstream error timeout invalid_header;
}
```



### fastcgi_connect_timeout

  Syntax:`fastcgi_connect_timeout time;`

  Default: `fastcgi_connect_timeout 60s;`

  Context: `http`, `server`, `location`


Defines a timeout for establishing a connection with a FastCGI server. It should be noted that this timeout cannot usually exceed 75 seconds.

​	定义与FastCGI服务器建立连接的超时时间。应注意，此超时通常不能超过75秒。

### fastcgi_force_ranges

  Syntax:`fastcgi_force_ranges on | off;`

  Default: `fastcgi_force_ranges off;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.7.7.

​	此指令在版本1.7.7中出现。

Enables byte-range support for both cached and uncached responses from the FastCGI server regardless of the “Accept-Ranges” field in these responses.

​	无论这些响应中的“Accept-Ranges”字段如何，都会为来自FastCGI服务器的缓存和未缓存的响应启用字节范围支持。

### fastcgi_hide_header

  Syntax:`fastcgi_hide_header field;`

  Default: —

  Context: `http`, `server`, `location`


By default, nginx does not pass the header fields “Status” and “X-Accel-...” from the response of a FastCGI server to a client. The `fastcgi_hide_header` directive sets additional fields that will not be passed. If, on the contrary, the passing of fields needs to be permitted, the [fastcgi_pass_header]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass_header">}}) directive can be used.

​	默认情况下，nginx不会将来自FastCGI服务器的响应的头字段“Status”和“X-Accel-...”传递给客户端。`fastcgi_hide_header`指令设置不会传递的其他字段。相反，如果需要允许传递字段，则可以使用[fastcgi_pass_header]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass_header">}})指令。

### fastcgi_ignore_client_abort

  Syntax:`fastcgi_ignore_client_abort on | off;`

  Default: `fastcgi_ignore_client_abort off;`

  Context: `http`, `server`, `location`


Determines whether the connection with a FastCGI server should be closed when a client closes the connection without waiting for a response.

​	确定在客户端关闭连接而不等待响应时，是否应关闭与FastCGI服务器的连接。

### fastcgi_ignore_headers

  Syntax:`fastcgi_ignore_headers field ...;`

  Default: —

  Context: `http`, `server`, `location`

Disables processing of certain response header fields from the FastCGI server. The following fields can be ignored: “X-Accel-Redirect”, “X-Accel-Expires”, “X-Accel-Limit-Rate” (1.1.6), “X-Accel-Buffering” (1.1.6), “X-Accel-Charset” (1.1.6), “Expires”, “Cache-Control”, “Set-Cookie” (0.8.44), and “Vary” (1.7.7).

​	禁用从FastCGI服务器处理某些响应头字段。以下字段可以被忽略：`“X-Accel-Redirect”`、`“X-Accel-Expires”`、`“X-Accel-Limit-Rate”`（1.1.6）、`“X-Accel-Buffering”`（1.1.6）、`“X-Accel-Charset”`（1.1.6）、`“Expires”`、`“Cache-Control”`、`“Set-Cookie”`（0.8.44）和`“Vary”`（1.7.7）。

If not disabled, processing of these header fields has the following effect:

​	如果没有禁用这些头字段的处理，则会产生以下效果：

- “X-Accel-Expires”, “Expires”, “Cache-Control”, “Set-Cookie”, and “Vary” set the parameters of response [caching]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_valid">}});
- “X-Accel-Redirect” performs an [internal redirect]({{< ref "/mod_ref/ngx_http_core_module#internal">}}) to the specified URI;
- “X-Accel-Limit-Rate” sets the [rate limit]({{< ref "/mod_ref/ngx_http_core_module#limit_rate">}}) for transmission of a response to a client;
- “X-Accel-Buffering” enables or disables [buffering]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}}) of a response;
- “X-Accel-Charset” sets the desired [charset]({{< ref "/mod_ref/ngx_http_charset_module#charset">}}) of a response.
- `“X-Accel-Expires”`、`“Expires”`、`“Cache-Control”`、`“Set-Cookie”`和`“Vary”`会设置响应的缓存参数；
- `“X-Accel-Redirect”`执行到指定URI的[内部重定向]({{< ref "/mod_ref/ngx_http_core_module#internal">}})；
- `“X-Accel-Limit-Rate”`设置传输响应给客户端的[速率限制]({{< ref "/mod_ref/ngx_http_core_module#limit_rate">}})；
- `“X-Accel-Buffering”`启用或禁用响应的[缓冲]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}})；
- `“X-Accel-Charset”`设置响应的期望[字符集]({{< ref "/mod_ref/ngx_http_charset_module#charset">}})。





### fastcgi_index

  Syntax:  `fastcgi_index name;`

  Default: —

  Context: `http`, `server`, `location`

Sets a file name that will be appended after a URI that ends with a slash, in the value of the `$fastcgi_script_name` variable. For example, with these settings

​	设置一个文件名，在以斜线结尾的URI之后，在`$fastcgi_script_name`变量的值中追加该文件名。例如，使用以下设置：

```
fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME /home/www/scripts/php$fastcgi_script_name;
```

and the “`/page.php`” request, the `SCRIPT_FILENAME` parameter will be equal to “`/home/www/scripts/php/page.php`”, and with the “`/`” request it will be equal to “`/home/www/scripts/php/index.php`”.

对于“`/page.php`”请求，`SCRIPT_FILENAME`参数将等于“`/home/www/scripts/php/page.php`”，而对于“`/`”请求，它将等于“`/home/www/scripts/php/index.php`”。

### fastcgi_intercept_errors

  Syntax:`fastcgi_intercept_errors on | off;`

  Default: `fastcgi_intercept_errors off;`

  Context: `http`, `server`, `location`


Determines whether FastCGI server responses with codes greater than or equal to 300 should be passed to a client or be intercepted and redirected to nginx for processing with the [error_page]({{< ref "/mod_ref/ngx_http_core_module#error_page">}}) directive.

​	确定是否应将大于或等于300的FastCGI服务器响应代码传递给客户端，还是拦截并将其重定向到nginx以使用[error_page]({{< ref "/mod_ref/ngx_http_core_module#error_page">}})指令进行处理。

### fastcgi_keep_conn

  Syntax:`fastcgi_keep_conn on | off;`

  Default: `fastcgi_keep_conn off;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.1.4.

​	此指令在版本1.1.4中出现。

By default, a FastCGI server will close a connection right after sending the response. However, when this directive is set to the value `on`, nginx will instruct a FastCGI server to keep connections open. This is necessary, in particular, for [keepalive]({{< ref "/mod_ref/ngx_http_upstream_module#keepalive">}}) connections to FastCGI servers to function.

​	默认情况下，FastCGI服务器将在发送响应后立即关闭连接。但是，当将此指令设置为`on`时，nginx将指示FastCGI服务器保持连接打开。这在特定情况下是必需的，特别是用于使FastCGI服务器的[keepalive]({{< ref "/mod_ref/ngx_http_upstream_module#keepalive">}})连接正常工作。

### fastcgi_limit_rate

  Syntax:  `fastcgi_limit_rate rate;`

  Default: `fastcgi_limit_rate 0;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.7.7.

​	此指令在版本1.7.7中出现。

Limits the speed of reading the response from the FastCGI server. The `rate` is specified in bytes per second. The zero value disables rate limiting. The limit is set per a request, and so if nginx simultaneously opens two connections to the FastCFI server, the overall rate will be twice as much as the specified limit. The limitation works only if [buffering]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}}) of responses from the FastCGI server is enabled.

​	限制从FastCGI服务器读取响应的速度。`rate`以每秒字节数指定。零值会禁用速率限制。限制是针对每个请求设置的，因此如果nginx同时打开两个到FastCGI服务器的连接，则总速率将是指定限制的两倍。此限制仅在启用了从FastCGI服务器的响应的[缓冲]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}})时才起作用。

### fastcgi_max_temp_file_size

  Syntax:  `fastcgi_max_temp_file_size size;`

  Default: `fastcgi_max_temp_file_size 1024m;`

  Context: `http`, `server`, `location`

When [buffering]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}}) of responses from the FastCGI server is enabled, and the whole response does not fit into the buffers set by the [fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}}) and [fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}}) directives, a part of the response can be saved to a temporary file. This directive sets the maximum `size` of the temporary file. The size of data written to the temporary file at a time is set by the [fastcgi_temp_file_write_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_file_write_size">}}) directive.

​	当启用从FastCGI服务器的响应的[缓冲]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffering">}})并且整个响应不适合由[fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}})和[fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}})指令设置的缓冲区时，部分响应可以保存到临时文件中。此指令设置临时文件的最大`size`。每次写入临时文件的数据大小由[fastcgi_temp_file_write_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_file_write_size">}})指令设置。

The zero value disables buffering of responses to temporary files.

​	零值会禁用响应缓冲到临时文件。

This restriction does not apply to responses that will be [cached]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache">}}) or [stored]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_store">}}) on disk.

​	这个限制不适用于将要在磁盘上进行[缓存]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache">}})或[存储]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_store">}})的响应。



### fastcgi_next_upstream

  Syntax:`fastcgi_next_upstream error | timeout | invalid_header | http_500 | http_503 | http_403 | http_404 | http_429 | non_idempotent | off ...;`

  Default: `fastcgi_next_upstream error timeout;`

  Context: `http`, `server`, `location`

Specifies in which cases a request should be passed to the next server:

​	指定请求在哪些情况下应该传递给下一个服务器：

- `error`

  an error occurred while establishing a connection with the server, passing a request to it, or reading the response header;

  在与服务器建立连接、传递请求或读取响应头时发生错误；

- `timeout`

  a timeout has occurred while establishing a connection with the server, passing a request to it, or reading the response header;

  在与服务器建立连接、传递请求或读取响应头时发生超时；

- `invalid_header`

  a server returned an empty or invalid response;

  服务器返回空响应或无效响应；

- `http_500`

  a server returned a response with the code 500;

  服务器返回状态码为 500 的响应；

- `http_503`

  a server returned a response with the code 503;

  服务器返回状态码为 503 的响应；

- `http_403`

  a server returned a response with the code 403;

  服务器返回状态码为 403 的响应；

- `http_404`

  a server returned a response with the code 404;

  服务器返回状态码为 404 的响应；

- `http_429`

  a server returned a response with the code 429 (1.11.13);

  服务器返回状态码为 429 的响应（1.11.13 版本后）；

- `non_idempotent`

  normally, requests with a [non-idempotent](https://datatracker.ietf.org/doc/html/rfc7231#section-4.2.2) method (`POST`, `LOCK`, `PATCH`) are not passed to the next server if a request has been sent to an upstream server (1.9.13); enabling this option explicitly allows retrying such requests;

  通常，对于 [非幂等](https://datatracker.ietf.org/doc/html/rfc7231#section-4.2.2) 方法（如 `POST`、`LOCK`、`PATCH`）的请求，如果已经向上游服务器发送了请求，则不会传递给下一个服务器（1.9.13 版本之后）；启用此选项可以显式允许重试此类请求；

- `off`

  disables passing a request to the next server.
  
  禁用将请求传递给下一个服务器。



One should bear in mind that passing a request to the next server is only possible if nothing has been sent to a client yet. That is, if an error or timeout occurs in the middle of the transferring of a response, fixing this is impossible.

​	需要注意，只有在尚未向客户端发送任何内容之前，才有可能将请求传递给下一个服务器。也就是说，如果在传输响应的过程中出现错误或超时，将无法修复。

The directive also defines what is considered an [unsuccessful attempt]({{< ref "/mod_ref/ngx_http_upstream_module#max_fails">}}) of communication with a server. The cases of `error`, `timeout` and `invalid_header` are always considered unsuccessful attempts, even if they are not specified in the directive. The cases of `http_500`, `http_503`, and `http_429` are considered unsuccessful attempts only if they are specified in the directive. The cases of `http_403` and `http_404` are never considered unsuccessful attempts.

​	该指令还定义了与服务器通信的[不成功尝试]({{< ref "/mod_ref/ngx_http_upstream_module#max_fails">}})的条件。无论是否在指令中指定，错误、超时和无效头字段的情况始终被视为不成功尝试。只有在指令中指定的情况下，`http_500`、`http_503` 和 `http_429` 才被视为不成功尝试。`http_403` 和 `http_404` 的情况永远不被视为不成功尝试。

Passing a request to the next server can be limited by [the number of tries]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream_tries">}}) and by [time]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream_timeout">}}).

​	将请求传递给下一个服务器可以通过 [尝试次数]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream_tries">}}) 和 [时间]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream_timeout">}}) 进行限制。

### fastcgi_next_upstream_timeout

  Syntax:`fastcgi_next_upstream_timeout time;`

  Default: `fastcgi_next_upstream_timeout 0;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.7.5.

​	此指令在版本 1.7.5 中引入。

Limits the time during which a request can be passed to the [next server]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}}). The `0` value turns off this limitation.

​	限制请求可以传递给[下一个服务器]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}})的时间。将值设置为 `0` 可以关闭此限制。

### fastcgi_next_upstream_tries

  Syntax:`fastcgi_next_upstream_tries number;`

  Default: `fastcgi_next_upstream_tries 0;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.7.5.

​	此指令在版本 1.7.5 中引入。

Limits the number of possible tries for passing a request to the [next server]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}}). The `0` value turns off this limitation.

​	限制尝试将请求传递给[下一个服务器]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}})的次数。将值设置为 `0` 可以关闭此限制。

### fastcgi_no_cache

  Syntax:`fastcgi_no_cache string ...;`

  Default: —

  Context: `http`, `server`, `location`

Defines conditions under which the response will not be saved to a cache. If at least one value of the string parameters is not empty and is not equal to “0” then the response will not be saved:

​	定义不保存响应到缓存的条件。如果至少有一个字符串参数的值不为空且不等于“0”，则响应不会被保存：

```
fastcgi_no_cache $cookie_nocache $arg_nocache$arg_comment;
fastcgi_no_cache $http_pragma    $http_authorization;
```

Can be used along with the [fastcgi_cache_bypass]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_bypass">}}) directive.

​	可以与 [fastcgi_cache_bypass]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_bypass">}}) 指令一起使用。

### fastcgi_param

  Syntax:`fastcgi_param parameter value [if_not_empty];`

  Default: —

  Context: `http`, `server`, `location`

Sets a `parameter` that should be passed to the FastCGI server. The `value` can contain text, variables, and their combination. These directives are inherited from the previous configuration level if and only if there are no `fastcgi_param` directives defined on the current level.

​	设置要传递给 FastCGI 服务器的 `parameter`。`value` 可以包含文本、变量及其组合。如果在当前级别未定义 `fastcgi_param` 指令，则这些指令会从前一个配置级别继承。

The following example shows the minimum required settings for PHP:

​	以下示例显示了用于 PHP 的最小必需设置：

```
fastcgi_param SCRIPT_FILENAME /home/www/scripts/php$fastcgi_script_name;
fastcgi_param QUERY_STRING    $query_string;
```



The `SCRIPT_FILENAME` parameter is used in PHP for determining the script name, and the `QUERY_STRING` parameter is used to pass request parameters.

​	`SCRIPT_FILENAME` 参数用于确定脚本名称，`QUERY_STRING` 参数用于传递请求参数。

For scripts that process `POST` requests, the following three parameters are also required:

​	对于处理 `POST` 请求的脚本，还需要以下三个参数：

```
fastcgi_param REQUEST_METHOD  $request_method;
fastcgi_param CONTENT_TYPE    $content_type;
fastcgi_param CONTENT_LENGTH  $content_length;
```



If PHP was built with the `--enable-force-cgi-redirect` configuration parameter, the `REDIRECT_STATUS` parameter should also be passed with the value “200”:

​	如果 PHP 使用了 `--enable-force-cgi-redirect` 配置参数构建，则还应传递具有值“200”的 `REDIRECT_STATUS` 参数：

```
fastcgi_param REDIRECT_STATUS 200;
```

If the directive is specified with `if_not_empty` (1.1.11) then such a parameter will be passed to the server only if its value is not empty:

​	如果使用了 `if_not_empty`（1.1.11 版本之后），则只有在其值不为空时，此参数才会传递给服务器：

```
fastcgi_param HTTPS           $https if_not_empty;
```





### fastcgi_pass

  Syntax:  `fastcgi_pass address;`

  Default: —

  Context: `location`, `if in location`

Sets the address of a FastCGI server. The address can be specified as a domain name or IP address, and a port:

​	设置 FastCGI 服务器的地址。地址可以指定为域名或 IP 地址以及端口：

```
fastcgi_pass localhost:9000;
```

or as a UNIX-domain socket path:

​	或指定为 UNIX 域套接字路径：

```
fastcgi_pass unix:/tmp/fastcgi.socket;
```

If a domain name resolves to several addresses, all of them will be used in a round-robin fashion. In addition, an address can be specified as a [server group](https://nginx.org/en/docs/http/ngx_http_upstream_module.html).

​	如果域名解析为多个地址，则会轮询使用这些地址。此外，地址可以指定为[服务器组](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)。

Parameter value can contain variables. In this case, if an address is specified as a domain name, the name is searched among the described [server groups](https://nginx.org/en/docs/http/ngx_http_upstream_module.html), and, if not found, is determined using a [resolver]({{< ref "/mod_ref/ngx_http_core_module#resolver">}}).

​	参数值可以包含变量。在这种情况下，如果地址指定为域名，则会在描述的 [服务器组](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) 中查找该名称；如果找不到，则使用 [resolver]({{< ref "/mod_ref/ngx_http_core_module#resolver">}}) 来确定。

### fastcgi_pass_header

  Syntax:`fastcgi_pass_header field;`

  Default: —

  Context: `http`, `server`, `location`

Permits passing [otherwise disabled]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_hide_header">}}) header fields from a FastCGI server to a client.

​	允许将从 FastCGI 服务器传递的[本来被禁用]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_hide_header">}})的头字段传递给客户端。



### fastcgi_pass_request_body

  Syntax:`fastcgi_pass_request_body on | off;`

  Default: `fastcgi_pass_request_body on;`

  Context: `http`, `server`, `location`


Indicates whether the original request body is passed to the FastCGI server. See also the [fastcgi_pass_request_headers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass_request_headers">}}) directive.

​	指示是否将原始请求体传递给 FastCGI 服务器。也请参阅 [fastcgi_pass_request_headers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass_request_headers">}}) 指令。

### fastcgi_pass_request_headers

  Syntax:`fastcgi_pass_request_headers on | off;`

  Default: `fastcgi_pass_request_headers on;`

  Context: `http`, `server`, `location`


Indicates whether the header fields of the original request are passed to the FastCGI server. See also the [fastcgi_pass_request_body]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass_request_body">}}) directive.

​	指示是否将原始请求的头字段传递给 FastCGI 服务器。也请参阅 [fastcgi_pass_request_body]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_pass_request_body">}}) 指令。

### fastcgi_read_timeout

  Syntax:`fastcgi_read_timeout time;`

  Default: `fastcgi_read_timeout 60s;`

  Context: `http`, `server`, `location`


Defines a timeout for reading a response from the FastCGI server. The timeout is set only between two successive read operations, not for the transmission of the whole response. If the FastCGI server does not transmit anything within this time, the connection is closed.

​	定义从 FastCGI 服务器读取响应的超时时间。超时仅在两次连续的读取操作之间设置，不适用于整个响应的传输。如果 FastCGI 服务器在此时间内未传输任何内容，则连接将关闭。

### fastcgi_request_buffering

  Syntax:`fastcgi_request_buffering on | off;`

  Default: `fastcgi_request_buffering on;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.7.11.

​	此指令在版本 1.7.11 中引入。

Enables or disables buffering of a client request body.

​	启用或禁用对客户端请求体的缓冲。

When buffering is enabled, the entire request body is [read]({{< ref "/mod_ref/ngx_http_core_module#client_body_buffer_size">}}) from the client before sending the request to a FastCGI server.

​	启用缓冲时，整个请求体从客户端读取，然后将请求发送到 FastCGI 服务器。

When buffering is disabled, the request body is sent to the FastCGI server immediately as it is received. In this case, the request cannot be passed to the [next server]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}}) if nginx already started sending the request body.

​	禁用缓冲时，请求体会在接收到时立即发送到 FastCGI 服务器。在这种情况下，如果 nginx 已经开始发送请求体，请求将无法传递给[下一个服务器]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_next_upstream">}})。

### fastcgi_send_lowat

  Syntax:  `fastcgi_send_lowat size;`

  Default: `fastcgi_send_lowat 0;`

  Context: `http`, `server`, `location`

If the directive is set to a non-zero value, nginx will try to minimize the number of send operations on outgoing connections to a FastCGI server by using either `NOTE_LOWAT` flag of the [kqueue]({{< ref "/introduction/connectionProcessingMethods#kqueue">}}) method, or the `SO_SNDLOWAT` socket option, with the specified `size`.

​	如果将此指令设置为非零值，则 nginx 将尝试通过使用 `kqueue` 方法的 `NOTE_LOWAT` 标志或使用指定的 `size` 的 `SO_SNDLOWAT` 套接字选项来减少传出连接到 FastCGI 服务器的发送操作次数。

This directive is ignored on Linux, Solaris, and Windows.

​	在 Linux、Solaris 和 Windows 上，此指令将被忽略。

### fastcgi_send_timeout

  Syntax:`fastcgi_send_timeout time;`

  Default: `fastcgi_send_timeout 60s;`

  Context: `http`, `server`, `location`


Sets a timeout for transmitting a request to the FastCGI server. The timeout is set only between two successive write operations, not for the transmission of the whole request. If the FastCGI server does not receive anything within this time, the connection is closed.

​	设置将请求传递到 FastCGI 服务器的超时时间。超时仅在两次连续的写操作之间设置，不适用于整个请求的传输。如果 FastCGI 服务器在此时间内未接收到任何内容，则连接将关闭。

### fastcgi_socket_keepalive

  Syntax:`fastcgi_socket_keepalive on | off;`

  Default: `fastcgi_socket_keepalive off;`

  Context: `http`, `server`, `location`

This directive appeared in version 1.15.6.

​	此指令在版本 1.15.6 中引入。

Configures the “TCP keepalive” behavior for outgoing connections to a FastCGI server. By default, the operating system’s settings are in effect for the socket. If the directive is set to the value “`on`”, the `SO_KEEPALIVE` socket option is turned on for the socket.

​	配置向 FastCGI 服务器的传出连接的“TCP keepalive”行为。默认情况下，套接字的操作系统设置对该套接字生效。如果将此指令设置为值“on”，则将为套接字打开 `SO_KEEPALIVE` 套接字选项。

### fastcgi_split_path_info

  Syntax:`fastcgi_split_path_info regex;`

  Default: —

  Context: `location`

Defines a regular expression that captures a value for the `$fastcgi_path_info` variable. The regular expression should have two captures: the first becomes a value of the `$fastcgi_script_name` variable, the second becomes a value of the `$fastcgi_path_info` variable. For example, with these settings

​	定义一个正则表达式，用于捕获 `$fastcgi_path_info` 变量的值。正则表达式应该有两个捕获组：第一个成为 `$fastcgi_script_name` 变量的值，第二个成为 `$fastcgi_path_info` 变量的值。例如，对于以下设置：

```
location ~ ^(.+\.php)(.*)$ {
    fastcgi_split_path_info       ^(.+\.php)(.*)$;
    fastcgi_param SCRIPT_FILENAME /path/to/php$fastcgi_script_name;
    fastcgi_param PATH_INFO       $fastcgi_path_info;
```

and the “`/show.php/article/0001`” request, the `SCRIPT_FILENAME` parameter will be equal to “`/path/to/php/show.php`”, and the `PATH_INFO` parameter will be equal to “`/article/0001`”.

对于请求“`/show.php/article/0001`”，`SCRIPT_FILENAME` 参数将等于“`/path/to/php/show.php`”，`PATH_INFO` 参数将等于“`/article/0001`”。

### fastcgi_store

  Syntax:`fastcgi_store on | off | string;`

  Default: `fastcgi_store off;`

  Context: `http`, `server`, `location`

Enables saving of files to a disk. The `on` parameter saves files with paths corresponding to the directives [alias]({{< ref "/mod_ref/ngx_http_core_module#alias">}}) or [root]({{< ref "/mod_ref/ngx_http_core_module#root">}}). The `off` parameter disables saving of files. In addition, the file name can be set explicitly using the `string` with variables:

​	启用将文件保存到磁盘。`on` 参数将文件保存在与 [alias]({{< ref "/mod_ref/ngx_http_core_module#alias">}}) 或 [root]({{< ref "/mod_ref/ngx_http_core_module#root">}}) 指令对应的路径。`off` 参数禁用文件保存。此外，可以使用带有变量的 `string` 来显式设置文件名：

```
fastcgi_store /data/www$original_uri;
```

The modification time of files is set according to the received “Last-Modified” response header field. The response is first written to a temporary file, and then the file is renamed. Starting from version 0.8.9, temporary files and the persistent store can be put on different file systems. However, be aware that in this case a file is copied across two file systems instead of the cheap renaming operation. It is thus recommended that for any given location both saved files and a directory holding temporary files, set by the [fastcgi_temp_path]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_path">}}) directive, are put on the same file system.

​	文件的修改时间将根据接收到的“Last-Modified”响应头字段设置。响应首先被写入临时文件，然后文件被重命名。从版本 0.8.9 开始，临时文件和持久存储可以放在不同的文件系统上。然而，请注意，在这种情况下，文件会在两个文件系统之间复制，而不是便宜的重命名操作。因此，建议在给定位置的情况下，保存的文件和保存临时文件的目录（由 [fastcgi_temp_path]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_temp_path">}}) 指令设置）都放在同一个文件系统上。

This directive can be used to create local copies of static unchangeable files, e.g.:

​	此指令可用于创建静态不可变文件的本地副本，例如：

```
location /images/ {
    root                 /data/www;
    error_page           404 = /fetch$uri;
}

location /fetch/ {
    internal;

    fastcgi_pass         backend:9000;
    ...

    fastcgi_store        on;
    fastcgi_store_access user:rw group:rw all:r;
    fastcgi_temp_path    /data/temp;

    alias                /data/www/;
}
```





### fastcgi_store_access

  Syntax:`fastcgi_store_access users:permissions ...;`

  Default: `fastcgi_store_access user:rw;`

  Context: `http`, `server`, `location`

Sets access permissions for newly created files and directories, e.g.:

​	设置新创建的文件和目录的访问权限，例如：

```
fastcgi_store_access user:rw group:rw all:r;
```

If any `group` or `all` access permissions are specified then `user` permissions may be omitted:

​	如果指定了任何 `group` 或 `all` 访问权限，则可以省略 `user` 权限：

```
fastcgi_store_access group:rw all:r;
```





### fastcgi_temp_file_write_size

  Syntax:  `fastcgi_temp_file_write_size size;`

  Default: `fastcgi_temp_file_write_size 8k|16k;`

  Context: `http`, `server`, `location`

Limits the `size` of data written to a temporary file at a time, when buffering of responses from the FastCGI server to temporary files is enabled. By default, `size` is limited by two buffers set by the [fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}}) and [fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}}) directives. The maximum size of a temporary file is set by the [fastcgi_max_temp_file_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_max_temp_file_size">}}) directive.

​	限制在启用从 FastCGI 服务器到临时文件的响应缓冲时，一次写入临时文件的数据的 `size`。默认情况下，`size` 受 [fastcgi_buffer_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffer_size">}}) 和 [fastcgi_buffers]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_buffers">}}) 指令设置的两个缓冲区的限制。临时文件的最大大小由 [fastcgi_max_temp_file_size]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_max_temp_file_size">}}) 指令设置。



### fastcgi_temp_path

  Syntax:`fastcgi_temp_path path [level1 [level2 [level3]]];`

  Default: `fastcgi_temp_path fastcgi_temp;`

  Context: `http`, `server`, `location`

Defines a directory for storing temporary files with data received from FastCGI servers. Up to three-level subdirectory hierarchy can be used underneath the specified directory. For example, in the following configuration

​	定义一个目录，用于存储从 FastCGI 服务器接收的数据的临时文件。在指定的目录下，可以使用高达三级子目录层次结构。例如，在以下配置中：

```
fastcgi_temp_path /spool/nginx/fastcgi_temp 1 2;
```

a temporary file might look like this:

一个临时文件可能如下所示：

```
/spool/nginx/fastcgi_temp/7/45/00000123457
```

See also the `use_temp_path` parameter of the [fastcgi_cache_path]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_path">}}) directive.

​	另请参阅 [fastcgi_cache_path]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_cache_path">}}) 指令的 `use_temp_path` 参数。

## 传递给 FastCGI 服务器的参数 Parameters Passed to a FastCGI Server

HTTP request header fields are passed to a FastCGI server as parameters. In applications and scripts running as FastCGI servers, these parameters are usually made available as environment variables. For example, the “User-Agent” header field is passed as the `HTTP_USER_AGENT` parameter. In addition to HTTP request header fields, it is possible to pass arbitrary parameters using the [fastcgi_param]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_param">}}) directive.

​	HTTP 请求头字段会作为参数传递给 FastCGI 服务器。在作为 FastCGI 服务器运行的应用程序和脚本中，这些参数通常以环境变量的形式提供。例如，“User-Agent”头字段将作为 `HTTP_USER_AGENT` 参数传递。除了 HTTP 请求头字段外，还可以使用 [fastcgi_param]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_param">}}) 指令传递任意参数。

## Embedded Variables

The `ngx_http_fastcgi_module` module supports embedded variables that can be used to set parameters using the [fastcgi_param]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_param">}}) directive:

​	`ngx_http_fastcgi_module` 模块支持嵌入式变量，可以使用 [fastcgi_param]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_param">}}) 指令来设置参数：

### `$fastcgi_script_name`

  request URI or, if a URI ends with a slash, request URI with an index file name configured by the [fastcgi_index]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_index">}}) directive appended to it. This variable can be used to set the `SCRIPT_FILENAME` and `PATH_TRANSLATED` parameters that determine the script name in PHP. For example, for the “`/info/`” request with the following directives 

  请求的 URI，或者如果 URI 以斜杠结尾，则附加由 [fastcgi_index]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_index">}}) 指令配置的索引文件名的请求 URI。此变量可用于设置在 PHP 中确定脚本名称的 `SCRIPT_FILENAME` 和 `PATH_TRANSLATED` 参数。例如，对于具有以下指令的“`/info/`”请求：

  ```
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME /home/www/scripts/php$fastcgi_script_name;
  ```

  the `SCRIPT_FILENAME` parameter will be equal to “`/home/www/scripts/php/info/index.php`”.When using the [fastcgi_split_path_info]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_split_path_info">}}) directive, the `$fastcgi_script_name` variable equals the value of the first capture set by the directive.

  `SCRIPT_FILENAME` 参数将等于“`/home/www/scripts/php/info/index.php`”。在使用 [fastcgi_split_path_info]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_split_path_info">}}) 指令时，`$fastcgi_script_name` 变量等于由该指令设置的第一个捕获组的值。

###  `$fastcgi_path_info`

  the value of the second capture set by the [fastcgi_split_path_info]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_split_path_info">}}) directive. This variable can be used to set the `PATH_INFO` parameter.
  
  由 [fastcgi_split_path_info]({{< ref "/mod_ref/ngx_http_fastcgi_module#fastcgi_split_path_info">}}) 指令设置的第二个捕获组的值。此变量可用于设置 `PATH_INFO` 参数。