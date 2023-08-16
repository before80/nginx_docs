+++
title = "ngx_http_dav_module"
date = 2023-08-15T08:13:12+08:00
weight = 110
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Module ngx_http_dav_module

https://nginx.org/en/docs/http/ngx_http_dav_module.html



The `ngx_http_dav_module` module is intended for file management automation via the WebDAV protocol. The module processes HTTP and WebDAV methods PUT, DELETE, MKCOL, COPY, and MOVE.

​	`ngx_http_dav_module` 模块旨在通过WebDAV协议实现文件管理自动化。该模块处理HTTP和WebDAV方法PUT、DELETE、MKCOL、COPY和MOVE。

This module is not built by default, it should be enabled with the `--with-http_dav_module` configuration parameter.

​	此模块默认未构建，应通过`--with-http_dav_module`配置参数启用。

WebDAV clients that require additional WebDAV methods to operate will not work with this module.

​	需要额外WebDAV方法以进行操作的WebDAV客户端将无法使用此模块。



## 示例配置 Example Configuration



```
location / {
    root                  /data/www;

    client_body_temp_path /data/client_temp;

    dav_methods PUT DELETE MKCOL COPY MOVE;

    create_full_put_path  on;
    dav_access            group:rw  all:r;

    limit_except GET {
        allow 192.168.1.0/32;
        deny  all;
    }
}
```





## 指令 Directives

### create_full_put_path

  Syntax:`create_full_put_path on | off;`

  Default: `create_full_put_path off;`

  Context: `http`, `server`, `location`


The WebDAV specification only allows creating files in already existing directories. This directive allows creating all needed intermediate directories.

​	WebDAV规范仅允许在已经存在的目录中创建文件。此指令允许创建所有所需的中间目录。

### dav_access

  Syntax:`dav_access users:permissions ...;`

  Default: `dav_access user:rw;`

  Context: `http`, `server`, `location`

Sets access permissions for newly created files and directories, e.g.:

​	设置新创建的文件和目录的访问权限，例如：

```
dav_access user:rw group:rw all:r;
```

If any `group` or `all` access permissions are specified then `user` permissions may be omitted:

​	如果指定了任何`group`或`all`访问权限，则可以省略`user`权限：

```
dav_access group:rw all:r;
```





### dav_methods

  Syntax:`dav_methods off | method ...;`

  Default: `dav_methods off;`

  Context: `http`, `server`, `location`

Allows the specified HTTP and WebDAV methods. The parameter `off` denies all methods processed by this module. The following methods are supported: `PUT`, `DELETE`, `MKCOL`, `COPY`, and `MOVE`.

​	允许指定的HTTP和WebDAV方法。参数`off`拒绝此模块处理的所有方法。支持以下方法：`PUT`、`DELETE`、`MKCOL`、`COPY`和`MOVE`。

A file uploaded with the PUT method is first written to a temporary file, and then the file is renamed. Starting from version 0.8.9, temporary files and the persistent store can be put on different file systems. However, be aware that in this case a file is copied across two file systems instead of the cheap renaming operation. It is thus recommended that for any given location both saved files and a directory holding temporary files, set by the [client_body_temp_path]({{< ref "/mod_ref/ngx_http_core_module#client_body_temp_path">}}) directive, are put on the same file system.

​	使用PUT方法上传的文件首先写入临时文件，然后将文件重命名。从版本0.8.9开始，临时文件和持久存储可以放在不同的文件系统上。但是，请注意，在这种情况下，文件将在两个文件系统之间复制，而不是便宜的重命名操作。因此，建议在给定位置为保存的文件和保存临时文件的目录（由[client_body_temp_path]({{< ref "/mod_ref/ngx_http_core_module#client_body_temp_path">}})指令设置）使用相同的文件系统。

When creating a file with the PUT method, it is possible to specify the modification date by passing it in the “Date” header field.

​	使用PUT方法创建文件时，可以通过在“Date”头字段中传递修改日期来指定。



### min_delete_depth

  Syntax:  `min_delete_depth number;`

  Default: `min_delete_depth 0;`

  Context: `http`, `server`, `location`

Allows the DELETE method to remove files provided that the number of elements in a request path is not less than the specified number. For example, the directive

​	允许DELETE方法删除文件，前提是请求路径中的元素数不少于指定的数字。例如，指令

```
min_delete_depth 4;
```

allows removing files on requests

允许在请求为

```
/users/00/00/name
/users/00/00/name/pic.jpg
/users/00/00/page.html
```

and denies the removal of

时删除文件，并拒绝删除

```
/users/00/00
```

