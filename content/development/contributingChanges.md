+++
title = "contributingChanges"
date = 2023-08-14T17:01:06+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

# Contributing Changes

https://nginx.org/en/docs/contributing_changes.html



## Getting Sources

[Mercurial](https://www.mercurial-scm.org/) is used to store source code. The [repository](http://hg.nginx.org/nginx) can be cloned with the following command:

```
hg clone http://hg.nginx.org/nginx
```





## Formatting Changes

Changes should be formatted according to the [code style](https://nginx.org/en/docs/dev/development_guide.html#code_style) used by nginx. Sometimes, there is no clear rule; in such cases examine how existing nginx sources are formatted and mimic this style. Changes will more likely be accepted if style corresponds to the surrounding code.

Commit the changes to create a Mercurial [changeset](https://www.mercurial-scm.org/wiki/ChangeSet). Please ensure that the specified [e-mail](https://www.mercurial-scm.org/wiki/QuickStart#Setting_a_username) address and real name of the change’s author are correct.

The commit message should have a single-line synopsis followed by verbose description after an empty line. It is desirable that the first line is no longer than 67 symbols. The resulting changeset as a patch can be obtained using the `hg export` command:

```
# HG changeset patch
# User Filipe Da Silva <username@example.com>
# Date 1368089668 -7200
#      Thu May 09 10:54:28 2013 +0200
# Node ID 2220de0521ca2c0b664a8ea1e201ce1cb90fd7a2
# Parent  822b82191940ef309cd1e6502f94d50d811252a1
Mail: removed surplus ngx_close_connection() call.

It is already called for a peer connection a few lines above.

diff -r 822b82191940 -r 2220de0521ca src/mail/ngx_mail_auth_http_module.c
--- a/src/mail/ngx_mail_auth_http_module.c      Wed May 15 15:04:49 2013 +0400
+++ b/src/mail/ngx_mail_auth_http_module.c      Thu May 09 10:54:28 2013 +0200
@@ -699,7 +699,6 @@ ngx_mail_auth_http_process_headers(ngx_m

                     p = ngx_pnalloc(s->connection->pool, ctx->err.len);
                     if (p == NULL) {
-                        ngx_close_connection(ctx->peer.connection);
                         ngx_destroy_pool(ctx->pool);
                         ngx_mail_session_internal_server_error(s);
                         return;
```





## Before Submitting

Several points are worth to consider before submitting changes:

- The proposed changes should work properly on a wide range of [supported platforms](https://nginx.org/en/index.html#tested_os_and_platforms).

- Try to make it clear why the suggested change is needed, and provide a use case, if possible.

- Passing your changes through the test suite is a good way to ensure that they do not cause a regression. The repository with tests can be cloned with the following command:

  ```
  hg clone http://hg.nginx.org/nginx-tests
  ```





## Submitting Changes

The proposed changes should be sent to the [nginx development](https://nginx.org/en/support.html#nginx_devel) mailing list. The preferred and convenient method of submitting changesets is with the [patchbomb](https://www.mercurial-scm.org/wiki/PatchbombExtension) extension.



## License

Submitting changes implies granting project a permission to use it under an appropriate [license](https://nginx.org/LICENSE).