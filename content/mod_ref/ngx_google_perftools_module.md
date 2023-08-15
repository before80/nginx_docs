+++
title = "ngx_google_perftools_module"
date = 2023-08-15T08:25:24+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Module ngx_google_perftools_module

https://nginx.org/en/docs/ngx_google_perftools_module.html





The `ngx_google_perftools_module` module (0.6.29) enables profiling of nginx worker processes using [Google Performance Tools](https://github.com/gperftools/gperftools). The module is intended for nginx developers.

This module is not built by default, it should be enabled with the `--with-google_perftools_module` configuration parameter.

This module requires the [gperftools](https://github.com/gperftools/gperftools) library.





## Example Configuration



```
google_perftools_profiles /path/to/profile;
```

Profiles will be stored as `/path/to/profile.<worker_pid>`.



## Directives



### google_perftools_profiles

  Syntax:`google_perftools_profiles file;`

| Default: | —                                 |
  Context: `main`


Sets a file name that keeps profiling information of nginx worker process. The ID of the worker process is always a part of the file name and is appended to the end of the file name, after a dot.