+++
title = "开发指南"
date = 2023-08-14T17:01:19+08:00
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Development guide - 开发指南

https://nginx.org/en/docs/dev/development_guide.html



## 简介 Introduction



### 代码布局 Code layout

- `auto` — Build scripts
- `auto` — 构建脚本
- `src`
  - `core` — Basic types and functions — string, array, log, pool, etc.
  - `core` — 基本类型和函数 — 字符串、数组、日志、池等。
  - `event` — Event core
  - `event` — 事件核心
    - `modules` — Event notification modules: `epoll`, `kqueue`, `select` etc.
    - `modules` — 事件通知模块：`epoll`、`kqueue`、`select` 等。
  - `http` — Core HTTP module and common code
  - `http` — 核心 HTTP 模块和通用代码
    - `modules` — Other HTTP modules
    - `modules` — 其他 HTTP 模块
    - `v2` — HTTP/2
    - `v2` — HTTP/2
  - `mail` — Mail modules
  - `mail` — 邮件模块
  - `os` — Platform-specific code
  - `os` — 平台特定代码
    - `unix`
    - `win32`
  - `stream` — Stream modules
  - `stream` — 流模块



### 包含文件 Include files

The following two `#include` statements must appear at the beginning of every nginx file:

​	每个 nginx 文件的开头都必须出现以下两个 `#include` 语句：

```
#include <ngx_config.h>
#include <ngx_core.h>
```

In addition to that, HTTP code should include

​	除此之外，HTTP 代码应包含：

```
#include <ngx_http.h>
```

Mail code should include

​	邮件代码应包含：

```
#include <ngx_mail.h>
```

Stream code should include

​	流代码应包含：

```
#include <ngx_stream.h>
```



### 整数 Integers

For general purposes, nginx code uses two integer types, `ngx_int_t` and `ngx_uint_t`, which are typedefs for `intptr_t` and `uintptr_t` respectively.

​	在一般情况下，nginx 代码使用两种整数类型，`ngx_int_t` 和 `ngx_uint_t`，它们分别是 `intptr_t` 和 `uintptr_t` 的类型定义。



### 常见返回码 Common return codes

Most functions in nginx return the following codes:

​	nginx 中的大多数函数返回以下代码：

- `NGX_OK` — Operation succeeded.
- `NGX_OK` — 操作成功。
- `NGX_ERROR` — Operation failed.
- `NGX_ERROR` — 操作失败。
- `NGX_AGAIN` — Operation incomplete; call the function again.
- `NGX_AGAIN` — 操作不完整；再次调用该函数。
- `NGX_DECLINED` — Operation rejected, for example, because it is disabled in the configuration. This is never an error.
- `NGX_DECLINED` — 操作被拒绝，例如，因为在配置中被禁用。这不是错误。
- `NGX_BUSY` — Resource is not available.
- `NGX_BUSY` — 资源不可用。
- `NGX_DONE` — Operation complete or continued elsewhere. Also used as an alternative success code.
- `NGX_DONE` — 操作已完成或在其他地方继续。也可用作替代的成功代码。
- `NGX_ABORT` — Function was aborted. Also used as an alternative error code.
- `NGX_ABORT` — 函数被中止。也可用作替代的错误代码。



### 错误处理 Error handling

The `ngx_errno` macro returns the last system error code. It's mapped to `errno` on POSIX platforms and to `GetLastError()` call in Windows. The `ngx_socket_errno` macro returns the last socket error number. Like the `ngx_errno` macro, it's mapped to `errno` on POSIX platforms. It's mapped to the `WSAGetLastError()` call on Windows. Accessing the values of `ngx_errno` or `ngx_socket_errno` more than once in a row can cause performance issues. If the error value might be used multiple times, store it in a local variable of type `ngx_err_t`. To set errors, use the `ngx_set_errno(errno)` and `ngx_set_socket_errno(errno)` macros.

​	宏 `ngx_errno` 返回最后一个系统错误代码。在 POSIX 平台上，它映射到 `errno`，在 Windows 上则映射到 `GetLastError()` 调用。宏 `ngx_socket_errno` 返回最后一个套接字错误编号。与 `ngx_errno` 宏一样，在 POSIX 平台上它映射到 `errno`。在 Windows 上映射到 `WSAGetLastError()` 调用。连续多次访问 `ngx_errno` 或 `ngx_socket_errno` 的值可能会导致性能问题。如果错误值可能被多次使用，将其存储在类型为 `ngx_err_t` 的本地变量中。要设置错误，使用宏 `ngx_set_errno(errno)` 和 `ngx_set_socket_errno(errno)`。

The values of `ngx_errno` and `ngx_socket_errno` can be passed to the logging functions `ngx_log_error()` and `ngx_log_debugX()`, in which case system error text is added to the log message.

​	`ngx_errno` 和 `ngx_socket_errno` 的值可以传递给日志函数 `ngx_log_error()` 和 `ngx_log_debugX()`，在这种情况下，系统错误文本将添加到日志消息中。

Example using `ngx_errno`:

​	以下是使用 `ngx_errno` 的示例：

```
ngx_int_t
ngx_my_kill(ngx_pid_t pid, ngx_log_t *log, int signo)
{
    ngx_err_t  err;

    if (kill(pid, signo) == -1) {
        err = ngx_errno;

        ngx_log_error(NGX_LOG_ALERT, log, err, "kill(%P, %d) failed", pid, signo);

        if (err == NGX_ESRCH) {
            return 2;
        }

        return 1;
    }

    return 0;
}
```



## 字符串 Strings



### 概述 Overview

For C strings, nginx uses the unsigned character type pointer `u_char *`.

​	对于 C 字符串，nginx 使用无符号字符类型指针 `u_char *`。

The nginx string type `ngx_str_t` is defined as follows:

​	nginx 字符串类型 `ngx_str_t` 定义如下：

```
typedef struct {
    size_t      len;
    u_char     *data;
} ngx_str_t;
```

The `len` field holds the string length and `data` holds the string data. The string, held in `ngx_str_t`, may or may not be null-terminated after the `len` bytes. In most cases it’s not. However, in certain parts of the code (for example, when parsing configuration), `ngx_str_t` objects are known to be null-terminated, which simplifies string comparison and makes it easier to pass the strings to syscalls.

​	`len` 字段保存字符串长度，`data` 保存字符串数据。在 `ngx_str_t` 中保存的字符串在 `len` 字节后可能有或可能没有以 null 终止。在大多数情况下，它不会有。然而，在代码的某些部分（例如解析配置时），`ngx_str_t` 对象被认为是以 null 终止的，这简化了字符串比较并使其更容易传递到系统调用。

The string operations in nginx are declared in `src/core/ngx_string.h` Some of them are wrappers around standard C functions:

​	nginx 中的字符串操作在 `src/core/ngx_string.h` 中声明。其中一些是标准 C 函数的包装：

- `ngx_strcmp()`
- `ngx_strncmp()`
- `ngx_strstr()`
- `ngx_strlen()`
- `ngx_strchr()`
- `ngx_memcmp()`
- `ngx_memset()`
- `ngx_memcpy()`
- `ngx_memmove()`



Other string functions are nginx-specific

​	其他字符串函数是 nginx 特有的：

- `ngx_memzero()` — Fills memory with zeroes.
- `ngx_memzero()` — 用零填充内存。
- `ngx_explicit_memzero()` — Does the same as `ngx_memzero()`, but this call is never removed by the compiler's dead store elimination optimization. This function can be used to clear sensitive data such as passwords and keys.
- `ngx_explicit_memzero()` — 与 `ngx_memzero()` 相同，但这个调用永远不会被编译器的无用存储消除优化删除。该函数可用于清除敏感数据，如密码和密钥。
- `ngx_cpymem()` — Does the same as `ngx_memcpy()`, but returns the final destination address This one is handy for appending multiple strings in a row.
- `ngx_cpymem()` — 与 `ngx_memcpy()` 相同，但返回最终的目标地址。对于连续追加多个字符串非常方便。
- `ngx_movemem()` — Does the same as `ngx_memmove()`, but returns the final destination address.
- `ngx_movemem()` — 与 `ngx_memmove()` 相同，但返回最终的目标地址。
- `ngx_strlchr()` — Searches for a character in a string, delimited by two pointers.
- `ngx_strlchr()` — 在两个指针之间的字符串中搜索字符。



The following functions perform case conversion and comparison:

​	以下函数执行大小写转换和比较：

- `ngx_tolower()`
- `ngx_toupper()`
- `ngx_strlow()`
- `ngx_strcasecmp()`
- `ngx_strncasecmp()`



The following macros simplify string initialization:

​	以下宏简化字符串初始化：

- `ngx_string(text)` — static initializer for the `ngx_str_t` type from the C string literal `text`
- `ngx_string(text)` — 使用 C 字符串字面量 `text` 对 `ngx_str_t` 类型进行静态初始化
- `ngx_null_string` — static empty string initializer for the `ngx_str_t` type
- `ngx_null_string` — 使用 C 字符串字面量初始化 `ngx_str_t` 类型的静态空字符串
- `ngx_str_set(str, text)` — initializes string `str` of `ngx_str_t *` type with the C string literal `text`
- `ngx_str_set(str, text)` — 使用 C 字符串字面量 `text` 初始化 `ngx_str_t *` 类型的字符串 `str`
- `ngx_str_null(str)` — initializes string `str` of `ngx_str_t *` type with the empty string
- `ngx_str_null(str)` — 使用空字符串初始化 `ngx_str_t *` 类型的字符串 `str`



### 格式化 Formatting

The following formatting functions support nginx-specific types:

​	以下格式化函数支持 nginx 特定的类型：

- `ngx_sprintf(buf, fmt, ...)`
- `ngx_snprintf(buf, max, fmt, ...)`
- `ngx_slprintf(buf, last, fmt, ...)`
- `ngx_vslprintf(buf, last, fmt, args)`
- `ngx_vsnprintf(buf, max, fmt, args)`



The full list of formatting options, supported by these functions is in `src/core/ngx_string.c`. Some of them are:

​	这些函数支持的完整格式化选项列表在 `src/core/ngx_string.c` 中。其中一些是：

- `%O` — `off_t`
- `%T` — `time_t`
- `%z` — `ssize_t`
- `%i` — `ngx_int_t`
- `%p` — `void *`
- `%V` — `ngx_str_t *`
- `%s` — `u_char *` (null-terminated) （以 null 终止）
- `%*s` — `size_t + u_char *`

You can prepend `u` on most types to make them unsigned. To convert output to hex, use `X` or `x`.

​	您可以在大多数类型前面添加 `u` 来使它们成为无符号的。要将输出转换为十六进制，请使用 `X` 或 `x`。

For example:

​	例如：

```
u_char      buf[NGX_INT_T_LEN];
size_t      len;
ngx_uint_t  n;

/* set n here */

len = ngx_sprintf(buf, "%ui", n) — buf;
```





### 数值转换 Numeric conversion

Several functions for numeric conversion are implemented in nginx. The first four each convert a string of given length to a positive integer of the indicated type. They return `NGX_ERROR` on error.

​	nginx 中实现了几个用于数值转换的函数。前四个函数分别将给定长度的字符串转换为所示类型的正整数。出现错误时返回 `NGX_ERROR`。 

- `ngx_atoi(line, n)` — `ngx_int_t`
- `ngx_atosz(line, n)` — `ssize_t`
- `ngx_atoof(line, n)` — `off_t`
- `ngx_atotm(line, n)` — `time_t`



There are two additional numeric conversion functions. Like the first four, they return `NGX_ERROR` on error.

​	还有两个额外的数值转换函数。与前四个函数一样，出现错误时返回 `NGX_ERROR`。

- `ngx_atofp(line, n, point)` — Converts a fixed point floating number of given length to a positive integer of type `ngx_int_t`. The result is shifted left by `point` decimal positions. The string representation of the number is expected to have no more than `points` fractional digits. For example, `ngx_atofp("10.5", 4, 2)` returns `1050`.
- `ngx_atofp(line, n, point)` — 将给定长度的定点浮点数转换为类型为 `ngx_int_t` 的正整数。结果向左移动 `point` 个小数位。预期数字的字符串表示不应超过 `points` 个小数位。例如，`ngx_atofp("10.5", 4, 2)` 返回 `1050`。
- `ngx_hextoi(line, n)` — Converts a hexadecimal representation of a positive integer to `ngx_int_t`.
- `ngx_hextoi(line, n)` — 将十六进制表示的正整数转换为 `ngx_int_t`。





### 正则表达式 Regular expressions

The regular expressions interface in nginx is a wrapper around the [PCRE](http://www.pcre.org/) library. The corresponding header file is `src/core/ngx_regex.h`.

​	nginx 中的正则表达式接口是 [PCRE](http://www.pcre.org/) 库的包装。对应的头文件是 `src/core/ngx_regex.h`。

To use a regular expression for string matching, it first needs to be compiled, which is usually done at the configuration phase. Note that since PCRE support is optional, all code using the interface must be protected by the surrounding `NGX_PCRE` macro:

​	要将正则表达式用于字符串匹配，首先需要编译它，这通常在配置阶段完成。请注意，由于 PCRE 支持是可选的，所有使用该接口的代码都必须受到周围的 `NGX_PCRE` 宏的保护：

```
#if (NGX_PCRE)
ngx_regex_t          *re;
ngx_regex_compile_t   rc;

u_char                errstr[NGX_MAX_CONF_ERRSTR];

ngx_str_t  value = ngx_string("message (\\d\\d\\d).*Codeword is '(?<cw>\\w+)'");

ngx_memzero(&rc, sizeof(ngx_regex_compile_t));

rc.pattern = value;
rc.pool = cf->pool;
rc.err.len = NGX_MAX_CONF_ERRSTR;
rc.err.data = errstr;
/* rc.options can be set to NGX_REGEX_CASELESS */

if (ngx_regex_compile(&rc) != NGX_OK) {
    ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "%V", &rc.err);
    return NGX_CONF_ERROR;
}

re = rc.regex;
#endif
```

After successful compilation, the `captures` and `named_captures` fields in the `ngx_regex_compile_t` structure contain the count of all captures and named captures, respectively, found in the regular expression.

​	编译成功后，`ngx_regex_compile_t` 结构中的 `captures` 和 `named_captures` 字段分别包含正则表达式中找到的所有捕获和命名捕获的数量。

The compiled regular expression can then be used for matching against strings:

​	然后可以使用已编译的正则表达式来与字符串进行匹配：

```
ngx_int_t  n;
int        captures[(1 + rc.captures) * 3];

ngx_str_t input = ngx_string("This is message 123. Codeword is 'foobar'.");

n = ngx_regex_exec(re, &input, captures, (1 + rc.captures) * 3);
if (n >= 0) {
    /* string matches expression */
    /* 字符串匹配表达式 */

} else if (n == NGX_REGEX_NO_MATCHED) {
    /* no match was found */
    /* 未找到匹配项 */

} else {
    /* some error */
    /* 一些错误 */
    ngx_log_error(NGX_LOG_ALERT, log, 0, ngx_regex_exec_n " failed: %i", n);
}
```

The arguments to `ngx_regex_exec()` are the compiled regular expression `re`, the string to match `input`, an optional array of integers to hold any `captures` that are found, and the array's `size`. The size of the `captures` array must be a multiple of three, as required by the [PCRE API](http://www.pcre.org/original/doc/html/pcreapi.html). In the example, the size is calculated from the total number of captures plus one for the matched string itself.

​	`ngx_regex_exec()` 函数的参数是已编译的正则表达式 `re`、要匹配的字符串 `input`、一个可选的整数数组以保存找到的任何 `captures`，以及数组的 `size`。`captures` 数组的大小必须是三的倍数，正如 [PCRE API](http://www.pcre.org/original/doc/html/pcreapi.html) 所要求的。在示例中，大小是根据总捕获数加上用于匹配的字符串本身的大小来计算的。

If there are matches, captures can be accessed as follows:

​	如果有匹配项，可以通过以下方式访问捕获项：

```
u_char     *p;
size_t      size;
ngx_str_t   name, value;

/* all captures */
/* 所有捕获项 */
for (i = 0; i < n * 2; i += 2) {
    value.data = input.data + captures[i];
    value.len = captures[i + 1] — captures[i];
}

/* accessing named captures */
/* 访问命名捕获项 */

size = rc.name_size;
p = rc.names;

for (i = 0; i < rc.named_captures; i++, p += size) {

    /* capture name */
    /* 捕获项名称 */
    name.data = &p[2];
    name.len = ngx_strlen(name.data);

    n = 2 * ((p[0] << 8) + p[1]);

    /* captured value */
    /* 捕获项的值 */
    value.data = &input.data[captures[n]];
    value.len = captures[n + 1] — captures[n];
}
```



The `ngx_regex_exec_array()` function accepts the array of `ngx_regex_elt_t` elements (which are just compiled regular expressions with associated names), a string to match, and a log. The function applies expressions from the array to the string until either a match is found or no more expressions are left. The return value is `NGX_OK` when there is a match and `NGX_DECLINED` otherwise, or `NGX_ERROR` in case of error.

​	`ngx_regex_exec_array()` 函数接受 `ngx_regex_elt_t` 元素（它们只是带有相关名称的已编译正则表达式）的数组、要匹配的字符串和日志。该函数将数组中的表达式应用于字符串，直到找到匹配项或不再剩下表达式。如果有匹配项，则返回值为 `NGX_OK`，否则为 `NGX_DECLINED`，或在出现错误时为 `NGX_ERROR`。



## 时间 Time

The `ngx_time_t` structure represents time with three separate types for seconds, milliseconds, and the GMT offset:

​	`ngx_time_t` 结构以秒、毫秒和 GMT 偏移的三种不同类型表示时间：

```
typedef struct {
    time_t      sec;
    ngx_uint_t  msec;
    ngx_int_t   gmtoff;
} ngx_time_t;
```

The `ngx_tm_t` structure is an alias for `struct tm` on UNIX platforms and `SYSTEMTIME` on Windows.

​	`ngx_tm_t` 结构是 UNIX 平台上 `struct tm` 的别名，在 Windows 上是 `SYSTEMTIME`。

To obtain the current time, it is usually sufficient to access one of the available global variables, representing the cached time value in the desired format.

​	要获取当前时间，通常可以访问其中一个可用的全局变量，表示所需格式的缓存时间值。

The available string representations are:

​	可用的字符串表示如下：

- `ngx_cached_err_log_time` — Used in error log entries: `"1970/09/28 12:00:00"`
- `ngx_cached_err_log_time` — 用于错误日志条目：`"1970/09/28 12:00:00"`
- `ngx_cached_http_log_time` — Used in HTTP access log entries: `"28/Sep/1970:12:00:00 +0600"`
- `ngx_cached_http_log_time` — 用于 HTTP 访问日志条目："28/Sep/1970:12:00:00 +0600"
- `ngx_cached_syslog_time` — Used in syslog entries: `"Sep 28 12:00:00"`
- `ngx_cached_syslog_time` — 用于 syslog 条目：`"Sep 28 12:00:00"`
- `ngx_cached_http_time` — Used in HTTP headers: `"Mon, 28 Sep 1970 06:00:00 GMT"`
- `ngx_cached_http_time` — 用于 HTTP 标头：`"Mon, 28 Sep 1970 06:00:00 GMT"`
- `ngx_cached_http_log_iso8601` — The ISO 8601 standard format: `"1970-09-28T12:00:00+06:00"`
- `ngx_cached_http_log_iso8601` — ISO 8601 标准格式：`"1970-09-28T12:00:00+06:00"`

The `ngx_time()` and `ngx_timeofday()` macros return the current time value in seconds and are the preferred way to access the cached time value.

​	宏 `ngx_time()` 和 `ngx_timeofday()` 返回以秒为单位的当前时间值，是访问缓存时间值的首选方法。

To obtain the time explicitly, use `ngx_gettimeofday()`, which updates its argument (pointer to `struct timeval`). The time is always updated when nginx returns to the event loop from system calls. To update the time immediately, call `ngx_time_update()`, or `ngx_time_sigsafe_update()` if updating the time in the signal handler context.

​	要显式获取时间，使用 `ngx_gettimeofday()`，它会更新其参数（指向 `struct timeval` 的指针）。当 nginx 从系统调用返回到事件循环时，始终会更新时间。要立即更新时间，请调用 `ngx_time_update()` 或 `ngx_time_sigsafe_update()`（如果在信号处理程序上下文中更新时间）。

The following functions convert `time_t` into the indicated broken-down time representation. The first function in each pair converts `time_t` to `ngx_tm_t` and the second (with the `_libc_` infix) to `struct tm`:

​	以下函数将 `time_t` 转换为所示的分解时间表示。每对中的第一个函数将 `time_t` 转换为 `ngx_tm_t`，第二个函数（带 `_libc_` 中缀）将其转换为 `struct tm`：

 

- `ngx_gmtime(), ngx_libc_gmtime()` — Time expressed as UTC
- `ngx_gmtime(), ngx_libc_gmtime()` — 表示为 UTC 的时间
- `ngx_localtime(), ngx_libc_localtime()` — Time expressed relative to the local time zone
- `ngx_localtime(), ngx_libc_localtime()` — 相对于本地时区表示的时间

The `ngx_http_time(buf, time)` function returns a string representation suitable for use in HTTP headers (for example, `"Mon, 28 Sep 1970 06:00:00 GMT"`). The `ngx_http_cookie_time(buf, time)` returns a string representation function returns a string representation suitable for HTTP cookies (`"Thu, 31-Dec-37 23:55:55 GMT"`).

​	`ngx_http_time(buf, time)` 函数返回适用于 HTTP 标头的字符串表示（例如 `"Mon, 28 Sep 1970 06:00:00 GMT"`）。`ngx_http_cookie_time(buf, time)` 函数返回适用于 HTTP cookie 的字符串表示（`"Thu, 31-Dec-37 23:55:55 GMT"`）。

## 容器 Containers



### 数组 Array

The nginx array type `ngx_array_t` is defined as follows

​	nginx 的数组类型 `ngx_array_t` 定义如下：

```
typedef struct {
    void        *elts;
    ngx_uint_t   nelts;
    size_t       size;
    ngx_uint_t   nalloc;
    ngx_pool_t  *pool;
} ngx_array_t;
```

The elements of the array are available in the `elts` field. The `nelts` field holds the number of elements. The `size` field holds the size of a single element and is set when the array is initialized.

​	数组的元素位于 `elts` 字段中。`nelts` 字段保存元素数量。`size` 字段保存单个元素的大小，并在数组初始化时设置。

Use the `ngx_array_create(pool, n, size)` call to create an array in a pool, and the `ngx_array_init(array, pool, n, size)` call to initialize an array object that has already been allocated.

​	使用 `ngx_array_create(pool, n, size)` 调用在内存池中创建一个数组，使用 `ngx_array_init(array, pool, n, size)` 调用来初始化已经分配的数组对象。

```
ngx_array_t  *a, b;

/* create an array of strings with preallocated memory for 10 elements */
/* 在预分配了 10 个元素的内存中创建一个字符串数组 */
a = ngx_array_create(pool, 10, sizeof(ngx_str_t));

/* initialize string array for 10 elements */
/* 初始化一个预分配了 10 个元素的字符串数组 */
ngx_array_init(&b, pool, 10, sizeof(ngx_str_t));
```

Use the following functions to add elements to an array:

​	使用以下函数将元素添加到数组中：

- `ngx_array_push(a)` adds one tail element and returns pointer to it
- `ngx_array_push(a)` 添加一个尾部元素并返回指向它的指针
- `ngx_array_push_n(a, n)` adds `n` tail elements and returns pointer to the first one
- `ngx_array_push_n(a, n)` 添加 `n` 个尾部元素并返回指向第一个元素的指针



If the currently allocated amount of memory is not large enough to accommodate the new elements, a new block of memory is allocated and the existing elements are copied to it. The new memory block is normally twice as large as the existing one.

​	如果当前分配的内存量不足以容纳新元素，将分配一个新的内存块，并将现有元素复制到其中。新的内存块通常是现有内存块的两倍大。

```
s = ngx_array_push(a);
ss = ngx_array_push_n(&b, 3);
```



### 列表 List

In nginx a list is a sequence of arrays, optimized for inserting a potentially large number of items. The `ngx_list_t` list type is defined as follows:

​	在 nginx 中，列表是用于插入可能大量项的序列的数组序列，`ngx_list_t` 列表类型定义如下：

```
typedef struct {
    ngx_list_part_t  *last;
    ngx_list_part_t   part;
    size_t            size;
    ngx_uint_t        nalloc;
    ngx_pool_t       *pool;
} ngx_list_t;
```

The actual items are stored in list parts, which are defined as follows:

​	实际项存储在列表部分中，列表部分定义如下：

```
typedef struct ngx_list_part_s  ngx_list_part_t;

struct ngx_list_part_s {
    void             *elts;
    ngx_uint_t        nelts;
    ngx_list_part_t  *next;
};
```

Before use, a list must be initialized by calling `ngx_list_init(list, pool, n, size)` or created by calling `ngx_list_create(pool, n, size)`. Both functions take as arguments the size of a single item and a number of items per list part. To add an item to a list, use the `ngx_list_push(list)` function. To iterate over the items, directly access the list fields as shown in the example:

​	在使用之前，必须通过调用 `ngx_list_init(list, pool, n, size)` 来初始化列表，或通过调用 `ngx_list_create(pool, n, size)` 来创建列表。这两个函数都需要单个项的大小和每个列表部分的项数作为参数。要将项添加到列表中，请使用 `ngx_list_push(list)` 函数。要遍历项，请直接访问列表字段，如下例所示：

```
ngx_str_t        *v;
ngx_uint_t        i;
ngx_list_t       *list;
ngx_list_part_t  *part;

list = ngx_list_create(pool, 100, sizeof(ngx_str_t));
if (list == NULL) { /* error */ /* 错误处理 */ }

/* add items to the list */
/* 向列表添加项 */

v = ngx_list_push(list);
if (v == NULL) { /* error */ /* 错误处理 */ }
ngx_str_set(v, "foo");

v = ngx_list_push(list);
if (v == NULL) { /* error */ /* 错误处理 */ }
ngx_str_set(v, "bar");

/* iterate over the list */
/* 遍历列表 */

part = &list->part;
v = part->elts;

for (i = 0; /* void */; i++) {

    if (i >= part->nelts) {
        if (part->next == NULL) {
            break;
        }

        part = part->next;
        v = part->elts;
        i = 0;
    }

    ngx_do_smth(&v[i]);
}
```

Lists are primarily used for HTTP input and output headers.

​	列表主要用于 HTTP 输入和输出头。

Lists do not support item removal. However, when needed, items can internally be marked as missing without actually being removed from the list. For example, to mark HTTP output headers (which are stored as `ngx_table_elt_t` objects) as missing, set the `hash` field in `ngx_table_elt_t` to zero. Items marked in this way are explicitly skipped when the headers are iterated over.

​	列表不支持删除项。但是，当需要时，可以将项内部标记为丢失，而实际上并未从列表中删除。例如，要将 HTTP 输出头（存储为 `ngx_table_elt_t` 对象）标记为丢失，请将 `ngx_table_elt_t` 中的 `hash` 字段设置为零。以这种方式标记的项在迭代标题时会被显式跳过。



### 队列 Queue

In nginx a queue is an intrusive doubly linked list, with each node defined as follows:

​	在 nginx 中，队列是一种侵入式的双向链表，每个节点的定义如下：

```
typedef struct ngx_queue_s  ngx_queue_t;

struct ngx_queue_s {
    ngx_queue_t  *prev;
    ngx_queue_t  *next;
};
```

The head queue node is not linked with any data. Use the `ngx_queue_init(q)` call to initialize the list head before use. Queues support the following operations:

​	头节点不与任何数据链接。在使用前，请使用 `ngx_queue_init(q)` 调用初始化链表头。队列支持以下操作：

- `ngx_queue_insert_head(h, x)`, `ngx_queue_insert_tail(h, x)` — Insert a new node
- `ngx_queue_insert_head(h, x)`、`ngx_queue_insert_tail(h, x)` — 插入一个新节点
- `ngx_queue_remove(x)` — Remove a queue node
- `ngx_queue_remove(x)` — 删除一个队列节点
- `ngx_queue_split(h, q, n)` — Split a queue at a node, returning the queue tail in a separate queue
- `ngx_queue_split(h, q, n)` — 在节点处拆分队列，并将队列尾部返回到单独的队列
- `ngx_queue_add(h, n)` — Add a second queue to the first queue
- `ngx_queue_add(h, n)` — 将第二个队列添加到第一个队列
- `ngx_queue_head(h)`, `ngx_queue_last(h)` — Get first or last queue node
- `ngx_queue_head(h)`、`ngx_queue_last(h)` — 获取第一个或最后一个队列节点
- `ngx_queue_sentinel(h)` - Get a queue sentinel object to end iteration at
- `ngx_queue_sentinel(h)` - 获取用于终止迭代的队列 sentinel 对象
- `ngx_queue_data(q, type, link)` — Get a reference to the beginning of a queue node data structure, considering the queue field offset in it
- `ngx_queue_data(q, type, link)` — 获取队列节点数据结构开头的引用，考虑其中的队列字段偏移量



An example:

​	示例：

```
typedef struct {
    ngx_str_t    value;
    ngx_queue_t  queue;
} ngx_foo_t;

ngx_foo_t    *f;
ngx_queue_t   values, *q;

ngx_queue_init(&values);

f = ngx_palloc(pool, sizeof(ngx_foo_t));
if (f == NULL) { /* error */ /* 错误处理 */ }
ngx_str_set(&f->value, "foo");

ngx_queue_insert_tail(&values, &f->queue);

/* insert more nodes here */
/* 在此处插入更多节点 */

for (q = ngx_queue_head(&values);
     q != ngx_queue_sentinel(&values);
     q = ngx_queue_next(q))
{
    f = ngx_queue_data(q, ngx_foo_t, queue);

    ngx_do_smth(&f->value);
}
```



### 红黑树 Red-Black tree

The `src/core/ngx_rbtree.h` header file provides access to the effective implementation of red-black trees.

​	头文件 `src/core/ngx_rbtree.h` 提供了对红黑树有效实现的访问。

```
typedef struct {
    ngx_rbtree_t       rbtree;
    ngx_rbtree_node_t  sentinel;

    /* custom per-tree data here */
    /* 此处为每个树的自定义数据 */
} my_tree_t;

typedef struct {
    ngx_rbtree_node_t  rbnode;

    /* custom per-node data */
    /* 自定义的每个节点数据 */
    foo_t              val;
} my_node_t;
```

To deal with a tree as a whole, you need two nodes: root and sentinel. Typically, they are added to a custom structure, allowing you to organize your data into a tree in which the leaves contain a link to or embed your data.

​	要处理整个树，需要两个节点：根节点和哨兵节点。通常，它们被添加到自定义结构中，使您可以将数据组织成一个树，其中叶子包含链接到或嵌入您的数据的指针。

To initialize a tree:

​	要初始化树：

```
my_tree_t  root;

ngx_rbtree_init(&root.rbtree, &root.sentinel, insert_value_function);
```

To traverse a tree and insert new values, use the "`insert_value`" functions. For example, the `ngx_str_rbtree_insert_value` function deals with the `ngx_str_t` type. Its arguments are pointers to a root node of an insertion, the newly created node to be added, and a tree sentinel.

​	要遍历树并插入新值，使用“`insert_value`”函数。例如，`ngx_str_rbtree_insert_value` 函数处理 `ngx_str_t` 类型。它的参数是插入的根节点、要添加的新创建节点以及树的哨兵。

```
void ngx_str_rbtree_insert_value(ngx_rbtree_node_t *temp,
                                 ngx_rbtree_node_t *node,
                                 ngx_rbtree_node_t *sentinel)
```

The traversal is pretty straightforward and can be demonstrated with the following lookup function pattern:

​	遍历非常简单，可以用以下查找函数模式来演示：

```
my_node_t *
my_rbtree_lookup(ngx_rbtree_t *rbtree, foo_t *val, uint32_t hash)
{
    ngx_int_t           rc;
    my_node_t          *n;
    ngx_rbtree_node_t  *node, *sentinel;

    node = rbtree->root;
    sentinel = rbtree->sentinel;

    while (node != sentinel) {

        n = (my_node_t *) node;

        if (hash != node->key) {
            node = (hash < node->key) ? node->left : node->right;
            continue;
        }

        rc = compare(val, node->val);

        if (rc < 0) {
            node = node->left;
            continue;
        }

        if (rc > 0) {
            node = node->right;
            continue;
        }

        return n;
    }

    return NULL;
}
```

The `compare()` function is a classic comparator function that returns a value less than, equal to, or greater than zero. To speed up lookups and avoid comparing user objects that can be big, an integer hash field is used.

​	`compare()` 函数是一个经典的比较器函数，返回小于零、等于零或大于零的值。为了加速查找并避免比较可能很大的用户对象，使用了整数哈希字段。

To add a node to a tree, allocate a new node, initialize it and call `ngx_rbtree_insert()`:

​	要向树中添加节点，请分配一个新节点，对其进行初始化并调用 `ngx_rbtree_insert()`：

```
    my_node_t          *my_node;
    ngx_rbtree_node_t  *node;

    my_node = ngx_palloc(...);
    init_custom_data(&my_node->val);

    node = &my_node->rbnode;
    node->key = create_key(my_node->val);

    ngx_rbtree_insert(&root->rbtree, node);
```

To remove a node, call the `ngx_rbtree_delete()` function:

​	要删除节点，请调用 `ngx_rbtree_delete()` 函数：

```
ngx_rbtree_delete(&root->rbtree, node);
```



### 哈希表 Hash

Hash table functions are declared in `src/core/ngx_hash.h`. Both exact and wildcard matching are supported. The latter requires extra setup and is described in a separate section below.

​	哈希表函数在 `src/core/ngx_hash.h` 中声明。支持精确匹配和通配符匹配。后者需要额外的设置，在下面的独立部分中进行了描述。

Before initializing a hash, you need to know the number of elements it will hold so that nginx can build it optimally. Two parameters that need to be configured are `max_size` and `bucket_size`, as detailed in a separate [document](https://nginx.org/en/docs/hash.html). They are usually configurable by the user. Hash initialization settings are stored with the `ngx_hash_init_t` type, and the hash itself is `ngx_hash_t`:

​	在初始化哈希之前，您需要知道它将容纳的元素数量，以便 nginx 可以对其进行最佳优化。需要配置的两个参数是 `max_size` 和 `bucket_size`，如单独的[文档](https://nginx.org/en/docs/hash.html)中所述。它们通常可以由用户配置。哈希初始化设置存储在 `ngx_hash_init_t` 类型中，哈希本身是 `ngx_hash_t`：

```
ngx_hash_t       foo_hash;
ngx_hash_init_t  hash;

hash.hash = &foo_hash;
hash.key = ngx_hash_key;
hash.max_size = 512;
hash.bucket_size = ngx_align(64, ngx_cacheline_size);
hash.name = "foo_hash";
hash.pool = cf->pool;
hash.temp_pool = cf->temp_pool;
```

The `key` is a pointer to a function that creates the hash integer key from a string. There are two generic key-creation functions: `ngx_hash_key(data, len)` and `ngx_hash_key_lc(data, len)`. The latter converts a string to all lowercase characters, so the passed string must be writable. If that is not true, pass the `NGX_HASH_READONLY_KEY` flag to the function, initializing the key array (see below).

​	`key` 是指向从字符串创建哈希整数键的函数的指针。有两个通用的键创建函数：`ngx_hash_key(data, len)` 和 `ngx_hash_key_lc(data, len)`。后者将字符串转换为所有小写字符，因此传递的字符串必须是可写的。如果不是这样，请将 `NGX_HASH_READONLY_KEY` 标志传递给函数，初始化键数组（见下文）。

The hash keys are stored in `ngx_hash_keys_arrays_t` and are initialized with `ngx_hash_keys_array_init(arr, type)`: The second parameter (`type`) controls the amount of resources preallocated for the hash and can be either `NGX_HASH_SMALL` or `NGX_HASH_LARGE`. The latter is appropriate if you expect the hash to contain thousands of elements.

​	哈希键存储在 `ngx_hash_keys_arrays_t` 中，并使用 `ngx_hash_keys_array_init(arr, type)` 进行初始化：第二个参数（`type`）控制为哈希预分配的资源量，可以是 `NGX_HASH_SMALL` 或 `NGX_HASH_LARGE`。如果希望哈希包含数千个元素，后者是适当的。

```
ngx_hash_keys_arrays_t  foo_keys;

foo_keys.pool = cf->pool;
foo_keys.temp_pool = cf->temp_pool;

ngx_hash_keys_array_init(&foo_keys, NGX_HASH_SMALL);
```



To insert keys into a hash keys array, use the `ngx_hash_add_key(keys_array, key, value, flags)` function:

​	要将键插入哈希键数组中，请使用 `ngx_hash_add_key(keys_array, key, value, flags)` 函数：

```
ngx_str_t k1 = ngx_string("key1");
ngx_str_t k2 = ngx_string("key2");

ngx_hash_add_key(&foo_keys, &k1, &my_data_ptr_1, NGX_HASH_READONLY_KEY);
ngx_hash_add_key(&foo_keys, &k2, &my_data_ptr_2, NGX_HASH_READONLY_KEY);
```



To build the hash table, call the `ngx_hash_init(hinit, key_names, nelts)` function:

​	要构建哈希表，请调用 `ngx_hash_init(hinit, key_names, nelts)` 函数：

```
ngx_hash_init(&hash, foo_keys.keys.elts, foo_keys.keys.nelts);
```

The function fails if `max_size` or `bucket_size` parameters are not big enough.

When the hash is built, use the `ngx_hash_find(hash, key, name, len)` function to look up elements:

​	如果 `max_size` 或 `bucket_size` 参数不足够大，则函数将失败。哈希构建完毕后，使用 `ngx_hash_find(hash, key, name, len)` 函数查找元素：

```
my_data_t   *data;
ngx_uint_t   key;

key = ngx_hash_key(k1.data, k1.len);

data = ngx_hash_find(&foo_hash, key, k1.data, k1.len);
if (data == NULL) {
    /* key not found */
    /* 未找到键 */
}
```





## 通配符匹配 Wildcard matching

To create a hash that works with wildcards, use the `ngx_hash_combined_t` type. It includes the hash type described above and has two additional keys arrays: `dns_wc_head` and `dns_wc_tail`. The initialization of basic properties is similar to a regular hash:

​	要创建支持通配符的哈希表，可以使用 `ngx_hash_combined_t` 类型。它包括上面描述的哈希类型，并且还有两个额外的键数组：`dns_wc_head` 和 `dns_wc_tail`。基本属性的初始化类似于常规的哈希表：

```
ngx_hash_init_t      hash
ngx_hash_combined_t  foo_hash;

hash.hash = &foo_hash.hash;
hash.key = ...;
```



It is possible to add wildcard keys using the `NGX_HASH_WILDCARD_KEY` flag:

​	可以使用 `NGX_HASH_WILDCARD_KEY` 标志来添加通配符键：

```
/* k1 = ".example.org"; */
/* k2 = "foo.*";        */
ngx_hash_add_key(&foo_keys, &k1, &data1, NGX_HASH_WILDCARD_KEY);
ngx_hash_add_key(&foo_keys, &k2, &data2, NGX_HASH_WILDCARD_KEY);
```

The function recognizes wildcards and adds keys into the corresponding arrays. Please refer to the [map](https://nginx.org/en/docs/http/ngx_http_map_module.html#map) module documentation for the description of the wildcard syntax and the matching algorithm.

​	这个函数可以识别通配符并将键添加到相应的数组中。有关通配符语法和匹配算法的详细描述，请参阅 [map](https://nginx.org/en/docs/http/ngx_http_map_module.html#map) 模块的文档。

Depending on the contents of added keys, you may need to initialize up to three key arrays: one for exact matching (described above), and two more to enable matching starting from the head or tail of a string:

​	根据添加的键的内容，您可能需要初始化多达三个键数组：一个用于精确匹配（如上所述），另外两个用于从字符串的头部或尾部开始匹配：

```
if (foo_keys.dns_wc_head.nelts) {

    ngx_qsort(foo_keys.dns_wc_head.elts,
              (size_t) foo_keys.dns_wc_head.nelts,
              sizeof(ngx_hash_key_t),
              cmp_dns_wildcards);

    hash.hash = NULL;
    hash.temp_pool = pool;

    if (ngx_hash_wildcard_init(&hash, foo_keys.dns_wc_head.elts,
                               foo_keys.dns_wc_head.nelts)
        != NGX_OK)
    {
        return NGX_ERROR;
    }

    foo_hash.wc_head = (ngx_hash_wildcard_t *) hash.hash;
}
```

The keys array needs to be sorted, and initialization results must be added to the combined hash. The initialization of `dns_wc_tail` array is done similarly.

​	键数组需要进行排序，并且初始化的结果必须添加到组合哈希表中。`dns_wc_tail` 数组的初始化类似。

The lookup in a combined hash is handled by the `ngx_hash_find_combined(chash, key, name, len)`:

​	在组合哈希表中查找由 `ngx_hash_find_combined(chash, key, name, len)` 处理：

```
/* key = "bar.example.org"; — will match ".example.org" */
/* key = "foo.example.com"; — will match "foo.*"        */

hkey = ngx_hash_key(key.data, key.len);
res = ngx_hash_find_combined(&foo_hash, hkey, key.data, key.len);
```





## 内存管理 Memory management



### 堆内存 Heap

To allocate memory from system heap, use the following functions:

​	要从系统堆中分配内存，可以使用以下函数：

- `ngx_alloc(size, log)` — Allocate memory from system heap. This is a wrapper around `malloc()` with logging support. Allocation error and debugging information is logged to `log`.
- `ngx_alloc(size, log)` — 从系统堆中分配内存。这是对 `malloc()` 的封装，支持日志记录。分配错误和调试信息会记录到 `log`。
- `ngx_calloc(size, log)` — Allocate memory from system heap like `ngx_alloc()`, but fill memory with zeros after allocation.
- `ngx_calloc(size, log)` — 类似于 `ngx_alloc()` 从系统堆中分配内存，但分配后会将内存填充为零。
- `ngx_memalign(alignment, size, log)` — Allocate aligned memory from system heap. This is a wrapper around `posix_memalign()` on those platforms that provide that function. Otherwise implementation falls back to `ngx_alloc()` which provides maximum alignment.
- `ngx_memalign(alignment, size, log)` — 从系统堆中分配对齐的内存。在那些提供了 `posix_memalign()` 函数的平台上，这是对该函数的封装。否则，实现会退回到提供最大对齐的 `ngx_alloc()`。
- `ngx_free(p)` — Free allocated memory. This is a wrapper around `free()`
- `ngx_free(p)` — 释放先前分配的内存。这是对 `free()` 的封装。





### 内存池 Pool

Most nginx allocations are done in pools. Memory allocated in an nginx pool is freed automatically when the pool is destroyed. This provides good allocation performance and makes memory control easy.

​	大多数 nginx 的分配都是在内存池中进行的。在 nginx 内存池中分配的内存在销毁内存池时会自动释放。这提供了良好的分配性能，并使内存控制变得简单。

A pool internally allocates objects in continuous blocks of memory. Once a block is full, a new one is allocated and added to the pool memory block list. When the requested allocation is too large to fit into a block, the request is forwarded to the system allocator and the returned pointer is stored in the pool for further deallocation.

​	内存池内部以连续的内存块分配对象。一旦一个块已满，就会分配一个新块并将其添加到内存池的内存块列表中。当所请求的分配过大以至于无法适应一个块时，请求将转发到系统分配器，并将返回的指针存储在内存池中以供进一步释放使用。

The type for nginx pools is `ngx_pool_t`. The following operations are supported:

​	nginx 内存池的类型是 `ngx_pool_t`。支持以下操作： 

- `ngx_create_pool(size, log)` — Create a pool with specified block size. The pool object returned is allocated in the pool as well. The `size` should be at least `NGX_MIN_POOL_SIZE` and a multiple of `NGX_POOL_ALIGNMENT`.
- `ngx_create_pool(size, log)` — 创建一个具有指定块大小的内存池。返回的内存池对象也在内存池中分配。`size` 应该至少为 `NGX_MIN_POOL_SIZE`，且是 `NGX_POOL_ALIGNMENT` 的倍数。
- `ngx_destroy_pool(pool)` — Free all pool memory, including the pool object itself.
- `ngx_destroy_pool(pool)` — 释放内存池中的所有内存，包括内存池对象本身。
- `ngx_palloc(pool, size)` — Allocate aligned memory from the specified pool.
- `ngx_palloc(pool, size)` — 从指定的内存池中分配对齐的内存。
- `ngx_pcalloc(pool, size)` — Allocate aligned memory from the specified pool and fill it with zeroes.
- `ngx_pcalloc(pool, size)` — 从指定的内存池中分配对齐的内存，并将其填充为零。
- `ngx_pnalloc(pool, size)` — Allocate unaligned memory from the specified pool. Mostly used for allocating strings.
- `ngx_pnalloc(pool, size)` — 从指定的内存池中分配不对齐的内存。通常用于分配字符串。
- `ngx_pfree(pool, p)` — Free memory that was previously allocated in the specified pool. Only allocations that result from requests forwarded to the system allocator can be freed.
- `ngx_pfree(pool, p)` — 释放先前在指定的内存池中分配的内存。只能释放由请求转发到系统分配器所产生的分配。



```
u_char      *p;
ngx_str_t   *s;
ngx_pool_t  *pool;

pool = ngx_create_pool(1024, log);
if (pool == NULL) { /* error */ }

s = ngx_palloc(pool, sizeof(ngx_str_t));
if (s == NULL) { /* error */ }
ngx_str_set(s, "foo");

p = ngx_pnalloc(pool, 3);
if (p == NULL) { /* error */ }
ngx_memcpy(p, "foo", 3);
```

Chain links (`ngx_chain_t`) are actively used in nginx, so the nginx pool implementation provides a way to reuse them. The `chain` field of `ngx_pool_t` keeps a list of previously allocated links ready for reuse. For efficient allocation of a chain link in a pool, use the `ngx_alloc_chain_link(pool)` function. This function looks up a free chain link in the pool list and allocates a new chain link if the pool list is empty. To free a link, call the `ngx_free_chain(pool, cl)` function.

​	链表链接（`ngx_chain_t`）在 nginx 中得到广泛使用，因此 nginx 内存池实现了一种重用它们的方式。`ngx_pool_t` 的 `chain` 字段保持先前分配的链接的列表，以便随时可以重用它们。为了在内存池中高效地分配链表链接，使用 `ngx_alloc_chain_link(pool)` 函数。此函数在池列表中查找一个空闲链表链接，并在列表为空时分配一个新的链表链接。要释放链接，调用 `ngx_free_chain(pool, cl)` 函数。

Cleanup handlers can be registered in a pool. A cleanup handler is a callback with an argument which is called when pool is destroyed. A pool is usually tied to a specific nginx object (like an HTTP request) and is destroyed when the object reaches the end of its lifetime. Registering a pool cleanup is a convenient way to release resources, close file descriptors or make final adjustments to the shared data associated with the main object.

​	可以在内存池中注册清理处理程序。清理处理程序是一个带有参数的回调，当内存池被销毁时会调用。内存池通常与特定的 nginx 对象相关联（如 HTTP 请求），并在对象的生命周期结束时销毁。注册内存池清理是释放资源、关闭文件描述符或对与主对象关联的共享数据进行最终调整的便捷方式。

To register a pool cleanup, call `ngx_pool_cleanup_add(pool, size)`, which returns a `ngx_pool_cleanup_t` pointer to be filled in by the caller. Use the `size` argument to allocate context for the cleanup handler.

​	要注册内存池清理，调用 `ngx_pool_cleanup_add(pool, size)` 函数，该函数返回一个 `ngx_pool_cleanup_t` 指针，由调用者填充。使用 `size` 参数为清理处理程序分配上下文。

```
ngx_pool_cleanup_t  *cln;

cln = ngx_pool_cleanup_add(pool, 0);
if (cln == NULL) { /* error */ }

cln->handler = ngx_my_cleanup;
cln->data = "foo";

...

static void
ngx_my_cleanup(void *data)
{
    u_char  *msg = data;

    ngx_do_smth(msg);
}
```



### 共享内存 Shared memory

Shared memory is used by nginx to share common data between processes. The `ngx_shared_memory_add(cf, name, size, tag)` function adds a new shared memory entry `ngx_shm_zone_t` to a cycle. The function receives the `name` and `size` of the zone. Each shared zone must have a unique name. If a shared zone entry with the provided `name` and `tag` already exists, the existing zone entry is reused. The function fails with an error if an existing entry with the same name has a different tag. Usually, the address of the module structure is passed as `tag`, making it possible to reuse shared zones by name within one nginx module.

​	共享内存用于在 nginx 进程之间共享公共数据。`ngx_shared_memory_add(cf, name, size, tag)` 函数将一个新的共享内存条目 `ngx_shm_zone_t` 添加到循环中。该函数接收区域的 `name` 和 `size`。每个共享区域必须有一个唯一的名称。如果具有相同名称的共享区域条目与提供的 `name` 和 `tag` 已经存在，则会重新使用现有的区域条目。如果具有相同名称的现有条目具有不同的标记，则该函数将失败并显示错误。通常，将模块结构的地址作为 `tag` 传递，从而可以在一个 nginx 模块中按名称重用共享区域。

The shared memory entry structure `ngx_shm_zone_t` has the following fields:

​	共享内存条目结构 `ngx_shm_zone_t` 具有以下字段：

- `init` — Initialization callback, called after the shared zone is mapped to actual memory

- `init` — 初始化回调，在共享区域映射到实际内存后调用

- `data` — Data context, used to pass arbitrary data to the `init` callback

- `data` — 数据上下文，用于将任意数据传递给 `init` 回调

- `noreuse` — Flag that disables reuse of a shared zone from the old cycle

- `noreuse` — 禁用从旧循环重用共享区域的标志

- `tag` — Shared zone tag

- `tag` — 共享区域标签

- `shm` — Platform-specific object of type `ngx_shm_t`, having at least the following fields:
  
- `shm` — 平台特定的类型为 `ngx_shm_t` 的对象，至少具有以下字段：
  - `addr` — Mapped shared memory address, initially NULL
  - `size` — Shared memory size
  - `name` — Shared memory name
  - `log` — Shared memory log
  - `exists` — Flag that indicates shared memory was inherited from the master process (Windows-specific)
  - `addr` — 映射的共享内存地址，最初为 NULL
  - `size` — 共享内存大小
  - `name` — 共享内存名称
  - `log` — 共享内存日志
  - `exists` — 指示共享内存是否从主进程继承而来的标志（仅适用于 Windows）



Shared zone entries are mapped to actual memory in `ngx_init_cycle()` after the configuration is parsed. On POSIX systems, the `mmap()` syscall is used to create the shared anonymous mapping. On Windows, the `CreateFileMapping()`/ `MapViewOfFileEx()` pair is used.

​	共享区域条目在解析配置后由 `ngx_init_cycle()` 中的映射到实际内存。在 POSIX 系统上，使用 `mmap()` 系统调用来创建共享匿名映射。在 Windows 上，使用 `CreateFileMapping()`/`MapViewOfFileEx()` 组合。

For allocating in shared memory, nginx provides the slab pool `ngx_slab_pool_t` type. A slab pool for allocating memory is automatically created in each nginx shared zone. The pool is located in the beginning of the shared zone and can be accessed by the expression `(ngx_slab_pool_t *) shm_zone->shm.addr`. To allocate memory in a shared zone, call either `ngx_slab_alloc(pool, size)` or `ngx_slab_calloc(pool, size)`. To free memory, call `ngx_slab_free(pool, p)`.

​	为了在共享内存中进行分配，nginx 提供了 `ngx_slab_pool_t` 类型的分配。为分配内存的每个 nginx 共享区域自动创建了一个分配内存的 slab 池。该池位于共享区域的开头，并且可以通过表达式 `(ngx_slab_pool_t *) shm_zone->shm.addr` 来访问。要在共享区域中分配内存，调用 `ngx_slab_alloc(pool, size)` 或 `ngx_slab_calloc(pool, size)`。要释放内存，调用 `ngx_slab_free(pool, p)`。

Slab pool divides all shared zone into pages. Each page is used for allocating objects of the same size. The specified size must be a power of 2, and greater than the minimum size of 8 bytes. Nonconforming values are rounded up. A bitmask for each page tracks which blocks are in use and which are free for allocation. For sizes greater than a half page (which is usually 2048 bytes), allocation is done an entire page at a time

​	Slab 池将所有共享区域划分为页面。每个页面用于分配相同大小的对象。指定的大小必须是 2 的幂，且大于最小大小 8 字节。非标准的值会被舍入。每个页面都有一个位掩码，用于跟踪哪些块正在使用，哪些可以用于分配。对于大于半页（通常为 2048 字节）大小的分配，会整页进行分配。

To protect data in shared memory from concurrent access, use the mutex available in the `mutex` field of `ngx_slab_pool_t`. A mutex is most commonly used by the slab pool while allocating and freeing memory, but it can be used to protect any other user data structures allocated in the shared zone. To lock or unlock a mutex, call `ngx_shmtx_lock(&shpool->mutex)` or `ngx_shmtx_unlock(&shpool->mutex)` respectively.

​	为了保护共享内存中的数据免受并发访问，可以使用 `ngx_slab_pool_t` 的 `mutex` 字段中的互斥锁。在分配和释放内存时，Slab 池通常会使用互斥锁，但可以用于保护在共享区域中分配的任何其他用户数据结构。要锁定或解锁互斥锁，分别调用 `ngx_shmtx_lock(&shpool->mutex)` 或 `ngx_shmtx_unlock(&shpool->mutex)`。

```
ngx_str_t        name;
ngx_foo_ctx_t   *ctx;
ngx_shm_zone_t  *shm_zone;

ngx_str_set(&name, "foo");

/* allocate shared zone context */
/* 分配共享区域上下文 */
ctx = ngx_pcalloc(cf->pool, sizeof(ngx_foo_ctx_t));
if (ctx == NULL) {
    /* error */
    /* 错误处理 */
}

/* add an entry for 64k shared zone */
/* 添加一个 64k 的共享区域条目 */
shm_zone = ngx_shared_memory_add(cf, &name, 65536, &ngx_foo_module);
if (shm_zone == NULL) {
    /* error */
    /* 错误处理 */
}

/* register init callback and context */
/* 注册初始化回调和上下文 */
shm_zone->init = ngx_foo_init_zone;
shm_zone->data = ctx;


...


static ngx_int_t
ngx_foo_init_zone(ngx_shm_zone_t *shm_zone, void *data)
{
    ngx_foo_ctx_t  *octx = data;

    size_t            len;
    ngx_foo_ctx_t    *ctx;
    ngx_slab_pool_t  *shpool;

    value = shm_zone->data;

    if (octx) {
        /* reusing a shared zone from old cycle */
        /* 从旧循环中重用共享区域 */
        ctx->value = octx->value;
        return NGX_OK;
    }

    shpool = (ngx_slab_pool_t *) shm_zone->shm.addr;

    if (shm_zone->shm.exists) {
        /* initialize shared zone context in Windows nginx worker */
        /* 在 Windows nginx 工作进程中初始化共享区域上下文 */
        ctx->value = shpool->data;
        return NGX_OK;
    }

    /* initialize shared zone */
    /* 初始化共享区域 */

    ctx->value = ngx_slab_alloc(shpool, sizeof(ngx_uint_t));
    if (ctx->value == NULL) {
        return NGX_ERROR;
    }

    shpool->data = ctx->value;

    return NGX_OK;
}
```



## 日志记录 Logging

For logging nginx uses `ngx_log_t` objects. The nginx logger supports several types of output:

​	nginx 使用 `ngx_log_t` 对象进行日志记录。nginx 日志记录器支持以下几种类型的输出：

- stderr — Logging to standard error (stderr)
- file — Logging to a file
- syslog — Logging to syslog
- memory — Logging to internal memory storage for development purposes; the memory can be accessed later with a debugger
- stderr — 日志记录到标准错误（stderr）
- file — 日志记录到文件
- syslog — 日志记录到 syslog
- memory — 为开发目的将日志记录到内部内存存储中；可以稍后使用调试器访问内存

A logger instance can be a chain of loggers, linked to each other with the `next` field. In this case, each message is written to all loggers in the chain.

​	一个日志记录器实例可以是一个日志链，通过 `next` 字段链接在一起。在这种情况下，每条消息都会写入链中的所有日志记录器。

For each logger, a severity level controls which messages are written to the log (only events assigned that level or higher are logged). The following severity levels are supported:

​	对于每个日志记录器，严重级别控制着写入日志的消息（只有分配了该级别或更高级别的事件才会被记录）。支持以下严重级别： 

- `NGX_LOG_EMERG`
- `NGX_LOG_ALERT`
- `NGX_LOG_CRIT`
- `NGX_LOG_ERR`
- `NGX_LOG_WARN`
- `NGX_LOG_NOTICE`
- `NGX_LOG_INFO`
- `NGX_LOG_DEBUG`

For debug logging, the debug mask is checked as well. The debug masks are:

​	对于调试日志记录，还会检查调试掩码。调试掩码包括：

- `NGX_LOG_DEBUG_CORE`
- `NGX_LOG_DEBUG_ALLOC`
- `NGX_LOG_DEBUG_MUTEX`
- `NGX_LOG_DEBUG_EVENT`
- `NGX_LOG_DEBUG_HTTP`
- `NGX_LOG_DEBUG_MAIL`
- `NGX_LOG_DEBUG_STREAM`

Normally, loggers are created by existing nginx code from `error_log` directives and are available at nearly every stage of processing in cycle, configuration, client connection and other objects.

​	通常，日志记录器是由现有的 nginx 代码从 `error_log` 指令创建的，并在几乎每个处理阶段的周期、配置、客户端连接和其他对象中都可用。

Nginx provides the following logging macros:

​	nginx 提供以下日志记录宏：

- `ngx_log_error(level, log, err, fmt, ...)` — Error logging
- `ngx_log_debug0(level, log, err, fmt)`, `ngx_log_debug1(level, log, err, fmt, arg1)` etc — Debug logging with up to eight supported formatting arguments
- `ngx_log_error(level, log, err, fmt, ...)` — 错误日志记录
- `ngx_log_debug0(level, log, err, fmt)`、`ngx_log_debug1(level, log, err, fmt, arg1)` 等 — 带有最多八个支持的格式化参数的调试日志记录



A log message is formatted in a buffer of size `NGX_MAX_ERROR_STR` (currently, 2048 bytes) on stack. The message is prepended with the severity level, process ID (PID), connection ID (stored in `log->connection`), and the system error text. For non-debug messages `log->handler` is called as well to prepend more specific information to the log message. HTTP module sets `ngx_http_log_error()` function as log handler to log client and server addresses, current action (stored in `log->action`), client request line, server name etc.

​	日志消息在大小为 `NGX_MAX_ERROR_STR`（当前为 2048 字节）的堆栈上格式化。消息前面带有严重级别、进程 ID（PID）、连接 ID（存储在 `log->connection` 中）和系统错误文本。对于非调试消息，还会调用 `log->handler` 来在日志消息中添加更多的特定信息。HTTP 模块将 `ngx_http_log_error()` 函数设置为日志处理程序，以记录客户端和服务器地址、当前操作（存储在 `log->action` 中）、客户端请求行、服务器名称等。

```
/* specify what is currently done */
/* 指定当前正在执行的操作 */
log->action = "sending mp4 to client";

/* error and debug log */
/* 错误和调试日志 */
ngx_log_error(NGX_LOG_INFO, c->log, 0, "client prematurely
              closed connection");

ngx_log_debug2(NGX_LOG_DEBUG_HTTP, mp4->file.log, 0,
               "mp4 start:%ui, length:%ui", mp4->start, mp4->length);
```

The example above results in log entries like these:

​	上面的示例将产生类似以下的日志条目：

```
2016/09/16 22:08:52 [info] 17445#0: *1 client prematurely closed connection while
sending mp4 to client, client: 127.0.0.1, server: , request: "GET /file.mp4 HTTP/1.1"
2016/09/16 23:28:33 [debug] 22140#0: *1 mp4 start:0, length:10000
```



## 循环 Cycle

A cycle object stores the nginx runtime context created from a specific configuration. Its type is `ngx_cycle_t`. The current cycle is referenced by the `ngx_cycle` global variable and inherited by nginx workers as they start. Each time the nginx configuration is reloaded, a new cycle is created from the new nginx configuration; the old cycle is usually deleted after the new one is successfully created.

​	循环对象存储了从特定配置创建的nginx运行时上下文。它的类型是`ngx_cycle_t`。当前循环由`ngx_cycle`全局变量引用，并在nginx worker启动时被继承。每次重新加载nginx配置时，都会从新的nginx配置创建一个新的循环；通常在成功创建新循环后会删除旧循环。

A cycle is created by the `ngx_init_cycle()` function, which takes the previous cycle as its argument. The function locates the previous cycle's configuration file and inherits as many resources as possible from the previous cycle. A placeholder cycle called "init cycle" is created as nginx start, then is replaced by an actual cycle built from configuration.

​	循环是通过`ngx_init_cycle()`函数创建的，该函数以前一个循环作为其参数。该函数定位前一个循环的配置文件，并尽可能多地继承前一个循环的资源。一个名为“init cycle”的占位循环在nginx启动时创建，然后被从配置构建的实际循环替换。

Members of the cycle include:

​	循环的成员包括：

- `pool` — Cycle pool. Created for each new cycle.
- `pool` — 循环池。为每个新循环创建。
- `log` — Cycle log. Initially inherited from the old cycle, it is set to point to `new_log` after the configuration is read.
- `log` — 循环日志。最初从旧循环继承，读取配置后设置为指向`new_log`。
- `new_log` — Cycle log, created by the configuration. It's affected by the root-scope `error_log` directive.
- `new_log` — 循环日志，由配置创建。它受根范围的`error_log`指令影响。
- `connections`, `connection_n` — Array of connections of type `ngx_connection_t`, created by the event module while initializing each nginx worker. The `worker_connections` directive in the nginx configuration sets the number of connections `connection_n`.
- `connections`、`connection_n` — `ngx_connection_t`类型的连接数组，由事件模块在初始化每个nginx worker时创建。nginx配置中的`worker_connections`指令设置连接数`connection_n`。
- `free_connections`, `free_connection_n` — List and number of currently available connections. If no connections are available, an nginx worker refuses to accept new clients or connect to upstream servers.
- `free_connections`、`free_connection_n` — 当前可用连接的列表和数量。如果没有可用的连接，nginx worker将拒绝接受新客户端或连接到上游服务器。
- `files`, `files_n` — Array for mapping file descriptors to nginx connections. This mapping is used by the event modules, having the `NGX_USE_FD_EVENT` flag (currently, it's `poll` and `devpoll`).
- `files`、`files_n` — 用于将文件描述符映射到nginx连接的数组。此映射由事件模块使用，具有`NGX_USE_FD_EVENT`标志（目前为`poll`和`devpoll`）。
- `conf_ctx` — Array of core module configurations. The configurations are created and filled during reading of nginx configuration files.
- `conf_ctx` — 核心模块配置的数组。在读取nginx配置文件时创建和填充这些配置。
- `modules`, `modules_n` — Array of modules of type `ngx_module_t`, both static and dynamic, loaded by the current configuration.
- `modules`、`modules_n` — `ngx_module_t`类型的模块数组，包括静态模块和动态模块，由当前配置加载。
- `listening` — Array of listening objects of type `ngx_listening_t`. Listening objects are normally added by the `listen` directive of different modules which call the `ngx_create_listening()` function. Listen sockets are created based on the listening objects.
- `listening` — `ngx_listening_t`类型的监听对象数组。监听对象通常由不同模块的`listen`指令添加，这些模块调用`ngx_create_listening()`函数。基于监听对象创建监听套接字。
- `paths` — Array of paths of type `ngx_path_t`. Paths are added by calling the function `ngx_add_path()` from modules which are going to operate on certain directories. These directories are created by nginx after reading configuration, if missing. Moreover, two handlers can be added for each path:
- `paths` — `ngx_path_t`类型的路径数组。路径由将要操作特定目录的模块调用`ngx_add_path()`函数添加。如果缺少这些目录，nginx在读取配置后会创建它们。此外，对于每个路径，可以添加两个处理程序：
  - path loader — Executes only once in 60 seconds after starting or reloading nginx. Normally, the loader reads the directory and stores data in nginx shared memory. The handler is called from the dedicated nginx process “nginx cache loader”.
  - 路径加载器 — 在启动或重新加载nginx后的60秒内仅执行一次。通常，加载器读取目录并将数据存储在nginx共享内存中。处理程序从专用的nginx进程“nginx cache loader”调用。
  - path manager — Executes periodically. Normally, the manager removes old files from the directory and updates nginx memory to reflect the changes. The handler is called from the dedicated “nginx cache manager” process.
  - 路径管理器 — 定期执行。通常，管理器从目录中删除旧文件并更新nginx内存以反映更改。处理程序从专用的“nginx cache manager”进程调用。
- `open_files` — List of open file objects of type `ngx_open_file_t`, which are created by calling the function `ngx_conf_open_file()`. Currently, nginx uses this kind of open files for logging. After reading the configuration, nginx opens all files in the `open_files` list and stores each file descriptor in the object's `fd` field. The files are opened in append mode and are created if missing. The files in the list are reopened by nginx workers upon receiving the reopen signal (most often `USR1`). In this case the descriptor in the `fd` field is changed to a new value.
- `open_files` — `ngx_open_file_t`类型的打开文件对象列表，通过调用`ngx_conf_open_file()`函数创建。目前，nginx将此类打开文件用于记录。在读取配置后，nginx打开`open_files`列表中的所有文件，并将每个文件描述符存储在对象的`fd`字段中。这些文件以追加模式打开，如果缺少则会创建。nginx worker在接收到重新打开信号（通常是`USR1`）时会重新打开列表中的文件。在这种情况下，`fd`字段中的描述符会更改为新值。
- `shared_memory` — List of shared memory zones, each added by calling the `ngx_shared_memory_add()` function. Shared zones are mapped to the same address range in all nginx processes and are used to share common data, for example the HTTP cache in-memory tree.
- `shared_memory` — 共享内存区域的列表，每个区域通过调用`ngx_shared_memory_add()`函数添加。共享区域在所有nginx进程中映射到相同的地址范围，用于共享公共数据，例如HTTP缓存内存树。





## 缓冲区 Buffer

For input/output operations, nginx provides the buffer type `ngx_buf_t`. Normally, it's used to hold data to be written to a destination or read from a source. A buffer can reference data in memory or in a file and it's technically possible for a buffer to reference both at the same time. Memory for the buffer is allocated separately and is not related to the buffer structure `ngx_buf_t`.

​	对于输入/输出操作，nginx提供了缓冲区类型`ngx_buf_t`。通常，它用于保存要写入目标或从源读取的数据。缓冲区可以引用内存中的数据或文件中的数据，从技术上讲，缓冲区可以同时引用两者。缓冲区的内存是单独分配的，与缓冲区结构`ngx_buf_t`无关。

The `ngx_buf_t` structure has the following fields:

​	`ngx_buf_t`结构具有以下字段

- `start`, `end` — The boundaries of the memory block allocated for the buffer.
- `start`、`end` — 为缓冲区分配的内存块的边界。
- `pos`, `last` — The boundaries of the memory buffer; normally a subrange of `start` .. `end`.
- `pos`、`last` — 内存缓冲区的边界；通常是`start`..`end`的子范围。
- `file_pos`, `file_last` — The boundaries of a file buffer, expressed as offsets from the beginning of the file.
- `file_pos`、`file_last` — 文件缓冲区的边界，以从文件开头的偏移量表示。
- `tag` — Unique value used to distinguish buffers; created by different nginx modules, usually for the purpose of buffer reuse.
- `tag` — 用于区分缓冲区的唯一值；由不同的nginx模块创建，通常用于缓冲区重用的目的。
- `file` — File object.
- `file` — 文件对象。
- `temporary` — Flag indicating that the buffer references writable memory.
- `temporary` — 表示缓冲区引用可写内存的标志。
- `memory` — Flag indicating that the buffer references read-only memory.
- `memory` — 表示缓冲区引用只读内存的标志。
- `in_file` — Flag indicating that the buffer references data in a file.
- `in_file` — 表示缓冲区引用文件中的数据的标志。
- `flush` — Flag indicating that all data prior to the buffer need to be flushed.
- `flush` — 表示需要刷新缓冲区之前的所有数据的标志。
- `recycled` — Flag indicating that the buffer can be reused and needs to be consumed as soon as possible.
- `recycled` — 表示可以重用缓冲区，并且需要尽快使用的标志。
- `sync` — Flag indicating that the buffer carries no data or special signal like `flush` or `last_buf`. By default nginx considers such buffers an error condition, but this flag tells nginx to skip the error check.
- `sync` — 表示缓冲区不携带数据或特殊信号（如`flush`或`last_buf`）的标志。默认情况下，nginx将这样的缓冲区视为错误条件，但此标志告诉nginx跳过错误检查。
- `last_buf` — Flag indicating that the buffer is the last in output.
- `last_buf` — 表示输出中的最后一个缓冲区的标志。
- `last_in_chain` — Flag indicating that there are no more data buffers in a request or subrequest.
- `last_in_chain` — 表示请求或子请求中没有更多数据缓冲区的标志。
- `shadow` — Reference to another ("shadow") buffer related to the current buffer, usually in the sense that the buffer uses data from the shadow. When the buffer is consumed, the shadow buffer is normally also marked as consumed.
- `shadow` — 引用与当前缓冲区相关的另一个（“shadow”）缓冲区，通常是指缓冲区使用来自该shadow的数据。当缓冲区被消耗时，通常也会将shadow缓冲区标记为已消耗。
- `last_shadow` — Flag indicating that the buffer is the last one that references a particular shadow buffer.
- `last_shadow` — 表示缓冲区是引用特定shadow缓冲区的最后一个缓冲区的标志。
- `temp_file` — Flag indicating that the buffer is in a temporary file.
- `temp_file` — 表示缓冲区在临时文件中的标志。



For input and output operations buffers are linked in chains. A chain is a sequence of chain links of type `ngx_chain_t`, defined as follows:

​	对于输入和输出操作，缓冲区被链接成链。链是类型为`ngx_chain_t`的链环的序列，定义如下：

```
typedef struct ngx_chain_s  ngx_chain_t;

struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
};
```

Each chain link keeps a reference to its buffer and a reference to the next chain link.

​	每个链环保持对其缓冲区的引用以及对下一个链环的引用。

An example of using buffers and chains:

​	以下是使用缓冲区和链环的示例：

```
ngx_chain_t *
ngx_get_my_chain(ngx_pool_t *pool)
{
    ngx_buf_t    *b;
    ngx_chain_t  *out, *cl, **ll;

    /* first buf */
    /* 第一个缓冲区 */
    cl = ngx_alloc_chain_link(pool);
    if (cl == NULL) { /* error */ }

    b = ngx_calloc_buf(pool);
    if (b == NULL) { /* error */ }

    b->start = (u_char *) "foo";
    b->pos = b->start;
    b->end = b->start + 3;
    b->last = b->end;
    b->memory = 1; /* read-only memory */ /* 只读内存 */

    cl->buf = b;
    out = cl;
    ll = &cl->next;

    /* second buf */
    /* 第二个缓冲区 */
    cl = ngx_alloc_chain_link(pool);
    if (cl == NULL) { /* error */ }

    b = ngx_create_temp_buf(pool, 3);
    if (b == NULL) { /* error */ }

    b->last = ngx_cpymem(b->last, "foo", 3);

    cl->buf = b;
    cl->next = NULL;
    *ll = cl;

    return out;
}
```



## 网络通信 Networking



### 连接 Connection

The connection type `ngx_connection_t` is a wrapper around a socket descriptor. It includes the following fields:

​	`ngx_connection_t`类型是对套接字描述符的包装。它包括以下字段：



- `fd` — Socket descriptor
- `fd` — 套接字描述符。
- `data` — Arbitrary connection context. Normally, it is a pointer to a higher-level object built on top of the connection, such as an HTTP request or a Stream session.
- `data` — 任意连接上下文。通常，它是指向构建在连接之上的更高级对象的指针，比如HTTP请求或Stream会话。
- `read`, `write` — Read and write events for the connection.
- `read`、`write` — 用于连接的读写事件。
- `recv`, `send`, `recv_chain`, `send_chain` — I/O operations for the connection.
- `recv`、`send`、`recv_chain`、`send_chain` — 用于连接的I/O操作。
- `pool` — Connection pool.
- `pool` — 连接池。
- `log` — Connection log.
- `log` — 连接日志。
- `sockaddr`, `socklen`, `addr_text` — Remote socket address in binary and text forms.
- `sockaddr`、`socklen`、`addr_text` — 远程套接字地址的二进制和文本形式。
- `local_sockaddr`, `local_socklen` — Local socket address in binary form. Initially, these fields are empty. Use the `ngx_connection_local_sockaddr()` function to get the local socket address.
- `local_sockaddr`、`local_socklen` — 本地套接字地址的二进制形式。最初，这些字段是空的。使用`ngx_connection_local_sockaddr()`函数获取本地套接字地址。
- `proxy_protocol_addr`, `proxy_protocol_port` - PROXY protocol client address and port, if the PROXY protocol is enabled for the connection.
- `proxy_protocol_addr`、`proxy_protocol_port` — 如果为连接启用了PROXY协议，则是PROXY协议客户端的地址和端口。
- `ssl` — SSL context for the connection.
- `ssl` — 连接的SSL上下文。
- `reusable` — Flag indicating the connection is in a state that makes it eligible for reuse.
- `reusable` — 表示连接处于可重用状态的标志。
- `close` — Flag indicating that the connection is being reused and needs to be closed.
- `close` — 表示连接正在被重用并且需要关闭的标志。



An nginx connection can transparently encapsulate the SSL layer. In this case the connection's `ssl` field holds a pointer to an `ngx_ssl_connection_t` structure, keeping all SSL-related data for the connection, including `SSL_CTX` and `SSL`. The `recv`, `send`, `recv_chain`, and `send_chain` handlers are set to SSL-enabled functions as well.

​	nginx连接可以透明地封装SSL层。在这种情况下，连接的`ssl`字段持有指向`ngx_ssl_connection_t`结构的指针，该结构保存连接的所有与SSL相关的数据，包括`SSL_CTX`和`SSL`。`recv`、`send`、`recv_chain`和`send_chain`处理程序也设置为启用SSL的函数。

The `worker_connections` directive in the nginx configuration limits the number of connections per nginx worker. All connection structures are precreated when a worker starts and stored in the `connections` field of the cycle object. To retrieve a connection structure, use the `ngx_get_connection(s, log)` function. It takes as its `s` argument a socket descriptor, which needs to be wrapped in a connection structure.

​	nginx配置中的`worker_connections`指令限制了每个nginx worker的连接数。在worker启动时，所有连接结构都会预先创建，并存储在循环对象的`connections`字段中。要检索连接结构，请使用`ngx_get_connection(s, log)`函数。它以套接字描述符作为`s`参数，该描述符需要包装在连接结构中。

Because the number of connections per worker is limited, nginx provides a way to grab connections that are currently in use. To enable or disable reuse of a connection, call the `ngx_reusable_connection(c, reusable)` function. Calling `ngx_reusable_connection(c, 1)` sets the `reuse` flag in the connection structure and inserts the connection into the `reusable_connections_queue` of the cycle. Whenever `ngx_get_connection()` finds out there are no available connections in the cycle's `free_connections` list, it calls `ngx_drain_connections()` to release a specific number of reusable connections. For each such connection, the `close` flag is set and its read handler is called which is supposed to free the connection by calling `ngx_close_connection(c)` and make it available for reuse. To exit the state when a connection can be reused `ngx_reusable_connection(c, 0)` is called. HTTP client connections are an example of reusable connections in nginx; they are marked as reusable until the first request byte is received from the client.

​	由于每个工作进程的连接数有限，nginx提供了一种方法来获取当前正在使用的连接。要启用或禁用连接的重用，调用`ngx_reusable_connection(c, reusable)`函数。调用`ngx_reusable_connection(c, 1)`会在连接结构中设置`reuse`标志，并将连接插入循环的`reusable_connections_queue`中。每当`ngx_get_connection()`发现在循环的`free_connections`列表中没有可用连接时，它会调用`ngx_drain_connections()`来释放一定数量的可重用连接。对于每个这样的连接，设置`close`标志并调用其读处理程序，该处理程序应该通过调用`ngx_close_connection(c)`来释放连接，并使其可供重用。要退出可以重新使用连接的状态，调用`ngx_reusable_connection(c, 0)`。HTTP客户端连接是nginx中可重用连接的一个示例；它们被标记为可重用，直到从客户端接收到第一个请求字节为止。



## 事件 Events



### 事件 Event

Event object `ngx_event_t` in nginx provides a mechanism for notification that a specific event has occurred.

​	nginx中的事件对象`ngx_event_t`提供了一种机制，用于通知特定事件已发生。

Fields in `ngx_event_t` include:

​	`ngx_event_t`中的字段包括：



- `data` — Arbitrary event context used in event handlers, usually as pointer to a connection related to the event.
- `data` — 在事件处理程序中使用的任意事件上下文，通常作为与事件相关的连接的指针。
- `handler` — Callback function to be invoked when the event happens.
- `handler` — 当事件发生时要调用的回调函数。
- `write` — Flag indicating a write event. Absence of the flag indicates a read event.
- `write` — 标志，指示写事件。缺少此标志表示读事件。
- `active` — Flag indicating that the event is registered for receiving I/O notifications, normally from notification mechanisms like `epoll`, `kqueue`, `poll`.
- `active` — 标志，指示该事件已注册用于接收I/O通知，通常来自诸如`epoll`、`kqueue`、`poll`之类的通知机制。
- `ready` — Flag indicating that the event has received an I/O notification.
- `ready` — 标志，指示该事件已收到I/O通知。
- `delayed` — Flag indicating that I/O is delayed due to rate limiting.
- `delayed` — 标志，指示I/O由于速率限制而被延迟。
- `timer` — Red-black tree node for inserting the event into the timer tree.
- `timer` — 红黑树节点，用于将事件插入计时器树中。
- `timer_set` — Flag indicating that the event timer is set and not yet expired.
- `timer_set` — 标志，指示事件计时器已设置且尚未过期。
- `timedout` — Flag indicating that the event timer has expired.
- `timedout` — 标志，指示事件计时器已过期。
- `eof` — Flag indicating that EOF occurred while reading data.
- `eof` — 标志，指示在读取数据时发生EOF。
- `pending_eof` — Flag indicating that EOF is pending on the socket, even though there may be some data available before it. The flag is delivered via the `EPOLLRDHUP` `epoll` event or `EV_EOF` `kqueue` flag.
- `pending_eof` — 标志，指示套接字上有挂起的EOF，尽管在此之前可能有一些可用的数据。该标志通过`EPOLLRDHUP` `epoll`事件或`EV_EOF` `kqueue`标志传递。
- `error` — Flag indicating that an error occurred during reading (for a read event) or writing (for a write event).
- `error` — 标志，指示在读取（对于读事件）或写入（对于写事件）期间发生错误。
- `cancelable` — Timer event flag indicating that the event should be ignored while shutting down the worker. Graceful worker shutdown is delayed until there are no non-cancelable timer events scheduled.
- `cancelable` — 计时器事件标志，指示在关闭工作进程时应忽略该事件。只有在没有可取消的计时器事件安排时，才会延迟优雅的工作进程关闭。
- `posted` — Flag indicating that the event is posted to a queue.
- `posted` — 标志，指示事件已发布到队列中。
- `queue` — Queue node for posting the event to a queue.
- `queue` — 用于将事件发布到队列的队列节点。





### I/O 事件 I/O events

Each connection obtained by calling the `ngx_get_connection()` function has two attached events, `c->read` and `c->write`, which are used for receiving notification that the socket is ready for reading or writing. All such events operate in Edge-Triggered mode, meaning that they only trigger notifications when the state of the socket changes. For example, doing a partial read on a socket does not make nginx deliver a repeated read notification until more data arrives on the socket. Even when the underlying I/O notification mechanism is essentially Level-Triggered (`poll`, `select` etc), nginx converts the notifications to Edge-Triggered. To make nginx event notifications consistent across all notifications systems on different platforms, the functions `ngx_handle_read_event(rev, flags)` and `ngx_handle_write_event(wev, lowat)` must be called after handling an I/O socket notification or calling any I/O functions on that socket. Normally, the functions are called once at the end of each read or write event handler.

​	通过调用`ngx_get_connection()`函数获得的每个连接都有两个附加的事件：`c->read`和`c->write`，用于接收通知套接字已准备好读取或写入。所有这些事件都在边缘触发模式下运行，这意味着它们只在套接字状态更改时触发通知。例如，在套接字上进行部分读取不会导致nginx在套接字上有更多数据到达之前多次触发读取通知。即使底层的I/O通知机制本质上是水平触发的（`poll`、`select`等），nginx也会将通知转换为边缘触发的。为了使nginx的事件通知在不同平台上的所有通知系统中保持一致，必须在处理I/O套接字通知或在该套接字上调用任何I/O函数后调用`ngx_handle_read_event(rev, flags)`和`ngx_handle_write_event(wev, lowat)`函数。通常，在每个读取或写入事件处理程序的末尾调用这些函数一次。



### 计时器事件 Timer events

An event can be set to send a notification when a timeout expires. The timer used by events counts milliseconds since some unspecified point in the past truncated to `ngx_msec_t` type. Its current value can be obtained from the `ngx_current_msec` variable.

​	可以设置事件在超时后发送通知。事件使用的计时器计算自某个不确定的过去时间点以来的毫秒数，截断为`ngx_msec_t`类型。可以从`ngx_current_msec`变量中获取其当前值。

The function `ngx_add_timer(ev, timer)` sets a timeout for an event, `ngx_del_timer(ev)` deletes a previously set timeout. The global timeout red-black tree `ngx_event_timer_rbtree` stores all timeouts currently set. The key in the tree is of type `ngx_msec_t` and is the time when the event occurs. The tree structure enables fast insertion and deletion operations, as well as access to the nearest timeouts, which nginx uses to find out how long to wait for I/O events and for expiring timeout events.

​	函数`ngx_add_timer(ev, timer)`为事件设置超时，`ngx_del_timer(ev)`删除先前设置的超时。全局超时红黑树`ngx_event_timer_rbtree`存储当前设置的所有超时。树中的键是`ngx_msec_t`类型，是事件发生的时间。该树结构使得快速的插入和删除操作成为可能，同时还可以访问最近的超时时间，nginx使用这些时间来确定等待I/O事件和到期的超时事件的时间。



### 已发布事件 Posted events

An event can be posted which means that its handler will be called at some point later within the current event loop iteration. Posting events is a good practice for simplifying code and escaping stack overflows. Posted events are held in a post queue. The `ngx_post_event(ev, q)` macro posts the event `ev` to the post queue `q`. The `ngx_delete_posted_event(ev)` macro deletes the event `ev` from the queue it's currently posted in. Normally, events are posted to the `ngx_posted_events` queue, which is processed late in the event loop — after all I/O and timer events are already handled. The function `ngx_event_process_posted()` is called to process an event queue. It calls event handlers until the queue is not empty. This means that a posted event handler can post more events to be processed within the current event loop iteration.

​	可以发布事件，这意味着其处理程序将在当前事件循环迭代的某个时间点被调用。发布事件是简化代码并避免堆栈溢出的良好实践。发布的事件保存在发布队列中。宏`ngx_post_event(ev, q)`将事件`ev`发布到队列`q`中。宏`ngx_delete_posted_event(ev)`从其当前发布的队列中删除事件`ev`。通常，事件发布到`ngx_posted_events`队列，它在事件循环的后期进行处理 — 在所有I/O和计时器事件已处理之后。调用函数`ngx_event_process_posted()`来处理事件队列。它会连续删除发布的事件队列中的第一个元素并调用元素的处理程序，直到队列为空。这意味着发布的事件处理程序可以在当前事件循环迭代内发布更多的事件。

An example:

​	示例：

```
void
ngx_my_connection_read(ngx_connection_t *c)
{
    ngx_event_t  *rev;

    rev = c->read;

    ngx_add_timer(rev, 1000);

    rev->handler = ngx_my_read_handler;

    ngx_my_read(rev);
}


void
ngx_my_read_handler(ngx_event_t *rev)
{
    ssize_t            n;
    ngx_connection_t  *c;
    u_char             buf[256];

    if (rev->timedout) { /* timeout expired */ /* 超时处理 */ }

    c = rev->data;

    while (rev->ready) {
        n = c->recv(c, buf, sizeof(buf));

        if (n == NGX_AGAIN) {
            break;
        }

        if (n == NGX_ERROR) { /* error */ }

        /* process buf */
        /* 处理 buf */
    }

    if (ngx_handle_read_event(rev, 0) != NGX_OK) { /* error */ }
}
```



### 事件循环 Event loop

Except for the nginx master process, all nginx processes do I/O and so have an event loop. (The nginx master process instead spends most of its time in the `sigsuspend()` call waiting for signals to arrive.) The nginx event loop is implemented in the `ngx_process_events_and_timers()` function, which is called repeatedly until the process exits.

​	除了nginx主进程外，所有nginx进程都会执行I/O操作，因此都有一个事件循环。（nginx主进程主要在`sigsuspend()`调用中等待信号的到来。）nginx事件循环在`ngx_process_events_and_timers()`函数中实现，该函数会反复调用，直到进程退出。

The event loop has the following stages:

​	事件循环分为以下阶段：

- Find the timeout that is closest to expiring, by calling `ngx_event_find_timer()`. This function finds the leftmost node in the timer tree and returns the number of milliseconds until the node expires.
- 通过调用`ngx_event_find_timer()`找到最接近过期的超时。该函数找到计时器树中的最左节点，并返回节点到期的毫秒数。
- Process I/O events by calling a handler, specific to the event notification mechanism, chosen by nginx configuration. This handler waits for at least one I/O event to happen, but only until the next timeout expires. When a read or write event occurs, the `ready` flag is set and the event's handler is called. For Linux, the `ngx_epoll_process_events()` handler is normally used, which calls `epoll_wait()` to wait for I/O events.
- 通过调用特定于事件通知机制的处理程序处理I/O事件，该处理程序由nginx配置选择。此处理程序等待至少发生一个I/O事件，但仅在下一个超时到期之前等待。发生读取或写入事件时，将设置`ready`标志并调用事件的处理程序。对于Linux，通常使用`ngx_epoll_process_events()`处理程序，它调用`epoll_wait()`等待I/O事件。
- Expire timers by calling `ngx_event_expire_timers()`. The timer tree is iterated from the leftmost element to the right until an unexpired timeout is found. For each expired node the `timedout` event flag is set, the `timer_set` flag is reset, and the event handler is called
- 通过调用`ngx_event_expire_timers()`来超时计时器。从最左元素向右迭代计时器树，直到找到未过期的超时为止。对于每个过期的节点，将设置`timedout`事件标志，重置`timer_set`标志，并调用事件处理程序。
- Process posted events by calling `ngx_event_process_posted()`. The function repeatedly removes the first element from the posted events queue and calls the element's handler, until the queue is empty.
- 通过调用`ngx_event_process_posted()`处理发布的事件。该函数会反复从发布的事件队列中删除第一个元素并调用元素的处理程序，直到队列为空。



All nginx processes handle signals as well. Signal handlers only set global variables which are checked after the `ngx_process_events_and_timers()` call.

​	所有nginx进程也处理信号。信号处理程序仅设置全局变量，这些变量在调用`ngx_process_events_and_timers()`后进行检查。



## 进程 Processes

There are several types of processes in nginx. The type of a process is kept in the `ngx_process` global variable, and is one of the following:

​	nginx有几种类型的进程。进程的类型存储在`ngx_process`全局变量中，可能是以下之一：

- `NGX_PROCESS_MASTER` — The master process, which reads the NGINX configuration, creates cycles, and starts and controls child processes. It does not perform any I/O and responds only to signals. Its cycle function is `ngx_master_process_cycle()`.
- `NGX_PROCESS_MASTER` — 主进程，负责读取NGINX配置，创建循环，并启动和控制子进程。它不执行任何I/O操作，仅响应信号。其循环函数是`ngx_master_process_cycle()`。
- `NGX_PROCESS_WORKER` — The worker process, which handles client connections. It is started by the master process and responds to its signals and channel commands as well. Its cycle function is `ngx_worker_process_cycle()`. There can be multiple worker processes, as configured by the `worker_processes` directive.
- `NGX_PROCESS_WORKER` — 工作进程，负责处理客户端连接。它由主进程启动，并响应其信号和通道命令。其循环函数是`ngx_worker_process_cycle()`。可以有多个工作进程，由`worker_processes`指令配置。
- `NGX_PROCESS_SINGLE` — The single process, which exists only in `master_process off` mode, and is the only process running in that mode. It creates cycles (like the master process does) and handles client connections (like the worker process does). Its cycle function is `ngx_single_process_cycle()`.
- `NGX_PROCESS_SINGLE` — 单一进程，仅在`master_process off`模式下存在，是该模式下唯一运行的进程。它创建循环（与主进程相似）并处理客户端连接（与工作进程类似）。其循环函数是`ngx_single_process_cycle()`。
- `NGX_PROCESS_HELPER` — The helper process, of which currently there are two types: cache manager and cache loader. The cycle function for both is `ngx_cache_manager_process_cycle()`.
- `NGX_PROCESS_HELPER` — 助手进程，目前有两种类型：缓存管理器和缓存加载器。两者的循环函数均为`ngx_cache_manager_process_cycle()`。

The nginx processes handle the following signals:

​	nginx进程处理以下信号：

- `NGX_SHUTDOWN_SIGNAL` (`SIGQUIT` on most systems) — Gracefully shutdown. Upon receiving this signal, the master process sends a shutdown signal to all child processes. When no child processes are left, the master destroys the cycle pool and exits. When a worker process receives this signal, it closes all listening sockets and waits until there are no non-cancelable events scheduled, then destroys the cycle pool and exits. When the cache manager or the cache loader process receives this signal, it exits immediately. The `ngx_quit` variable is set to `1` when a process receives this signal, and is immediately reset after being processed. The `ngx_exiting` variable is set to `1` while a worker process is in the shutdown state.
- `NGX_SHUTDOWN_SIGNAL`（大多数系统上的`SIGQUIT`） — 优雅关闭。接收到此信号后，主进程向所有子进程发送关闭信号。当没有子进程剩余时，主进程销毁循环池并退出。当工作进程接收到此信号后，它关闭所有监听套接字，并等待直到没有非取消定时器事件安排，然后销毁循环池并退出。当缓存管理器或缓存加载器进程接收到此信号时，它会立即退出。变量`ngx_quit`在进程接收到此信号时设置为`1`，并在处理后立即重置。变量`ngx_exiting`在工作进程处于关闭状态时设置为`1`。
- `NGX_TERMINATE_SIGNAL` (`SIGTERM` on most systems) — Terminate. Upon receiving this signal, the master process sends a terminate signal to all child processes. If a child process does not exit within 1 second, the master process sends the `SIGKILL` signal to kill it. When no child processes are left, the master process destroys the cycle pool and exits. When a worker process, the cache manager process or the cache loader process receives this signal, it destroys the cycle pool and exits. The variable `ngx_terminate` is set to `1` when this signal is received.
- `NGX_TERMINATE_SIGNAL`（大多数系统上的`SIGTERM`） — 终止。接收到此信号后，主进程向所有子进程发送终止信号。如果子进程在1秒内不退出，主进程将发送`SIGKILL`信号来终止它。当没有子进程剩余时，主进程销毁循环池并退出。工作进程、缓存管理器进程或缓存加载器进程接收到此信号后，它们会销毁循环池并退出。变量`ngx_terminate`在接收到此信号时设置为`1`。
- `NGX_NOACCEPT_SIGNAL` (`SIGWINCH` on most systems) - Shut down all worker and helper processes. Upon receiving this signal, the master process shuts down its child processes. If a previously started new nginx binary exits, the child processes of the old master are started again. When a worker process receives this signal, it shuts down in debug mode set by the `debug_points` directive.
- `NGX_NOACCEPT_SIGNAL`（大多数系统上的`SIGWINCH`） — 关闭所有工作进程和助手进程。接收到此信号后，主进程关闭其子进程。如果以前启动的新nginx二进制文件退出，则旧主进程的子进程会再次启动。当工作进程接收到此信号时，它会在由`debug_points`指令设置的调试模式下关闭。
- `NGX_RECONFIGURE_SIGNAL` (`SIGHUP` on most systems) - Reconfigure. Upon receiving this signal, the master process re-reads the configuration and creates a new cycle based on it. If the new cycle is created successfully, the old cycle is deleted and new child processes are started. Meanwhile, the old child processes receive the `NGX_SHUTDOWN_SIGNAL` signal. In single-process mode, nginx creates a new cycle, but keeps the old one until there are no longer clients with active connections tied to it. The worker and helper processes ignore this signal.
- `NGX_RECONFIGURE_SIGNAL`（大多数系统上的`SIGHUP`） — 重新配置。接收到此信号后，主进程重新读取配置并根据其创建新的循环。如果成功创建新循环，则删除旧循环并启动新的子进程。同时，旧子进程接收到`NGX_SHUTDOWN_SIGNAL`信号。在单进程模式下，nginx会创建一个新循环，但在没有与之相关联的活动连接的客户端存在时，将保留旧循环。工作进程和助手进程忽略此信号。
- `NGX_REOPEN_SIGNAL` (`SIGUSR1` on most systems) — Reopen files. The master process sends this signal to workers, which reopen all `open_files` related to the cycle.
- `NGX_REOPEN_SIGNAL`（大多数系统上的`SIGUSR1`） — 重新打开文件。主进程将此信号发送给工作进程，工作进程将重新打开与循环相关的所有`open_files`。
- `NGX_CHANGEBIN_SIGNAL` (`SIGUSR2` on most systems) — Change the nginx binary. The master process starts a new nginx binary and passes in a list of all listen sockets. The text-format list, passed in the `“NGINX”` environment variable, consists of descriptor numbers separated with semicolons. The new nginx binary reads the `“NGINX”` variable and adds the sockets to its init cycle. Other processes ignore this signal.
- `NGX_CHANGEBIN_SIGNAL`（大多数系统上的`SIGUSR2`） — 更改nginx二进制文件。主进程启动新的nginx二进制文件，并将所有监听套接字传递给它。以文本格式传递的列表以`“NGINX”`环境变量传递，由分号分隔的描述符号。新的nginx二进制文件读取`“NGINX”`变量，并将套接字添加到其初始化循环中。其他进程忽略此信号。

While all nginx worker processes are able to receive and properly handle POSIX signals, the master process does not use the standard `kill()` syscall to pass signals to workers and helpers. Instead, nginx uses inter-process socket pairs which allow sending messages between all nginx processes. Currently, however, messages are only sent from the master to its children. The messages carry the standard signals.

​	尽管所有nginx工作进程都能够接收并正确处理POSIX信号，但主进程不使用标准的`kill()`系统调用将信号传递给工作进程和助手进程。相反，nginx使用进程间套接字对，允许在所有nginx进程之间发送消息。然而，当前只从主进程发送消息到其子进程。消息携带标准信号。



## 线程 Threads

It is possible to offload into a separate thread tasks that would otherwise block the nginx worker process. For example, nginx can be configured to use threads to perform [file I/O](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio). Another use case is a library that doesn't have asynchronous interface and thus cannot be normally used with nginx. Keep in mind that the threads interface is a helper for the existing asynchronous approach to processing client connections, and by no means intended as a replacement.

​	可以将原本会阻塞nginx工作进程的任务分派到单独的线程中。例如，可以配置nginx使用线程执行[文件I/O](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio)。另一个用例是一个没有异步接口的库，因此无法正常与nginx一起使用。请注意，线程接口是现有异步处理客户端连接方法的辅助工具，绝不是替代品。

To deal with synchronization, the following wrappers over `pthreads` primitives are available:

​	为了处理同步问题，以下是对`pthreads`原语的包装：

- ```
  typedef pthread_mutex_t ngx_thread_mutex_t;
  ```

  - `ngx_int_t ngx_thread_mutex_create(ngx_thread_mutex_t *mtx, ngx_log_t *log);`
  - `ngx_int_t ngx_thread_mutex_destroy(ngx_thread_mutex_t *mtx, ngx_log_t *log);`
  - `ngx_int_t ngx_thread_mutex_lock(ngx_thread_mutex_t *mtx, ngx_log_t *log);`
  - `ngx_int_t ngx_thread_mutex_unlock(ngx_thread_mutex_t *mtx, ngx_log_t *log);`

- ```
  typedef pthread_cond_t ngx_thread_cond_t;
  ```

  - `ngx_int_t ngx_thread_cond_create(ngx_thread_cond_t *cond, ngx_log_t *log);`
  - `ngx_int_t ngx_thread_cond_destroy(ngx_thread_cond_t *cond, ngx_log_t *log);`
  - `ngx_int_t ngx_thread_cond_signal(ngx_thread_cond_t *cond, ngx_log_t *log);`
  - `ngx_int_t ngx_thread_cond_wait(ngx_thread_cond_t *cond, ngx_thread_mutex_t *mtx, ngx_log_t *log);`



Instead of creating a new thread for each task, nginx implements a [thread_pool](https://nginx.org/en/docs/ngx_core_module.html#thread_pool) strategy. Multiple thread pools may be configured for different purposes (for example, performing I/O on different sets of disks). Each thread pool is created at startup and contains a limited number of threads that process a queue of tasks. When a task is completed, a predefined completion handler is called.

​	与为每个任务创建新线程不同，nginx实现了线程池策略。可以为不同目的（例如，在不同的磁盘上执行I/O）配置多个线程池。每个线程池在启动时创建，并包含一定数量的线程，这些线程处理任务队列。完成任务后，将调用预定义的完成处理程序。

The `src/core/ngx_thread_pool.h` header file contains relevant definitions:

​	`src/core/ngx_thread_pool.h` 头文件包含相关定义：

```
struct ngx_thread_task_s {
    ngx_thread_task_t   *next;
    ngx_uint_t           id;
    void                *ctx;
    void               (*handler)(void *data, ngx_log_t *log);
    ngx_event_t          event;
};

typedef struct ngx_thread_pool_s  ngx_thread_pool_t;

ngx_thread_pool_t *ngx_thread_pool_add(ngx_conf_t *cf, ngx_str_t *name);
ngx_thread_pool_t *ngx_thread_pool_get(ngx_cycle_t *cycle, ngx_str_t *name);

ngx_thread_task_t *ngx_thread_task_alloc(ngx_pool_t *pool, size_t size);
ngx_int_t ngx_thread_task_post(ngx_thread_pool_t *tp, ngx_thread_task_t *task);
```

At configuration time, a module willing to use threads has to obtain a reference to a thread pool by calling `ngx_thread_pool_add(cf, name)`, which either creates a new thread pool with the given `name` or returns a reference to the pool with that name if it already exists.

​	在配置时，一个希望使用线程的模块必须通过调用 `ngx_thread_pool_add(cf, name)` 来获取对线程池的引用，其中 `name` 是线程池的名称。该函数将创建一个新的线程池（如果不存在）并返回对该线程池的引用。

To add a `task` into a queue of a specified thread pool `tp` at runtime, use the `ngx_thread_task_post(tp, task)` function. To execute a function in a thread, pass parameters and setup a completion handler using the `ngx_thread_task_t` structure:

​	要在运行时将 `task` 添加到指定线程池 `tp` 的任务队列中，可以使用 `ngx_thread_task_post(tp, task)` 函数。要在线程中执行函数，需要通过 `ngx_thread_task_t` 结构传递参数并设置完成处理程序。

```
typedef struct {
    int    foo;
} my_thread_ctx_t;


static void
my_thread_func(void *data, ngx_log_t *log)
{
    my_thread_ctx_t *ctx = data;

    /* this function is executed in a separate thread */
    /* 此函数在单独的线程中执行 */
}


static void
my_thread_completion(ngx_event_t *ev)
{
    my_thread_ctx_t *ctx = ev->data;

    /* executed in nginx event loop */
    /* 在 nginx 事件循环中执行 */
}


ngx_int_t
my_task_offload(my_conf_t *conf)
{
    my_thread_ctx_t    *ctx;
    ngx_thread_task_t  *task;

    task = ngx_thread_task_alloc(conf->pool, sizeof(my_thread_ctx_t));
    if (task == NULL) {
        return NGX_ERROR;
    }

    ctx = task->ctx;

    ctx->foo = 42;

    task->handler = my_thread_func;
    task->event.handler = my_thread_completion;
    task->event.data = ctx;

    if (ngx_thread_task_post(conf->thread_pool, task) != NGX_OK) {
        return NGX_ERROR;
    }

    return NGX_OK;
}
```





## 模块 Modules



### 添加新模块 Adding new modules

Each standalone nginx module resides in a separate directory that contains at least two files: `config` and a file with the module source code. The `config` file contains all information needed for nginx to integrate the module, for example:

​	每个独立的 nginx 模块都位于一个单独的目录中，其中包含至少两个文件：`config` 和模块源代码文件。`config` 文件包含 nginx 集成模块所需的所有信息，例如：

```
ngx_module_type=CORE
ngx_module_name=ngx_foo_module
ngx_module_srcs="$ngx_addon_dir/ngx_foo_module.c"

. auto/module

ngx_addon_name=$ngx_module_name
```

The `config` file is a POSIX shell script that can set and access the following variables:

​	`config` 文件是一个 POSIX shell 脚本，可以设置和访问以下变量：

- `ngx_module_type` — Type of module to build. Possible values are `CORE`, `HTTP`, `HTTP_FILTER`, `HTTP_INIT_FILTER`, `HTTP_AUX_FILTER`, `MAIL`, `STREAM`, or `MISC`.

- `ngx_module_type` — 要构建的模块类型。可能的值是 `CORE`、`HTTP`、`HTTP_FILTER`、`HTTP_INIT_FILTER`、`HTTP_AUX_FILTER`、`MAIL`、`STREAM` 或 `MISC`。

- `ngx_module_name` — Module names. To build multiple modules from a set of source files, specify a whitespace-separated list of names. The first name indicates the name of the output binary for the dynamic module. The names in the list must match the names used in the source code.

- `ngx_module_name` — 模块名称。要从一组源文件构建多个模块，请指定一个以空格分隔的名称列表。第一个名称表示动态模块的输出二进制文件名。列表中的名称必须与源代码中使用的名称匹配。

- `ngx_addon_name` — Name of the module as it appears in output on the console from the configure script.

- `ngx_addon_name` — 模块名称，显示在配置脚本的控制台输出中。

- `ngx_module_srcs` — Whitespace-separated list of source files used to compile the module. The `$ngx_addon_dir` variable can be used to represent the path to the module directory.

- `ngx_module_srcs` — 以空格分隔的用于编译模块的源文件列表。可以使用 `$ngx_addon_dir` 变量表示模块目录的路径。

- `ngx_module_incs` — Include paths required to build the module

- `ngx_module_incs` — 构建模块所需的包含路径。

- `ngx_module_deps` — Whitespace-separated list of the module's dependencies. Usually, it is the list of header files.

- `ngx_module_deps` — 模块的依赖项列表，以空格分隔。通常是头文件列表。

- `ngx_module_libs` — Whitespace-separated list of libraries to link with the module. For example, use `ngx_module_libs=-lpthread` to link `libpthread` library. The following macros can be used to link against the same libraries as nginx: `LIBXSLT`, `LIBGD`, `GEOIP`, `PCRE`, `OPENSSL`, `MD5`, `SHA1`, `ZLIB`, and `PERL`.

- `ngx_module_libs` — 要与模块链接的库的空格分隔列表。例如，使用 `ngx_module_libs=-lpthread` 链接 `libpthread` 库。可以使用以下宏来与 nginx 链接相同的库：`LIBXSLT`、`LIBGD`、`GEOIP`、`PCRE`、`OPENSSL`、`MD5`、`SHA1`、`ZLIB` 和 `PERL`。

- `ngx_module_link` — Variable set by the build system to `DYNAMIC` for a dynamic module or `ADDON` for a static module and used to determine different actions to perform depending on linking type.

- `ngx_module_link` — 由构建系统设置为 `DYNAMIC` 以用于动态模块，或设置为 `ADDON` 以用于静态模块，用于根据链接类型执行不同的操作。

- `ngx_module_order` — Load order for the module; useful for the `HTTP_FILTER` and `HTTP_AUX_FILTER` module types. The format for this option is a whitespace-separated list of modules. All modules in the list following the current module's name end up after it in the global list of modules, which sets up the order for modules initialization. For filter modules later initialization means earlier execution.

- `ngx_module_order` — 模块的加载顺序，对于 `HTTP_FILTER` 和 `HTTP_AUX_FILTER` 模块类型很有用。此选项的格式为以空格分隔的模块列表。列表中当前模块名后的所有模块都会在全局模块列表中排在它后面，从而设置了模块初始化的顺序。对于过滤器模块，较晚的初始化意味着更早的执行。

  The following modules are typically used as references. The `ngx_http_copy_filter_module` reads the data for other filter modules and is placed near the bottom of the list so that it is one of the first to be executed. The `ngx_http_write_filter_module` writes the data to the client socket and is placed near the top of the list, and is the last to be executed.

  ​	以下模块通常用作参考。`ngx_http_copy_filter_module` 读取其他过滤器模块的数据，因此它位于列表底部，以便在它之后执行。`ngx_http_write_filter_module` 将数据写入客户端套接字，并位于列表顶部，是最后一个执行的模块。
  
  By default, filter modules are placed before the `ngx_http_copy_filter` in the module list so that the filter handler is executed after the copy filter handler. For other module types the default is the empty string.
  
  ​	默认情况下，过滤器模块放置在全局模块列表中 `ngx_http_copy_filter` 之前，以便在复制过滤器处理程序之后执行过滤器处理程序。对于其他模块类型，默认值为空字符串。

To compile a module into nginx statically, use the `--add-module=/path/to/module` argument to the configure script. To compile a module for later dynamic loading into nginx, use the `--add-dynamic-module=/path/to/module` argument.

​	要将模块静态地编译到 nginx 中，可以使用配置脚本的 `--add-module=/path/to/module` 参数。要将模块编译为稍后动态加载到 nginx 中，可以使用配置脚本的 `--add-dynamic-module=/path/to/module` 参数。



### 核心模块 Core Modules

Modules are the building blocks of nginx, and most of its functionality is implemented as modules. The module source file must contain a global variable of type `ngx_module_t`, which is defined as follows:

​	模块是 nginx 的构建模块，其大部分功能都作为模块实现。模块源文件必须包含一个类型为 `ngx_module_t` 的全局变量，其定义如下：

```
struct ngx_module_s {

    /* private part is omitted */
    /* 私有部分被省略 */

    void                 *ctx;
    ngx_command_t        *commands;
    ngx_uint_t            type;

    ngx_int_t           (*init_master)(ngx_log_t *log);

    ngx_int_t           (*init_module)(ngx_cycle_t *cycle);

    ngx_int_t           (*init_process)(ngx_cycle_t *cycle);
    ngx_int_t           (*init_thread)(ngx_cycle_t *cycle);
    void                (*exit_thread)(ngx_cycle_t *cycle);
    void                (*exit_process)(ngx_cycle_t *cycle);

    void                (*exit_master)(ngx_cycle_t *cycle);

    /* stubs for future extensions are omitted */
    /* 未来扩展的存根被省略 */
};
```

The omitted private part includes the module version and a signature and is filled using the predefined macro `NGX_MODULE_V1`.

​	省略的私有部分包括模块版本和签名，并使用预定义的宏 `NGX_MODULE_V1` 进行填充。

Each module keeps its private data in the `ctx` field, recognizes the configuration directives, specified in the `commands` array, and can be invoked at certain stages of nginx lifecycle. The module lifecycle consists of the following events:

​	每个模块将其私有数据保留在 `ctx` 字段中，识别在 `commands` 数组中指定的配置指令，并且可以在 nginx 生命周期的特定阶段调用。模块生命周期包括以下事件：

- Configuration directive handlers are called as they appear in configuration files in the context of the master process.
- 配置指令处理程序按照它们在配置文件中出现的顺序在主进程的上下文中调用。
- After the configuration is parsed successfully, `init_module` handler is called in the context of the master process. The `init_module` handler is called in the master process each time a configuration is loaded.
- 成功解析配置后，`init_module` 处理程序在主进程的上下文中调用。每次加载配置时，都会在主进程中调用 `init_module` 处理程序。
- The master process creates one or more worker processes and the `init_process` handler is called in each of them.
- 主进程创建一个或多个工作进程，并在每个工作进程中调用 `init_process` 处理程序。
- When a worker process receives the shutdown or terminate command from the master, it invokes the `exit_process` handler.
- 当工作进程从主进程接收到关闭或终止命令时，它会调用 `exit_process` 处理程序。
- The master process calls the `exit_master` handler before exiting.
- 主进程在退出前调用 `exit_master` 处理程序。

Because threads are used in nginx only as a supplementary I/O facility with its own API, `init_thread` and `exit_thread` handlers are not currently called. There is also no `init_master` handler, because it would be unnecessary overhead.

​	由于在 nginx 中仅将线程用作具有自己 API 的补充 I/O 设施，`init_thread` 和 `exit_thread` 处理程序目前不会被调用。也没有 `init_master` 处理程序，因为它会产生不必要的开销。

The module `type` defines exactly what is stored in the `ctx` field. Its value is one of the following types:

​	模块 `type` 正好定义了存储在 `ctx` 字段中的内容。其值为以下类型之一：

- `NGX_CORE_MODULE`
- `NGX_EVENT_MODULE`
- `NGX_HTTP_MODULE`
- `NGX_MAIL_MODULE`
- `NGX_STREAM_MODULE`

The `NGX_CORE_MODULE` is the most basic and thus the most generic and most low-level type of module. The other module types are implemented on top of it and provide a more convenient way to deal with corresponding domains, like handling events or HTTP requests.

​	`NGX_CORE_MODULE` 是最基本、最通用和最底层的模块类型。其他模块类型都在其上实现，并提供了处理事件或 HTTP 请求等相应领域的更方便的方式。

The set of core modules includes `ngx_core_module`, `ngx_errlog_module`, `ngx_regex_module`, `ngx_thread_pool_module` and `ngx_openssl_module` modules. The HTTP module, the stream module, the mail module and event modules are core modules too. The context of a core module is defined as:

​	核心模块的集合包括 `ngx_core_module`、`ngx_errlog_module`、`ngx_regex_module`、`ngx_thread_pool_module` 和 `ngx_openssl_module` 模块。HTTP 模块、流模块、邮件模块和事件模块也是核心模块。核心模块的上下文定义如下：

```
typedef struct {
    ngx_str_t             name;
    void               *(*create_conf)(ngx_cycle_t *cycle);
    char               *(*init_conf)(ngx_cycle_t *cycle, void *conf);
} ngx_core_module_t;
```

where the `name` is a module name string, `create_conf` and `init_conf` are pointers to functions that create and initialize module configuration respectively. For core modules, nginx calls `create_conf` before parsing a new configuration and `init_conf` after all configuration is parsed successfully. The typical `create_conf` function allocates memory for the configuration and sets default values.

其中 `name` 是模块名称字符串，`create_conf` 和 `init_conf` 是指向分别创建和初始化模块配置的函数的指针。对于核心模块，nginx 在解析新配置之前调用 `create_conf`，在成功解析所有配置之后调用 `init_conf`。典型的 `create_conf` 函数为配置分配内存并设置默认值。

For example, a simplistic module called `ngx_foo_module` might look like this:

​	例如，一个名为 `ngx_foo_module` 的简单模块可能如下所示：

```
/*
 * Copyright (C) Author.
 */


#include <ngx_config.h>
#include <ngx_core.h>


typedef struct {
    ngx_flag_t  enable;
} ngx_foo_conf_t;


static void *ngx_foo_create_conf(ngx_cycle_t *cycle);
static char *ngx_foo_init_conf(ngx_cycle_t *cycle, void *conf);

static char *ngx_foo_enable(ngx_conf_t *cf, void *post, void *data);
static ngx_conf_post_t  ngx_foo_enable_post = { ngx_foo_enable };


static ngx_command_t  ngx_foo_commands[] = {

    { ngx_string("foo_enabled"),
      NGX_MAIN_CONF|NGX_DIRECT_CONF|NGX_CONF_FLAG,
      ngx_conf_set_flag_slot,
      0,
      offsetof(ngx_foo_conf_t, enable),
      &ngx_foo_enable_post },

      ngx_null_command
};


static ngx_core_module_t  ngx_foo_module_ctx = {
    ngx_string("foo"),
    ngx_foo_create_conf,
    ngx_foo_init_conf
};


ngx_module_t  ngx_foo_module = {
    NGX_MODULE_V1,
    &ngx_foo_module_ctx,                   /* module context */ /* 模块上下文 */
    ngx_foo_commands,                      /* module directives */ /* 模块指令 */
    NGX_CORE_MODULE,                       /* module type */ /* 模块类型 */
    NULL,                                  /* init master */
    NULL,                                  /* init module */
    NULL,                                  /* init process */
    NULL,                                  /* init thread */
    NULL,                                  /* exit thread */
    NULL,                                  /* exit process */
    NULL,                                  /* exit master */
    NGX_MODULE_V1_PADDING
};


static void *
ngx_foo_create_conf(ngx_cycle_t *cycle)
{
    ngx_foo_conf_t  *fcf;

    fcf = ngx_pcalloc(cycle->pool, sizeof(ngx_foo_conf_t));
    if (fcf == NULL) {
        return NULL;
    }

    fcf->enable = NGX_CONF_UNSET;

    return fcf;
}


static char *
ngx_foo_init_conf(ngx_cycle_t *cycle, void *conf)
{
    ngx_foo_conf_t *fcf = conf;

    ngx_conf_init_value(fcf->enable, 0);

    return NGX_CONF_OK;
}


static char *
ngx_foo_enable(ngx_conf_t *cf, void *post, void *data)
{
    ngx_flag_t  *fp = data;

    if (*fp == 0) {
        return NGX_CONF_OK;
    }

    ngx_log_error(NGX_LOG_NOTICE, cf->log, 0, "Foo Module is enabled");

    return NGX_CONF_OK;
}
```





### 配置指令 Configuration Directives

The `ngx_command_t` type defines a single configuration directive. Each module that supports configuration provides an array of such structures that describe how to process arguments and what handlers to call:

​	`ngx_command_t` 类型定义了单个配置指令。每个支持配置的模块都提供了一个这样的结构数组，描述如何处理参数以及调用什么处理程序：

```c
typedef struct ngx_command_s  ngx_command_t;

struct ngx_command_s {
    ngx_str_t             name;
    ngx_uint_t            type;
    char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
    ngx_uint_t            conf;
    ngx_uint_t            offset;
    void                 *post;
};
```

Terminate the array with the special value `ngx_null_command`. The `name` is the name of a directive as it appears in the configuration file, for example "worker_processes" or "listen". The `type` is a bit-field of flags that specify the number of arguments the directive takes, its type, and the context in which it appears. The flags are:

​	使用特殊值 `ngx_null_command` 来终止数组。`name` 是指令在配置文件中的名称，例如 "worker_processes" 或 "listen"。`type` 是一个位字段，指定指令的参数个数、类型以及出现的上下文。标志包括：

- `NGX_CONF_NOARGS` — Directive takes no arguments.
- `NGX_CONF_1MORE` — Directive takes one or more arguments.
- `NGX_CONF_2MORE` — Directive takes two or more arguments.
- `NGX_CONF_TAKE1`..`NGX_CONF_TAKE7` — Directive takes exactly the indicated number of arguments.
- `NGX_CONF_TAKE12`, `NGX_CONF_TAKE13`, `NGX_CONF_TAKE23`, `NGX_CONF_TAKE123`, `NGX_CONF_TAKE1234` — Directive may take different number of arguments. Options are limited to the given numbers. For example, `NGX_CONF_TAKE12` means it takes one or two arguments.
- `NGX_CONF_NOARGS` — 指令不带参数。
- `NGX_CONF_1MORE` — 指令带一个或更多参数。
- `NGX_CONF_2MORE` — 指令带两个或更多参数。
- `NGX_CONF_TAKE1`..`NGX_CONF_TAKE7` — 指令带恰好指定数量的参数。
- `NGX_CONF_TAKE12`、`NGX_CONF_TAKE13`、`NGX_CONF_TAKE23`、`NGX_CONF_TAKE123`、`NGX_CONF_TAKE1234` — 指令可以带不同数量的参数。选项被限制为给定的数量。例如，`NGX_CONF_TAKE12` 表示带一个或两个参数。

The flags for directive types are:

​	指令类型的标志包括：

- `NGX_CONF_BLOCK` — Directive is a block, that is, it can contain other directives within its opening and closing braces, or even implement its own parser to handle contents inside.
- `NGX_CONF_BLOCK` — 指令是一个块，即它可以包含其他指令在其大括号内，甚至可以实现自己的解析器来处理内部内容。
- `NGX_CONF_FLAG` — Directive takes a boolean value, either `on` or `off`.
- `NGX_CONF_FLAG` — 指令接受一个布尔值，要么是 `on`，要么是 `off`。

A directive's context defines where it may appear in the configuration:

​	指令的上下文定义了它可以在配置中的何处出现：

- `NGX_MAIN_CONF` — In the top level context.
- `NGX_HTTP_MAIN_CONF` — In the `http` block.
- `NGX_HTTP_SRV_CONF` — In a `server` block within the `http` block.
- `NGX_HTTP_LOC_CONF` — In a `location` block within the `http` block.
- `NGX_HTTP_UPS_CONF` — In an `upstream` block within the `http` block.
- `NGX_HTTP_SIF_CONF` — In an `if` block within a `server` block in the `http` block.
- `NGX_HTTP_LIF_CONF` — In an `if` block within a `location` block in the `http` block.
- `NGX_HTTP_LMT_CONF` — In a `limit_except` block within the `http` block.
- `NGX_STREAM_MAIN_CONF` — In the `stream` block.
- `NGX_STREAM_SRV_CONF` — In a `server` block within the `stream` block.
- `NGX_STREAM_UPS_CONF` — In an `upstream` block within the `stream` block.
- `NGX_MAIL_MAIN_CONF` — In the `mail` block.
- `NGX_MAIL_SRV_CONF` — In a `server` block within the `mail` block.
- `NGX_EVENT_CONF` — In the `event` block.
- `NGX_DIRECT_CONF` — Used by modules that don't create a hierarchy of contexts and only have one global configuration. This configuration is passed to the handler as the `conf` argument.
- `NGX_MAIN_CONF` — 位于顶级上下文。
- `NGX_HTTP_MAIN_CONF` — 位于 `http` 块中。
- `NGX_HTTP_SRV_CONF` — 位于 `http` 块内的 `server` 块中。
- `NGX_HTTP_LOC_CONF` — 位于 `http` 块内的 `location` 块中。
- `NGX_HTTP_UPS_CONF` — 位于 `http` 块内的 `upstream` 块中。
- `NGX_HTTP_SIF_CONF` — 位于 `http` 块内的 `server` 块中的 `if` 块中。
- `NGX_HTTP_LIF_CONF` — 位于 `http` 块内的 `location` 块中的 `if` 块中。
- `NGX_HTTP_LMT_CONF` — 位于 `http` 块内的 `limit_except` 块中。
- `NGX_STREAM_MAIN_CONF` — 位于 `stream` 块中。
- `NGX_STREAM_SRV_CONF` — 位于 `stream` 块内的 `server` 块中。
- `NGX_STREAM_UPS_CONF` — 位于 `stream` 块内的 `upstream` 块中。
- `NGX_MAIL_MAIN_CONF` — 位于 `mail` 块中。
- `NGX_MAIL_SRV_CONF` — 位于 `mail` 块内的 `server` 块中。
- `NGX_EVENT_CONF` — 位于 `event` 块中。
- `NGX_DIRECT_CONF` — 由不创建上下文层次结构且只具有一个全局配置的模块使用。该配置作为 `conf` 参数传递给处理程序。

The configuration parser uses these flags to throw an error in case of a misplaced directive and calls directive handlers supplied with a proper configuration pointer, so that the same directives in different locations can store their values in distinct places.

​	配置解析器使用这些标志在放置的指令出现错误时引发错误，并调用指令处理程序，同时提供适当的配置指针，使得不同位置的相同指令可以将其值存储在不同的位置。

The `set` field defines a handler that processes a directive and stores parsed values into the corresponding configuration. There's a number of functions that perform common conversions:

​	`set` 字段定义了处理指令并将解析的值存储到相应配置中的处理程序。有一些函数执行常见的转换：

- `ngx_conf_set_flag_slot` — Converts the literal strings `on` and `off` into an `ngx_flag_t` value with values 1 or 0, respectively.
- `ngx_conf_set_flag_slot` — 将字面字符串 `on` 和 `off` 转换为 `ngx_flag_t` 值，分别为 1 或 0。
- `ngx_conf_set_str_slot` — Stores a string as a value of the `ngx_str_t` type.
- `ngx_conf_set_str_slot` — 将字符串存储为 `ngx_str_t` 类型的值。
- `ngx_conf_set_str_array_slot` — Appends a value to an array `ngx_array_t` of strings `ngx_str_t`. The array is created if does not already exist.
- `ngx_conf_set_str_array_slot` — 将值追加到字符串 `ngx_str_t` 数组 `ngx_array_t` 中。如果数组不存在，则会创建它。
- `ngx_conf_set_keyval_slot` — Appends a key-value pair to an array `ngx_array_t` of key-value pairs `ngx_keyval_t`. The first string becomes the key and the second the value. The array is created if it does not already exist.
- `ngx_conf_set_keyval_slot` — 将键值对追加到键值对数组 `ngx_keyval_t` 中的数组 `ngx_array_t`。第一个字符串成为键，第二个成为值。如果数组不存在，则会创建它。
- `ngx_conf_set_num_slot` — Converts a directive's argument to an `ngx_int_t` value.
- `ngx_conf_set_num_slot` — 将指令的参数转换为 `ngx_int_t` 值。
- `ngx_conf_set_size_slot` — Converts a [size](https://nginx.org/en/docs/syntax.html) to a `size_t` value expressed in bytes.
- `ngx_conf_set_size_slot` — 将 [size](https://nginx.org/en/docs/syntax.html) 转换为以字节为单位的 `size_t` 值。
- `ngx_conf_set_off_slot` — Converts an [offset](https://nginx.org/en/docs/syntax.html) to an `off_t` value expressed in bytes.
- `ngx_conf_set_off_slot` — 将 [offset](https://nginx.org/en/docs/syntax.html) 转换为以字节为单位的 `off_t` 值。
- `ngx_conf_set_msec_slot` — Converts a [time](https://nginx.org/en/docs/syntax.html) to an `ngx_msec_t` value expressed in milliseconds.
- `ngx_conf_set_msec_slot` — 将 [time](https://nginx.org/en/docs/syntax.html) 转换为以毫秒为单位的 `ngx_msec_t` 值。
- `ngx_conf_set_sec_slot` — Converts a [time](https://nginx.org/en/docs/syntax.html) to a `time_t` value expressed in in seconds.
- `ngx_conf_set_sec_slot` — 将 [time](https://nginx.org/en/docs/syntax.html) 转换为以秒为单位的 `time_t` 值。
- `ngx_conf_set_bufs_slot` — Converts the two supplied arguments into an `ngx_bufs_t` object that holds the number and [size](https://nginx.org/en/docs/syntax.html) of buffers.
- `ngx_conf_set_bufs_slot` — 将两个提供的参数转换为包含缓冲区数和 [size](https://nginx.org/en/docs/syntax.html) 的 `ngx_bufs_t` 对象。
- `ngx_conf_set_enum_slot` — Converts the supplied argument into an `ngx_uint_t` value. The null-terminated array of `ngx_conf_enum_t` passed in the `post` field defines the acceptable strings and corresponding integer values.
- `ngx_conf_set_enum_slot` — 将提供的参数转换为 `ngx_uint_t` 值。在 `post` 字段中传递的 `ngx_conf_enum_t` 的空结尾数组定义了可接受的字符串和相应的整数值。
- `ngx_conf_set_bitmask_slot` — Converts the supplied arguments into an `ngx_uint_t` value. The mask values for each argument are ORed producing the result. The null-terminated array of `ngx_conf_bitmask_t` passed in the `post` field defines the acceptable strings and corresponding mask values.
- `ngx_conf_set_bitmask_slot` — 将提供的参数转换为 `ngx_uint_t` 值。为每个参数的掩码值执行 OR 操作，生成结果。在 `post` 字段中传递的 `ngx_conf_bitmask_t` 的空结尾数组定义了可接受的字符串和相应的掩码值。
- `set_path_slot` — Converts the supplied arguments to an `ngx_path_t` value and performs all required initializations. For details, see the documentation for the [proxy_temp_path](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_temp_path) directive.
- `set_path_slot` — 将提供的参数转换为 `ngx_path_t` 值，并执行所有必要的初始化。有关详细信息，请参阅 [proxy_temp_path](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_temp_path) 指令的文档。
- `set_access_slot` — Converts the supplied arguments to a file permissions mask. For details, see the documentation for the [proxy_store_access](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_store_access) directive.
- `set_access_slot` — 将提供的参数转换为文件权限掩码。有关详细信息，请参阅 [proxy_store_access](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_store_access) 指令的文档。



The `conf` field defines which configuration structure is passed to the directory handler. Core modules only have the global configuration and set `NGX_DIRECT_CONF` flag to access it. Modules like HTTP, Stream or Mail create hierarchies of configurations. For example, a module's configuration is created for `server`, `location` and `if` scopes.

​	`conf` 字段定义了传递给指令处理程序的配置结构。核心模块只有全局配置，并设置 `NGX_DIRECT_CONF` 标志来访问它。HTTP、Stream 或 Mail 等模块创建配置的层次结构。例如，模块的配置会针对 `server`、`location` 和 `if` 作用域创建。

- `NGX_HTTP_MAIN_CONF_OFFSET` — Configuration for the `http` block.
- `NGX_HTTP_MAIN_CONF_OFFSET` — `http` 块的配置。
- `NGX_HTTP_SRV_CONF_OFFSET` — Configuration for a `server` block within the `http` block.
- `NGX_HTTP_SRV_CONF_OFFSET` — `http` 块内的 `server` 块的配置。
- `NGX_HTTP_LOC_CONF_OFFSET` — Configuration for a `location` block within the `http`.
- `NGX_HTTP_LOC_CONF_OFFSET` — `http` 块内的 `location` 块的配置。
- `NGX_STREAM_MAIN_CONF_OFFSET` — Configuration for the `stream` block.
- `NGX_STREAM_MAIN_CONF_OFFSET` — `stream` 块的配置。
- `NGX_STREAM_SRV_CONF_OFFSET` — Configuration for a `server` block within the `stream` block.
- `NGX_STREAM_SRV_CONF_OFFSET` — `stream` 块内的 `server` 块的配置。
- `NGX_MAIL_MAIN_CONF_OFFSET` — Configuration for the `mail` block.
- `NGX_MAIL_MAIN_CONF_OFFSET` — `mail` 块的配置。
- `NGX_MAIL_SRV_CONF_OFFSET` — Configuration for a `server` block within the `mail` block.
- `NGX_MAIL_SRV_CONF_OFFSET` — `mail` 块内的 `server` 块的配置。



The `offset` defines the offset of a field in a module configuration structure that holds values for this particular directive. The typical use is to employ the `offsetof()` macro.

​	`offset` 定义了模块配置结构中某个字段的偏移量，该字段保存特定指令的值。通常使用 `offsetof()` 宏来进行偏移。

The `post` field has two purposes: it may be used to define a handler to be called after the main handler has completed, or to pass additional data to the main handler. In the first case, the `ngx_conf_post_t` structure needs to be initialized with a pointer to the handler, for example:

​	`post` 字段有两个用途：它可以用于定义在主处理程序完成后要调用的处理程序，或者用于向主处理程序传递附加数据。在第一种情况下，需要使用指向处理程序的指针来初始化 `ngx_conf_post_t` 结构，例如：

```
static char *ngx_do_foo(ngx_conf_t *cf, void *post, void *data);
static ngx_conf_post_t  ngx_foo_post = { ngx_do_foo };
```

The `post` argument is the `ngx_conf_post_t` object itself, and the `data` is a pointer to the value, converted from arguments by the main handler with the appropriate type.

​	`post` 参数是 `ngx_conf_post_t` 对象本身，而 `data` 是通过主处理程序以适当类型转换从参数转换的值的指针。



## HTTP



### 连接 Connection

Each HTTP client connection runs through the following stages:

​	每个 HTTP 客户端连接经过以下阶段：

- `ngx_event_accept()` accepts a client TCP connection. This handler is called in response to a read notification on a listen socket. A new `ngx_connection_t` object is created at this stage to wrap the newly accepted client socket. Each nginx listener provides a handler to pass the new connection object to. For HTTP connections it's `ngx_http_init_connection(c)`.
- `ngx_event_accept()` 接受客户端 TCP 连接。此处理程序在监听套接字上的读通知的响应中调用。在这个阶段创建一个新的 `ngx_connection_t` 对象，用于包装新接受的客户端套接字。每个 nginx 监听器都提供一个处理程序来将新的连接对象传递给它。对于 HTTP 连接，是 `ngx_http_init_connection(c)`。
- `ngx_http_init_connection()` performs early initialization of the HTTP connection. At this stage an `ngx_http_connection_t` object is created for the connection and its reference is stored in the connection's `data` field. Later it will be replaced by an HTTP request object. A PROXY protocol parser and the SSL handshake are started at this stage as well.
- `ngx_http_init_connection()` 执行 HTTP 连接的早期初始化。在这个阶段，为连接创建一个 `ngx_http_connection_t` 对象，并将其引用存储在连接的 `data` 字段中。稍后，它将被一个 HTTP 请求对象替换。在这个阶段还会启动 PROXY 协议解析器和 SSL 握手。
- `ngx_http_wait_request_handler()` read event handler is called when data is available on the client socket. At this stage an HTTP request object `ngx_http_request_t` is created and set to the connection's `data` field.
- `ngx_http_wait_request_handler()` 在客户端套接字上有数据可用时调用读事件处理程序。在这个阶段，会创建一个 HTTP 请求对象 `ngx_http_request_t`，并将其设置为连接的 `data` 字段。
- `ngx_http_process_request_line()` read event handler reads client request line. The handler is set by `ngx_http_wait_request_handler()`. The data is read into connection's `buffer`. The size of the buffer is initially set by the directive [client_header_buffer_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_header_buffer_size). The entire client header is supposed to fit in the buffer. If the initial size is not sufficient, a bigger buffer is allocated, with the capacity set by the `large_client_header_buffers` directive.
- `ngx_http_process_request_line()` 读事件处理程序读取客户端请求行。这个处理程序由 `ngx_http_wait_request_handler()` 设置。数据被读入连接的 `buffer` 中。缓冲区的大小最初由 [client_header_buffer_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_header_buffer_size) 指令设置。整个客户端头部应该适合于缓冲区中。如果初始大小不够，将分配一个更大的缓冲区，其容量由 `large_client_header_buffers` 指令设置。
- `ngx_http_process_request_headers()` read event handler, is set after `ngx_http_process_request_line()` to read the client request header.
- `ngx_http_process_request_headers()` 读事件处理程序，由 `ngx_http_process_request_line()` 之后设置，用于读取客户端请求头。
- `ngx_http_core_run_phases()` is called when the request header is completely read and parsed. This function runs request phases from `NGX_HTTP_POST_READ_PHASE` to `NGX_HTTP_CONTENT_PHASE`. The last phase is intended to generate a response and pass it along the filter chain. The response is not necessarily sent to the client at this phase. It might remain buffered and be sent at the finalization stage.
- `ngx_http_core_run_phases()` 在请求头完全读取和解析后调用。此函数从 `NGX_HTTP_POST_READ_PHASE` 执行到 `NGX_HTTP_CONTENT_PHASE`。最后一个阶段用于生成响应并将其传递给过滤器链。响应不一定会在此阶段发送给客户端。它可能会保留缓冲并在最终阶段发送。
- `ngx_http_finalize_request()` is usually called when the request has generated all the output or produced an error. In the latter case an appropriate error page is looked up and used as the response. If the response is not completely sent to the client by this point, an HTTP writer `ngx_http_writer()` is activated to finish sending outstanding data.
- `ngx_http_finalize_request()` 通常在请求生成所有输出或产生错误时调用。在后一种情况下，会查找适当的错误页面并用作响应。如果到此时响应尚未完全发送到客户端，则会激活 HTTP writer `ngx_http_writer()` 来完成发送未完成的数据。
- `ngx_http_finalize_connection()` is called when the complete response has been sent to the client and the request can be destroyed. If the client connection keepalive feature is enabled, `ngx_http_set_keepalive()` is called, which destroys the current request and waits for the next request on the connection. Otherwise, `ngx_http_close_request()` destroys both the request and the connection.
- `ngx_http_finalize_connection()` 在完整的响应已发送到客户端并可以销毁请求时调用。如果启用了客户端连接保持活动功能，将调用 `ngx_http_set_keepalive()`，它会销毁当前请求并等待连接上的下一个请求。否则，`ngx_http_close_request()` 会同时销毁请求和连接。



### 请求 Request

For each client HTTP request the `ngx_http_request_t` object is created. Some of the fields of this object are:

​	对于每个客户端 HTTP 请求，都会创建一个 `ngx_http_request_t` 对象。这个对象的一些字段如下：

- `connection` — Pointer to a `ngx_connection_t` client connection object. Several requests can reference the same connection object at the same time - one main request and its subrequests. After a request is deleted, a new request can be created on the same connection.

- `connection` — 指向 `ngx_connection_t` 客户端连接对象的指针。同一时间可以有多个请求引用同一个连接对象 - 一个主请求和其子请求。删除请求后，可以在同一个连接上创建新请求。

  Note that for HTTP connections `ngx_connection_t`'s `data` field points back to the request. Such requests are called active, as opposed to the other requests tied to the connection. An active request is used to handle client connection events and is allowed to output its response to the client. Normally, each request becomes active at some point so that it can send its output.

  注意，对于 HTTP 连接，`ngx_connection_t` 的 `data` 字段指回请求。这种请求称为活动请求，与连接上的其他请求相对。活动请求用于处理客户端连接事件，并允许将其响应输出给客户端。通常，每个请求在某个时候都会变成活动状态，以便它可以发送其输出。

- `ctx` — Array of HTTP module contexts. Each module of type `NGX_HTTP_MODULE` can store any value (normally, a pointer to a structure) in the request. The value is stored in the `ctx` array at the module's `ctx_index` position. The following macros provide a convenient way to get and set request contexts:

- `ctx` — HTTP 模块上下文的数组。每个类型为 `NGX_HTTP_MODULE` 的模块可以在请求中存储任何值（通常是指向结构的指针）。该值存储在模块的 `ctx` 数组中的模块的 `ctx_index` 位置。以下宏提供了一种方便的方法来获取和设置请求上下文：

  - `ngx_http_get_module_ctx(r, module)` — Returns the `module`'s context
  - `ngx_http_get_module_ctx(r, module)` — 返回 `module` 的上下文
  - `ngx_http_set_ctx(r, c, module)` — Sets `c` as the `module`'s context
  - `ngx_http_set_ctx(r, c, module)` — 将 `c` 设置为 `module` 的上下文

- `main_conf`, `srv_conf`, `loc_conf` — Arrays of current request configurations. Configurations are stored at the module's `ctx_index` positions.

- `main_conf`、`srv_conf`、`loc_conf` — 当前请求配置的数组。配置存储在模块的 `ctx_index` 位置。

- `read_event_handler`, `write_event_handler` - Read and write event handlers for the request. Normally, both the read and write event handlers for an HTTP connection are set to `ngx_http_request_handler()`. This function calls the `read_event_handler` and `write_event_handler` handlers for the currently active request.

- `read_event_handler`、`write_event_handler` — 请求的读取和写入事件处理程序。通常，HTTP 连接的读写事件处理程序都设置为 `ngx_http_request_handler()`。这个函数调用当前活动请求的 `read_event_handler` 和 `write_event_handler` 处理程序。

- `cache` — Request cache object for caching the upstream response.

- `cache` — 用于缓存上游响应的请求缓存对象。

- `upstream` — Request upstream object for proxying.

- `upstream` — 用于代理的请求上游对象。

- `pool` — Request pool. The request object itself is allocated in this pool, which is destroyed when the request is deleted. For allocations that need to be available throughout the client connection's lifetime, use `ngx_connection_t`'s pool instead.

- `pool` — 请求池。请求对象本身在这个池中分配，当请求被删除时，这个池将被销毁。对于需要在客户端连接的整个生命周期中可用的分配，使用 `ngx_connection_t` 的池。

- `header_in` — Buffer into which the client HTTP request header is read.

- `header_in` — 用于读取客户端 HTTP 请求头的缓冲区。

- `headers_in`, `headers_out` — Input and output HTTP headers objects. Both objects contain the `headers` field of type `ngx_list_t` for keeping the raw list of headers. In addition to that, specific headers are available for getting and setting as separate fields, for example `content_length_n`, `status` etc.

- `headers_in`、`headers_out` — 输入和输出的 HTTP 头对象。两个对象都包含 `ngx_list_t` 类型的 `headers` 字段，用于保存原始头的列表。除此之外，还有特定的头可用于单独获取和设置，例如 `content_length_n`、`status` 等。

- `request_body` — Client request body object.

- `request_body` — 客户端请求主体对象。

- `start_sec`, `start_msec` — Time point when the request was created, used for tracking request duration.

- `start_sec`、`start_msec` — 请求创建的时间点，用于跟踪请求持续时间。

- `method`, `method_name` — Numeric and text representation of the client HTTP request method. Numeric values for methods are defined in `src/http/ngx_http_request.h` with the macros `NGX_HTTP_GET`, `NGX_HTTP_HEAD`, `NGX_HTTP_POST`, etc.

- `method`、`method_name` — 客户端 HTTP 请求方法的数值和文本表示。方法的数值值在 `src/http/ngx_http_request.h` 中用宏 `NGX_HTTP_GET`、`NGX_HTTP_HEAD`、`NGX_HTTP_POST` 等定义。

- `http_protocol` — Client HTTP protocol version in its original text form (“HTTP/1.0”, “HTTP/1.1” etc).

- `http_protocol` — 客户端 HTTP 协议版本的原始文本形式（如 “HTTP/1.0”、“HTTP/1.1” 等）。

- `http_version` — Client HTTP protocol version in numeric form (`NGX_HTTP_VERSION_10`, `NGX_HTTP_VERSION_11`, etc.).

- `http_version` — 客户端 HTTP 协议版本的数值形式（`NGX_HTTP_VERSION_10`、`NGX_HTTP_VERSION_11` 等）。

- `http_major`, `http_minor` — Client HTTP protocol version in numeric form split into major and minor parts.

- `http_major`、`http_minor` — 数值形式的客户端 HTTP 协议版本，拆分为主版本和次版本。

- `request_line`, `unparsed_uri` — Request line and URI in the original client request.

- `request_line`、`unparsed_uri` — 客户端请求的请求行和 URI 的原始客户端请求。

- `uri`, `args`, `exten` — URI, arguments and file extension for the current request. The URI value here might differ from the original URI sent by the client due to normalization. Throughout request processing, these values can change as internal redirects are performed.

- `uri`、`args`、`exten` — 当前请求的 URI、参数和文件扩展名。这里的 URI 值可能与客户端发送的原始 URI 不同，因为进行了规范化。在请求处理期间，这些值可能会因执行内部重定向而发生变化。

- `main` — Pointer to a main request object. This object is created to process a client HTTP request, as opposed to subrequests, which are created to perform a specific subtask within the main request.

- `main` — 指向主请求对象的指针。这个对象被创建来处理客户端 HTTP 请求，与子请求相对，后者是为了在主请求内执行特定子任务而创建的。

- `parent` — Pointer to the parent request of a subrequest.

- `parent` — 指向子请求的父请求的指针。

- `postponed` — List of output buffers and subrequests, in the order in which they are sent and created. The list is used by the postpone filter to provide consistent request output when parts of it are created by subrequests.

- `postponed` — 输出缓冲区和子请求的列表，按发送和创建的顺序排列。该列表由延迟过滤器使用，以在部分由子请求创建的输出上提供一致的请求输出。

- `post_subrequest` — Pointer to a handler with the context to be called when a subrequest gets finalized. Unused for main requests.

- `post_subrequest` — 指向一个处理程序的指针，该处理程序具有在子请求完成时调用的上下文。对于主请求未使用。

- `posted_requests` — List of requests to be started or resumed, which is done by calling the request's `write_event_handler`. Normally, this handler holds the request main function, which at first runs request phases and then produces the output.

- `posted_requests` — 要启动或恢复的请求列表，通过调用请求的 `write_event_handler` 来完成。通常，此处理程序保存请求的主函数，该函数首先运行请求阶段，然后生成输出。

  A request is usually posted by the `ngx_http_post_request(r, NULL)` call. It is always posted to the main request `posted_requests` list. The function `ngx_http_run_posted_requests(c)` runs all requests that are posted in the main request of the passed connection's active request. All event handlers call `ngx_http_run_posted_requests`, which can lead to new posted requests. Normally, it is called after invoking a request's read or write handler.

  请求通常由 `ngx_http_post_request(r, NULL)` 调用。它始终被发布到主请求的 `posted_requests` 列表中。函数 `ngx_http_run_posted_requests(c)` 运行所有在连接的活动请求的主请求中发布的请求。所有事件处理程序调用 `ngx_http_run_posted_requests`，这可能导致新的发布请求。通常，在调用请求的读取或写入处理程序后，会调用它。

- `phase_handler` — Index of current request phase.

- `phase_handler` — 当前请求阶段的索引。

- `ncaptures`, `captures`, `captures_data` — Regex captures produced by the last regex match of the request. A regex match can occur at a number of places during request processing: map lookup, server lookup by SNI or HTTP Host, rewrite, proxy_redirect, etc. Captures produced by a lookup are stored in the above mentioned fields. The field `ncaptures` holds the number of captures, `captures` holds captures boundaries and `captures_data` holds the string against which the regex was matched and which is used to extract captures. After each new regex match, request captures are reset to hold new values.

- `ncaptures`、`captures`、`captures_data` — 请求的最后一个正则表达式匹配产生的正则表达式捕获。在请求处理过程中，正则表达式匹配可能发生在许多地方：映射查找、SNI 或 HTTP Host 的服务器查找、重写、proxy_redirect 等。由查找产生的捕获存储在上述字段中。字段 `ncaptures` 保存捕获的数量，`captures` 保存捕获的边界，`captures_data` 保存正则表达式匹配的字符串，用于提取捕获。在每次新的正则表达式匹配时，这些字段会被更新。

- `count` — Request reference counter. The field only makes sense for the main request. Increasing the counter is done by simple `r->main->count++`. To decrease the counter, call `ngx_http_finalize_request(r, rc)`. Creating of a subrequest and running the request body read process both increment the counter.

- `count` — 请求引用计数器。该字段仅对主请求有效。增加计数器的方法是通过简单的 `r->main->count++`。要减少计数器，请调用 `ngx_http_finalize_request(r, rc)`。创建子请求和运行请求主体读取过程都会增加计数器。

- `subrequests` — Current subrequest nesting level. Each subrequest inherits its parent's nesting level, decreased by one. An error is generated if the value reaches zero. The value for the main request is defined by the `NGX_HTTP_MAX_SUBREQUESTS` constant.

- `subrequests` — 当前子请求的嵌套级别。每个子请求都继承其父级的嵌套级别，减少一个。如果该值达到零，则会生成错误。主请求的值由常量 `NGX_HTTP_MAX_SUBREQUESTS` 定义。

- `uri_changes` — Number of URI changes remaining for the request. The total number of times a request can change its URI is limited by the `NGX_HTTP_MAX_URI_CHANGES` constant. With each change the value is decremented until it reaches zero, at which time an error is generated. Rewrites and internal redirects to normal or named locations are considered URI changes.

- `uri_changes` — 请求剩余的 URI 更改次数。请求可以更改其 URI 的总次数受到常量 `NGX_HTTP_MAX_URI_CHANGES` 的限制。每次更改时，该值会递减，直到达到零，此时会生成错误。重写和内部重定向到普通或命名位置被视为 URI 更改。

- `blocked` — Counter of blocks held on the request. While this value is non-zero, the request cannot be terminated. Currently, this value is increased by pending AIO operations (POSIX AIO and thread operations) and active cache lock.

- `blocked` — 请求上持有的块计数器。当此值为非零时，请求无法终止。目前，此值会增加挂起的 AIO 操作（POSIX AIO 和线程操作）和活动缓存锁。

- `buffered` — Bitmask showing which modules have buffered the output produced by the request. A number of filters can buffer output; for example, sub_filter can buffer data because of a partial string match, copy filter can buffer data because of the lack of free output buffers etc. As long as this value is non-zero, the request is not finalized pending the flush.

- `buffered` — 位掩码，显示哪些模块已经缓冲了请求产生的输出。许多过滤器可以缓冲输出；例如，由于部分字符串匹配，子过滤器可以缓冲数据，由于缺乏空闲输出缓冲区，复制过滤器可以缓冲数据等。只要这个值为非零，请求就不会在刷新之前终止。

- `header_only` — Flag indicating that the output does not require a body. For example, this flag is used by HTTP HEAD requests.

- `header_only` — 标志，指示输出不需要正文。例如，这个标志在 HTTP HEAD 请求中使用。

- `keepalive` — Flag indicating whether client connection keepalive is supported. The value is inferred from the HTTP version and the value of the “Connection” header.

- `keepalive` — 标志，指示是否支持客户端连接的保持活动状态。该值是根据 HTTP 版本和“Connection”头的值推断出的。

- `header_sent` — Flag indicating that the output header has already been sent by the request.

- `header_sent` — 标志，指示输出头部是否已由请求发送。

- `internal` — Flag indicating that the current request is internal. To enter the internal state, a request must pass through an internal redirect or be a subrequest. Internal requests are allowed to enter internal locations.

- `internal` — 标志，指示当前请求是内部请求。要进入内部状态，请求必须通过内部重定向或成为子请求。允许内部请求进入内部位置。

- `allow_ranges` — Flag indicating that a partial response can be sent to the client, as requested by the HTTP Range header.

- `allow_ranges` — 标志，指示是否可以向客户端发送部分响应，正如 HTTP Range 头所请求的那样。

- `subrequest_ranges` — Flag indicating that a partial response can be sent while a subrequest is being processed.

- `subrequest_ranges` — 标志，指示在处理子请求时是否可以发送部分响应。

- `single_range` — Flag indicating that only a single continuous range of output data can be sent to the client. This flag is usually set when sending a stream of data, for example from a proxied server, and the entire response is not available in one buffer.

- `single_range` — 标志，指示只能向客户端发送连续的单一范围的输出数据。当发送数据流时通常设置此标志，例如从代理服务器发送数据流，并且整个响应不在一个缓冲区中可用。

- `main_filter_need_in_memory`, `filter_need_in_memory` — Flags requesting that the output produced in memory buffers rather than files. This is a signal to the copy filter to read data from file buffers even if sendfile is enabled. The difference between the two flags is the location of the filter modules that set them. Filters called before the postpone filter in the filter chain set `filter_need_in_memory`, requesting that only the current request output come in memory buffers. Filters called later in the filter chain set `main_filter_need_in_memory`, requesting that both the main request and all subrequests read files in memory while sending output.

- `main_filter_need_in_memory`、`filter_need_in_memory` — 标志，请求输出是否需要在内存缓冲区中而不是文件中。这是给复制过滤器的信号，即使启用了 sendfile，也要从文件缓冲区读取数据。这两个标志之间的区别是设置它们的过滤器模块的位置。在过滤器链中在延迟过滤器之前被调用的过滤器设置 `filter_need_in_memory`，请求仅请求当前请求输出进入内存缓冲区。在过滤器链中稍后调用的过滤器设置 `main_filter_need_in_memory`，要求主请求和所有子请求在发送输出时都在内存中读取文件。

- `filter_need_temporary` — Flag requesting that the request output be produced in temporary buffers, but not in readonly memory buffers or file buffers. This is used by filters which may change output directly in the buffers where it's sent.

- `filter_need_temporary` — 标志，请求输出是否需要在临时缓冲区中生成，而不是在只读内存缓冲区或文件缓冲区中生成。这由可能直接在发送缓冲区中更改输出的过滤器使用。



### 配置 Configuration

Each HTTP module can have three types of configuration:

​	每个 HTTP 模块可以具有三种类型的配置：

- Main configuration — Applies to the entire `http` block. Functions as global settings for a module.
- 主配置 — 适用于整个 `http` 块。作为模块的全局设置。
- Server configuration — Applies to a single `server` block. Functions as server-specific settings for a module.
- 服务器配置 — 适用于单个 `server` 块。作为模块的服务器特定设置。
- Location configuration — Applies to a single `location`, `if` or `limit_except` block. Functions as location-specific settings for a module.
- 位置配置 — 适用于单个 `location`、`if` 或 `limit_except` 块。作为模块的位置特定设置。

Configuration structures are created at the nginx configuration stage by calling functions, which allocate the structures, initialize them and merge them. The following example shows how to create a simple location configuration for a module. The configuration has one setting, `foo`, of type unsigned integer.

​	在 nginx 配置阶段，通过调用函数来创建配置结构，分配这些结构，初始化它们并合并它们。以下示例展示了如何为一个模块创建简单的位置配置。配置有一个名为 `foo` 的设置，类型为无符号整数。

```
typedef struct {
    ngx_uint_t  foo;
} ngx_http_foo_loc_conf_t;


static ngx_http_module_t  ngx_http_foo_module_ctx = {
    NULL,                                  /* preconfiguration */
    NULL,                                  /* postconfiguration */

    NULL,                                  /* create main configuration */
    NULL,                                  /* init main configuration */

    NULL,                                  /* create server configuration */
    NULL,                                  /* merge server configuration */

    ngx_http_foo_create_loc_conf,          /* create location configuration */
    ngx_http_foo_merge_loc_conf            /* merge location configuration */
};


static void *
ngx_http_foo_create_loc_conf(ngx_conf_t *cf)
{
    ngx_http_foo_loc_conf_t  *conf;

    conf = ngx_pcalloc(cf->pool, sizeof(ngx_http_foo_loc_conf_t));
    if (conf == NULL) {
        return NULL;
    }

    conf->foo = NGX_CONF_UNSET_UINT;

    return conf;
}


static char *
ngx_http_foo_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child)
{
    ngx_http_foo_loc_conf_t *prev = parent;
    ngx_http_foo_loc_conf_t *conf = child;

    ngx_conf_merge_uint_value(conf->foo, prev->foo, 1);
}
```

As seen in the example, the `ngx_http_foo_create_loc_conf()` function creates a new configuration structure, and `ngx_http_foo_merge_loc_conf()` merges a configuration with configuration from a higher level. In fact, server and location configuration do not exist only at the server and location levels, but are also created for all levels above them. Specifically, a server configuration is also created at the main level and location configurations are created at the main, server, and location levels. These configurations make it possible to specify server- and location-specific settings at any level of an nginx configuration file. Eventually configurations are merged down. A number of macros like `NGX_CONF_UNSET` and `NGX_CONF_UNSET_UINT` are provided for indicating a missing setting and ignoring it while merging. Standard nginx merge macros like `ngx_conf_merge_value()` and `ngx_conf_merge_uint_value()` provide a convenient way to merge a setting and set the default value if none of the configurations provided an explicit value. For complete list of macros for different types, see `src/core/ngx_conf_file.h`.

​	如示例所示，`ngx_http_foo_create_loc_conf()` 函数创建一个新的配置结构，`ngx_http_foo_merge_loc_conf()` 函数将一个配置与更高级别的配置合并。实际上，服务器和位置配置不仅仅存在于服务器和位置级别，还会为它们上面的所有级别创建。具体来说，服务器配置也会在主级别创建，位置配置会在主、服务器和位置级别创建。这些配置使得可以在 nginx 配置文件的任何级别指定特定于服务器和位置的设置。最终会合并这些配置。一些宏，如 `NGX_CONF_UNSET` 和 `NGX_CONF_UNSET_UINT`，用于指示缺少设置并在合并时忽略它。标准的 nginx 合并宏，如 `ngx_conf_merge_value()` 和 `ngx_conf_merge_uint_value()`，为合并设置并在没有显式值的情况下设置默认值提供了一种便捷的方法。有关不同类型的完整宏列表，请参见 `src/core/ngx_conf_file.h`。

The following macros are available. for accessing configuration for HTTP modules at configuration time. They all take `ngx_conf_t` reference as the first argument.

​	以下是可用的宏，用于在配置时间访问 HTTP 模块的配置。它们都以 `ngx_conf_t` 引用作为第一个参数。

- `ngx_http_conf_get_module_main_conf(cf, module)`
- `ngx_http_conf_get_module_srv_conf(cf, module)`
- `ngx_http_conf_get_module_loc_conf(cf, module)`

The following example gets a pointer to a location configuration of standard nginx core module [ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html) and replaces the location content handler kept in the `handler` field of the structure.

​	以下示例获取了标准 nginx 核心模块 [ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html) 的位置配置指针，并替换了结构的 `handler` 字段中保存的位置内容处理程序。

```
static ngx_int_t ngx_http_foo_handler(ngx_http_request_t *r);


static ngx_command_t  ngx_http_foo_commands[] = {

    { ngx_string("foo"),
      NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS,
      ngx_http_foo,
      0,
      0,
      NULL },

      ngx_null_command
};


static char *
ngx_http_foo(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ngx_http_core_loc_conf_t  *clcf;

    clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
    clcf->handler = ngx_http_bar_handler;

    return NGX_CONF_OK;
}
```

The following macros are available for accessing configuration for HTTP modules at runtime.

​	以下宏可用于在运行时访问 HTTP 模块的配置。

- `ngx_http_get_module_main_conf(r, module)`
- `ngx_http_get_module_srv_conf(r, module)`
- `ngx_http_get_module_loc_conf(r, module)`

These macros receive a reference to an HTTP request `ngx_http_request_t`. The main configuration of a request never changes. Server configuration can change from the default after the virtual server for the request is chosen. Location configuration selected for processing a request can change multiple times as a result of a rewrite operation or internal redirect. The following example shows how to access a module's HTTP configuration at runtime.

​	这些宏接收一个 HTTP 请求的引用 `ngx_http_request_t`。请求的主配置永远不会改变。服务器配置可以在选择请求的虚拟服务器之后从默认值更改。选择用于处理请求的位置配置可以因重写操作或内部重定向而多次更改。以下示例展示了如何在运行时访问模块的 HTTP 配置。

```
static ngx_int_t
ngx_http_foo_handler(ngx_http_request_t *r)
{
    ngx_http_foo_loc_conf_t  *flcf;

    flcf = ngx_http_get_module_loc_conf(r, ngx_http_foo_module);

    ...
}
```



### 阶段 Phases

Each HTTP request passes through a sequence of phases. In each phase a distinct type of processing is performed on the request. Module-specific handlers can be registered in most phases, and many standard nginx modules register their phase handlers as a way to get called at a specific stage of request processing. Phases are processed successively and the phase handlers are called once the request reaches the phase. Following is the list of nginx HTTP phases.

​	每个 HTTP 请求都通过一系列阶段。在每个阶段中，对请求执行不同类型的处理。模块特定的处理程序可以在大多数阶段中注册，许多标准 nginx 模块将其阶段处理程序注册为在请求处理的特定阶段调用的一种方式。阶段按顺序处理，并且一旦请求达到阶段，就会调用阶段处理程序。以下是 nginx HTTP 阶段的列表。

- `NGX_HTTP_POST_READ_PHASE` — First phase. The [ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html) registers its handler at this phase to enable substitution of client addresses before any other module is invoked.
- `NGX_HTTP_POST_READ_PHASE` — 第一个阶段。[ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html) 在此阶段注册其处理程序，以在调用任何其他模块之前启用客户端地址的替换。
- `NGX_HTTP_SERVER_REWRITE_PHASE` — Phase where rewrite directives defined in a `server` block (but outside a `location` block) are processed. The [ngx_http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) installs its handler at this phase.
- `NGX_HTTP_SERVER_REWRITE_PHASE` — 重写指令在此阶段处理的阶段。[ngx_http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 在此阶段安装其处理程序。
- `NGX_HTTP_FIND_CONFIG_PHASE` — Special phase where a location is chosen based on the request URI. Before this phase, the default location for the relevant virtual server is assigned to the request, and any module requesting a location configuration receives the configuration for the default server location. This phase assigns a new location to the request. No additional handlers can be registered at this phase.
- `NGX_HTTP_FIND_CONFIG_PHASE` — 特殊的阶段，根据请求 URI 选择位置。在此阶段之前，将请求分配给相关虚拟服务器的默认位置，并且任何请求位置配置的模块都会接收默认服务器位置的配置。此阶段将新位置分配给请求。在此阶段不能注册其他处理程序。
- `NGX_HTTP_REWRITE_PHASE` — Same as `NGX_HTTP_SERVER_REWRITE_PHASE`, but for rewrite rules defined in the location, chosen in the previous phase.
- `NGX_HTTP_REWRITE_PHASE` — 与 `NGX_HTTP_SERVER_REWRITE_PHASE` 相同，但用于前一阶段中选择的位置中定义的重写规则。
- `NGX_HTTP_POST_REWRITE_PHASE` — Special phase where the request is redirected to a new location if its URI changed during a rewrite. This is implemented by the request going through the `NGX_HTTP_FIND_CONFIG_PHASE` again. No additional handlers can be registered at this phase.
- `NGX_HTTP_POST_REWRITE_PHASE` — 如果请求在重写期间更改了 URI，则将请求重定向到新位置的特殊阶段。这是通过请求再次通过 `NGX_HTTP_FIND_CONFIG_PHASE` 来实现的。在此阶段不能注册其他处理程序。
- `NGX_HTTP_PREACCESS_PHASE` — A common phase for different types of handlers, not associated with access control. The standard nginx modules [ngx_http_limit_conn_module ](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)and [ngx_http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) register their handlers at this phase.
- `NGX_HTTP_PREACCESS_PHASE` — 不与访问控制相关的不同类型的处理程序的常见阶段。标准的 nginx 模块 [ngx_http_limit_conn_module](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) 和 [ngx_http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) 在此阶段注册其处理程序。
- `NGX_HTTP_ACCESS_PHASE` — Phase where it is verified that the client is authorized to make the request. Standard nginx modules such as [ngx_http_access_module](https://nginx.org/en/docs/http/ngx_http_access_module.html) and [ngx_http_auth_basic_module ](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)register their handlers at this phase. By default the client must pass the authorization check of all handlers registered at this phase for the request to continue to the next phase. The [satisfy](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) directive, can be used to permit processing to continue if any of the phase handlers authorizes the client.
- `NGX_HTTP_ACCESS_PHASE` — 验证客户端是否有权进行请求的阶段。标准 nginx 模块，如 [ngx_http_access_module](https://nginx.org/en/docs/http/ngx_http_access_module.html) 和 [ngx_http_auth_basic_module ](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)在此阶段注册其处理程序。默认情况下，客户端必须通过此阶段注册的所有处理程序的授权检查，以便继续到下一个阶段。可以使用 [satisfy](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) 指令，如果阶段处理程序中的任何一个授权客户端，允许继续处理。
- `NGX_HTTP_POST_ACCESS_PHASE` — Special phase where the [satisfy any](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) directive is processed. If some access phase handlers denied access and none explicitly allowed it, the request is finalized. No additional handlers can be registered at this phase.
- `NGX_HTTP_POST_ACCESS_PHASE` — 特殊阶段，处理 [satisfy any](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) 指令。如果某些访问阶段处理程序拒绝访问并且没有明确允许访问，请求将被终止。在此阶段不能注册其他处理程序。
- `NGX_HTTP_PRECONTENT_PHASE` — Phase for handlers to be called prior to generating content. Standard modules such as [ngx_http_try_files_module](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) and [ngx_http_mirror_module](https://nginx.org/en/docs/http/ngx_http_mirror_module.html) register their handlers at this phase.
- `NGX_HTTP_PRECONTENT_PHASE` — 处理生成内容之前调用的阶段。标准模块，如 [ngx_http_try_files_module](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) 和 [ngx_http_mirror_module](https://nginx.org/en/docs/http/ngx_http_mirror_module.html) 在此阶段注册其处理程序。
- `NGX_HTTP_CONTENT_PHASE` — Phase where the response is normally generated. Multiple nginx standard modules register their handlers at this phase, including [ngx_http_index_module](https://nginx.org/en/docs/http/ngx_http_index_module.html) or `ngx_http_static_module`. They are called sequentially until one of them produces the output. It's also possible to set content handlers on a per-location basis. If the [ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html)'s location configuration has `handler` set, it is called as the content handler and the handlers installed at this phase are ignored.
- `NGX_HTTP_CONTENT_PHASE` — 正常生成响应的阶段。许多标准 nginx 模块在此阶段注册其处理程序，包括 [ngx_http_index_module](https://nginx.org/en/docs/http/ngx_http_index_module.html) 或 `ngx_http_static_module`。它们按顺序调用，直到其中一个产生输出为止。还可以根据位置设置内容处理程序。如果 [ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html) 的位置配置具有 `handler` 设置，则该位置作为内容处理程序调用，而在此阶段安装的处理程序将被忽略。
- `NGX_HTTP_LOG_PHASE` — Phase where request logging is performed. Currently, only the [ngx_http_log_module](https://nginx.org/en/docs/http/ngx_http_log_module.html) registers its handler at this stage for access logging. Log phase handlers are called at the very end of request processing, right before freeing the request.
- `NGX_HTTP_LOG_PHASE` — 执行请求记录的阶段。目前，仅 [ngx_http_log_module](https://nginx.org/en/docs/http/ngx_http_log_module.html) 在此阶段注册其处理程序，用于访问日志记录。日志阶段处理程序在请求处理结束之前的最后调用。

Following is the example of a preaccess phase handler.

​	以下是预访问阶段处理程序的示例。

```
static ngx_http_module_t  ngx_http_foo_module_ctx = {
    NULL,                                  /* preconfiguration */
    ngx_http_foo_init,                     /* postconfiguration */

    NULL,                                  /* create main configuration */
    NULL,                                  /* init main configuration */

    NULL,                                  /* create server configuration */
    NULL,                                  /* merge server configuration */

    NULL,                                  /* create location configuration */
    NULL                                   /* merge location configuration */
};


static ngx_int_t
ngx_http_foo_handler(ngx_http_request_t *r)
{
    ngx_str_t  *ua;

    ua = r->headers_in->user_agent;

    if (ua == NULL) {
        return NGX_DECLINED;
    }

    /* reject requests with "User-Agent: foo" */
    /* 拒绝具有 "User-Agent: foo" 的请求 */
    if (ua->value.len == 3 && ngx_strncmp(ua->value.data, "foo", 3) == 0) {
        return NGX_HTTP_FORBIDDEN;
    }

    return NGX_DECLINED;
}


static ngx_int_t
ngx_http_foo_init(ngx_conf_t *cf)
{
    ngx_http_handler_pt        *h;
    ngx_http_core_main_conf_t  *cmcf;

    cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);

    h = ngx_array_push(&cmcf->phases[NGX_HTTP_PREACCESS_PHASE].handlers);
    if (h == NULL) {
        return NGX_ERROR;
    }

    *h = ngx_http_foo_handler;

    return NGX_OK;
}
```

Phase handlers are expected to return specific codes:

​	阶段处理程序预期返回特定的代码：

- `NGX_OK` — Proceed to the next phase.
- `NGX_OK` — 继续到下一个阶段。
- `NGX_DECLINED` — Proceed to the next handler of the current phase. If the current handler is the last in the current phase, move to the next phase.
- `NGX_DECLINED` — 继续到当前阶段的下一个处理程序。如果当前处理程序是当前阶段中的最后一个处理程序，则移至下一个阶段。
- `NGX_AGAIN`, `NGX_DONE` — Suspend phase handling until some future event which can be an asynchronous I/O operation or just a delay, for example. It is assumed, that phase handling will be resumed later by calling `ngx_http_core_run_phases()`.
- `NGX_AGAIN`，`NGX_DONE` — 暂停阶段处理，直到未来事件（可以是异步 I/O 操作或延迟等）发生。假定将通过调用 `ngx_http_core_run_phases()` 来稍后恢复阶段处理。
- Any other value returned by the phase handler is treated as a request finalization code, in particular, an HTTP response code. The request is finalized with the code provided.
- 阶段处理程序返回的任何其他值都被视为请求完成代码，特别是 HTTP 响应代码。使用提供的代码完成请求。

For some phases, return codes are treated in a slightly different way. At the content phase, any return code other that `NGX_DECLINED` is considered a finalization code. Any return code from the location content handlers is considered a finalization code. At the access phase, in [satisfy any](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) mode, any return code other than `NGX_OK`, `NGX_DECLINED`, `NGX_AGAIN`, `NGX_DONE` is considered a denial. If no subsequent access handlers allow or deny access with a different code, the denial code will become the finalization code.

​	对于某些阶段，返回代码的处理方式略有不同。在内容阶段，除了 `NGX_DECLINED` 之外的任何返回代码都被视为完成代码。从位置内容处理程序返回的任何返回代码都被视为完成代码。在访问阶段中，在 [satisfy any](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) 模式下，除了 `NGX_OK`、`NGX_DECLINED`、`NGX_AGAIN`、`NGX_DONE` 之外的任何返回代码都被视为拒绝。如果后续的访问处理程序没有以不同的代码允许或拒绝访问，则拒绝代码将成为完成代码。



### 变量 Variables



### 访问现有变量 Accessing existing variables

Variables can be referenced by index (this is the most common method) or name (see [below](https://nginx.org/en/docs/dev/development_guide.html#http_creating_variables)). The index is created at configuration stage, when a variable is added to the configuration. To obtain the variable index, use `ngx_http_get_variable_index()`:

​	变量可以通过索引（这是最常见的方法）或名称来引用（参见 [下面](https://nginx.org/en/docs/dev/development_guide.html#http_creating_variables)）。索引是在配置阶段创建的，当变量添加到配置中时。要获取变量索引，使用 `ngx_http_get_variable_index()`：

```
ngx_str_t  name;  /* ngx_string("foo") */
ngx_int_t  index;

index = ngx_http_get_variable_index(cf, &name);
```

Here, `cf` is a pointer to nginx configuration and `name` points to a string containing the variable name. The function returns `NGX_ERROR` on error or a valid index otherwise, which is typically stored somewhere in the module's configuration for future use.

​	这里，`cf` 是指向 nginx 配置的指针，`name` 指向包含变量名称的字符串。如果出现错误，该函数返回 `NGX_ERROR`，否则返回一个有效的索引，通常将其存储在模块的配置中以供将来使用。

All HTTP variables are evaluated in the context of a given HTTP request, and results are specific to and cached in that HTTP request. All functions that evaluate variables return the `ngx_http_variable_value_t` type, representing the variable value:

​	所有 HTTP 变量在给定 HTTP 请求的上下文中进行评估，并且结果特定于并且在该 HTTP 请求中进行缓存。所有评估变量的函数都返回 `ngx_http_variable_value_t` 类型，表示变量值：

```
typedef ngx_variable_value_t  ngx_http_variable_value_t;

typedef struct {
    unsigned    len:28;

    unsigned    valid:1;
    unsigned    no_cacheable:1;
    unsigned    not_found:1;
    unsigned    escape:1;

    u_char     *data;
} ngx_variable_value_t;
```

where:

其中：

- `len` — The length of the value
- `data` — The value itself
- `valid` — The value is valid
- `not_found` — The variable was not found and thus the `data` and `len` fields are irrelevant; this can happen, for example, with variables like `$arg_foo` when a corresponding argument was not passed in a request
- `no_cacheable` — Do not cache result
- `escape` — Used internally by the logging module to mark values that require escaping on output.
- `len` — 值的长度
- `data` — 值本身
- `valid` — 值是有效的
- `not_found` — 未找到变量，因此 `data` 和 `len` 字段无关；例如，当没有传递相应的参数时，类似于 `$arg_foo` 的变量会发生这种情况
- `no_cacheable` — 不要缓存结果
- `escape` — 日志模块内部使用，用于标记需要在输出时进行转义的值。



The `ngx_http_get_flushed_variable()` and `ngx_http_get_indexed_variable()` functions are used to obtain the value of a variable. They have the same interface - accepting an HTTP request `r` as a context for evaluating the variable and an `index` that identifies it. An example of typical usage:

​	`ngx_http_get_flushed_variable()` 和 `ngx_http_get_indexed_variable()` 函数用于获取变量的值。它们具有相同的接口，接受 HTTP 请求 `r` 作为评估变量的上下文和标识它的 `index`。典型用法示例：

```
ngx_http_variable_value_t  *v;

v = ngx_http_get_flushed_variable(r, index);

if (v == NULL || v->not_found) {
    /* we failed to get value or there is no such variable, handle it */
    /* 未能获取值或没有此类变量，进行处理 */
    return NGX_ERROR;
}

/* some meaningful value is found */
/* 找到了某个有意义的值 */
```

The difference between functions is that the `ngx_http_get_indexed_variable()` returns a cached value and `ngx_http_get_flushed_variable()` flushes the cache for non-cacheable variables.

​	函数之间的区别在于，`ngx_http_get_indexed_variable()` 返回一个缓存值，而 `ngx_http_get_flushed_variable()` 刷新不可缓存变量的缓存。

Some modules, such as SSI and Perl, need to deal with variables for which the name is not known at configuration time. An index therefore cannot be used to access them, but the `ngx_http_get_variable(r, name, key)` function is available. It searches for a variable with a given `name` and its hash `key` derived from the name.

​	某些模块（例如 SSI 和 Perl）需要处理在配置时名称未知的变量。因此，索引不能用于访问它们，但是 `ngx_http_get_variable(r, name, key)` 函数可用。它会查找具有给定名称和从名称派生的哈希 `key` 的变量。



### 创建变量 Creating variables

To create a variable, use the `ngx_http_add_variable()` function. It takes as arguments a configuration (where the variable is registered), the variable name and flags that control the function's behaviour:

​	要创建一个变量，可以使用 `ngx_http_add_variable()` 函数。该函数的参数包括配置（变量注册的位置）、变量名称以及控制函数行为的标志：

- `NGX_HTTP_VAR_CHANGEABLE` — Enables redefinition of the variable: there is no conflict if another module defines a variable with the same name. This allows the [set](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set) directive to override variables.
- `NGX_HTTP_VAR_CHANGEABLE` — 允许重新定义变量：如果另一个模块使用相同的名称定义了一个变量，不会发生冲突。这允许 [set](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set) 指令覆盖变量。
- `NGX_HTTP_VAR_NOCACHEABLE` — Disables caching, which is useful for variables such as `$time_local`.
- `NGX_HTTP_VAR_NOCACHEABLE` — 禁用缓存，适用于诸如 `$time_local` 等变量。
- `NGX_HTTP_VAR_NOHASH` — Indicates that this variable is only accessible by index, not by name. This is a small optimization for use when it is known that the variable is not needed in modules like SSI or Perl.
- `NGX_HTTP_VAR_NOHASH` — 表示此变量只能通过索引访问，不能通过名称访问。这是一个小的优化，用于已知变量不会在类似 SSI 或 Perl 的模块中使用时。
- `NGX_HTTP_VAR_PREFIX` — The name of the variable is a prefix. In this case, a handler must implement additional logic to obtain the value of a specific variable. For example, all “`arg_`” variables are processed by the same handler, which performs lookup in request arguments and returns the value of a specific argument.
- `NGX_HTTP_VAR_PREFIX` — 变量名称是一个前缀。在这种情况下，处理程序必须实现附加逻辑，以获取特定变量的值。例如，所有以 “`arg_`” 开头的变量由同一个处理程序处理，该处理程序在请求参数中查找并返回特定参数的值。

The function returns NULL in case of error or a pointer to `ngx_http_variable_t` otherwise:

​	函数返回 NULL 表示错误，否则返回指向 `ngx_http_variable_t` 的指针：

```
struct ngx_http_variable_s {
    ngx_str_t                     name;
    ngx_http_set_variable_pt      set_handler;
    ngx_http_get_variable_pt      get_handler;
    uintptr_t                     data;
    ngx_uint_t                    flags;
    ngx_uint_t                    index;
};
```

The `get` and `set` handlers are called to obtain or set the variable value, `data` is passed to variable handlers, and `index` holds assigned variable index used to reference the variable.

​	`get` 和 `set` 处理程序用于获取或设置变量的值，`data` 传递给变量处理程序，`index` 保存分配的变量索引，用于引用该变量。

Usually, a null-terminated static array of `ngx_http_variable_t` structures is created by a module and processed at the preconfiguration stage to add variables into the configuration, for example:

​	通常，模块会创建一个以 null 结尾的静态 `ngx_http_variable_t` 结构数组，并在预配置阶段处理它，将变量添加到配置中，例如：

```
static ngx_http_variable_t  ngx_http_foo_vars[] = {

    { ngx_string("foo_v1"), NULL, ngx_http_foo_v1_variable, 0, 0, 0 },

      ngx_http_null_variable
};

static ngx_int_t
ngx_http_foo_add_variables(ngx_conf_t *cf)
{
    ngx_http_variable_t  *var, *v;

    for (v = ngx_http_foo_vars; v->name.len; v++) {
        var = ngx_http_add_variable(cf, &v->name, v->flags);
        if (var == NULL) {
            return NGX_ERROR;
        }

        var->get_handler = v->get_handler;
        var->data = v->data;
    }

    return NGX_OK;
}
```

This function in the example is used to initialize the `preconfiguration` field of the HTTP module context and is called before the parsing of HTTP configuration, so that the parser can refer to these variables.

​	这个示例中的函数用于初始化 HTTP 模块上下文的 `preconfiguration` 字段，在解析 HTTP 配置之前调用，以便解析器可以引用这些变量。

The `get` handler is responsible for evaluating a variable in the context of a specific request, for example:

​	`get` 处理程序负责在特定请求的上下文中评估变量，例如：

```
static ngx_int_t
ngx_http_variable_connection(ngx_http_request_t *r,
    ngx_http_variable_value_t *v, uintptr_t data)
{
    u_char  *p;

    p = ngx_pnalloc(r->pool, NGX_ATOMIC_T_LEN);
    if (p == NULL) {
        return NGX_ERROR;
    }

    v->len = ngx_sprintf(p, "%uA", r->connection->number) - p;
    v->valid = 1;
    v->no_cacheable = 0;
    v->not_found = 0;
    v->data = p;

    return NGX_OK;
}
```

It returns `NGX_ERROR` in case of internal error (for example, failed memory allocation) or `NGX_OK` otherwise. To learn the status of variable evaluation, inspect the flags in `ngx_http_variable_value_t` (see the description [above](https://nginx.org/en/docs/dev/development_guide.html#http_existing_variables)).

​	如果发生内部错误（例如，内存分配失败），则返回 `NGX_ERROR`；否则返回 `NGX_OK`。要了解变量评估的状态，请检查 `ngx_http_variable_value_t` 中的标志（参见[上面的](https://nginx.org/en/docs/dev/development_guide.html#http_existing_variables)描述）。

The `set` handler allows setting the property referenced by the variable. For example, the set handler of the `$limit_rate` variable modifies the request's `limit_rate` field:

​	`set` 处理程序允许设置变量引用的属性。例如，`$limit_rate` 变量的 set 处理程序修改请求的 `limit_rate` 字段：

```
...
{ ngx_string("limit_rate"), ngx_http_variable_request_set_size,
  ngx_http_variable_request_get_size,
  offsetof(ngx_http_request_t, limit_rate),
  NGX_HTTP_VAR_CHANGEABLE|NGX_HTTP_VAR_NOCACHEABLE, 0 },
...

static void
ngx_http_variable_request_set_size(ngx_http_request_t *r,
    ngx_http_variable_value_t *v, uintptr_t data)
{
    ssize_t    s, *sp;
    ngx_str_t  val;

    val.len = v->len;
    val.data = v->data;

    s = ngx_parse_size(&val);

    if (s == NGX_ERROR) {
        ngx_log_error(NGX_LOG_ERR, r->connection->log, 0,
                      "invalid size \"%V\"", &val);
        return;
    }

    sp = (ssize_t *) ((char *) r + data);

    *sp = s;

    return;
}
```





### 复杂值 Complex values

A complex value, despite its name, provides an easy way to evaluate expressions which can contain text, variables, and their combination.

​	复杂值，尽管其名称如此，提供了一种简单的方法来评估包含文本、变量及其组合的表达式。

The complex value description in `ngx_http_compile_complex_value` is compiled at the configuration stage into `ngx_http_complex_value_t` which is used at runtime to obtain results of expression evaluation.

​	`ngx_http_compile_complex_value` 中的复杂值描述在配置阶段编译为 `ngx_http_complex_value_t`，在运行时用于获取表达式评估的结果。

```
ngx_str_t                         *value;
ngx_http_complex_value_t           cv;
ngx_http_compile_complex_value_t   ccv;

value = cf->args->elts; /* directive arguments */ /* 指令参数 */

ngx_memzero(&ccv, sizeof(ngx_http_compile_complex_value_t));

ccv.cf = cf;
ccv.value = &value[1];
ccv.complex_value = &cv;
ccv.zero = 1;
ccv.conf_prefix = 1;

if (ngx_http_compile_complex_value(&ccv) != NGX_OK) {
    return NGX_CONF_ERROR;
}
```

Here, `ccv` holds all parameters that are required to initialize the complex value `cv`:

​	在这里，`ccv` 包含初始化复杂值 `cv` 所需的所有参数：

- `cf` — Configuration pointer
- `value` — String to be parsed (input)
- `complex_value` — Compiled value (output)
- `zero` — Flag that enables zero-terminating value
- `conf_prefix` — Prefixes the result with the configuration prefix (the directory where nginx is currently looking for configuration)
- `root_prefix` — Prefixes the result with the root prefix (the normal nginx installation prefix)
- `cf` — 配置指针
- `value` — 要解析的字符串（输入）
- `complex_value` — 编译后的值（输出）
- `zero` — 启用零终止值的标志
- `conf_prefix` — 将结果前缀为配置前缀（nginx 当前正在查找配置的目录）
- `root_prefix` — 将结果前缀为根前缀（正常的 nginx 安装前缀）

The `zero` flag is useful when results are to be passed to libraries that require zero-terminated strings, and prefixes are handy when dealing with filenames.

​	`zero` 标志在结果需要传递给需要零终止字符串的库时很有用，而前缀在处理文件名时很方便。

Upon successful compilation, `cv.lengths` contains information about the presence of variables in the expression. The NULL value means that the expression contained static text only, and so can be stored in a simple string rather than as a complex value.

​	成功编译后，`cv.lengths` 包含关于表达式中变量是否存在的信息。NULL 值表示表达式只包含静态文本，因此可以将其存储为简单字符串，而不是复杂值。

The `ngx_http_set_complex_value_slot()` is a convenient function used to initialize a complex value completely in the directive declaration itself.

​	`ngx_http_set_complex_value_slot()` 是一个方便的函数，用于在指令声明中完全初始化复杂值。

At runtime, a complex value can be calculated using the `ngx_http_complex_value()` function:

​	在运行时，可以使用 `ngx_http_complex_value()` 函数计算复杂值：

```
ngx_str_t  res;

if (ngx_http_complex_value(r, &cv, &res) != NGX_OK) {
    return NGX_ERROR;
}
```

Given the request `r` and previously compiled value `cv`, the function evaluates the expression and writes the result to `res`.

​	给定请求 `r` 和先前编译的值 `cv`，该函数评估表达式并将结果写入 `res`。



### 请求重定向 Request redirection

An HTTP request is always connected to a location via the `loc_conf` field of the `ngx_http_request_t` structure. This means that at any point the location configuration of any module can be retrieved from the request by calling `ngx_http_get_module_loc_conf(r, module)`. Request location can change several times during the request's lifetime. Initially, a default server location of the default server is assigned to a request. If the request switches to a different server (chosen by the HTTP “Host” header or SSL SNI extension), the request switches to the default location of that server as well. The next change of the location takes place at the `NGX_HTTP_FIND_CONFIG_PHASE` request phase. At this phase a location is chosen by request URI among all non-named locations configured for the server. The [ngx_http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) can change the request URI at the `NGX_HTTP_REWRITE_PHASE` request phase as a result of the [rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) directive and send the request back to the `NGX_HTTP_FIND_CONFIG_PHASE` phase for selection of a new location based on the new URI.

​	HTTP 请求始终通过 `ngx_http_request_t` 结构的 `loc_conf` 字段连接到位置。这意味着在任何时候都可以通过调用 `ngx_http_get_module_loc_conf(r, module)` 从请求中检索任何模块的位置配置。请求位置在请求的生命周期内可以多次更改。最初，请求分配给默认服务器位置的请求。如果请求切换到另一个服务器（由 HTTP 的 “Host” 头或 SSL SNI 扩展选择），请求也会切换到该服务器的默认位置。位置的下一个更改发生在 `NGX_HTTP_FIND_CONFIG_PHASE` 请求阶段。在此阶段，位置是根据请求 URI 在所有为服务器配置的非命名位置中选择的。[ngx_http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 可以在 `NGX_HTTP_REWRITE_PHASE` 请求阶段通过 [rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) 指令更改请求 URI，并将请求发送回 `NGX_HTTP_FIND_CONFIG_PHASE` 阶段，以基于新 URI 选择新位置。

It is also possible to redirect a request to a new location at any point by calling one of `ngx_http_internal_redirect(r, uri, args)` or `ngx_http_named_location(r, name)`.

​	还可以在任何时候通过调用 `ngx_http_internal_redirect(r, uri, args)` 或 `ngx_http_named_location(r, name)` 将请求重定向到新位置。

The `ngx_http_internal_redirect(r, uri, args)` function changes the request URI and returns the request to the `NGX_HTTP_SERVER_REWRITE_PHASE` phase. The request proceeds with a server default location. Later at `NGX_HTTP_FIND_CONFIG_PHASE` a new location is chosen based on the new request URI.

​	`ngx_http_internal_redirect(r, uri, args)` 函数更改请求 URI 并将请求返回到 `NGX_HTTP_SERVER_REWRITE_PHASE` 阶段。请求将继续使用服务器默认位置。稍后在 `NGX_HTTP_FIND_CONFIG_PHASE` 阶段，将根据新的请求 URI 选择新的位置。

The following example performs an internal redirect with the new request arguments.

​	以下示例执行了一个带有新请求参数的内部重定向。

```
ngx_int_t
ngx_http_foo_redirect(ngx_http_request_t *r)
{
    ngx_str_t  uri, args;

    ngx_str_set(&uri, "/foo");
    ngx_str_set(&args, "bar=1");

    return ngx_http_internal_redirect(r, &uri, &args);
}
```

The function `ngx_http_named_location(r, name)` redirects a request to a named location. The name of the location is passed as the argument. The location is looked up among all named locations of the current server, after which the requests switches to the `NGX_HTTP_REWRITE_PHASE` phase.

​	函数 `ngx_http_named_location(r, name)` 将请求重定向到命名位置。位置的名称作为参数传递。在当前服务器的所有命名位置中查找位置，然后请求切换到 `NGX_HTTP_REWRITE_PHASE` 阶段。

The following example performs a redirect to a named location @foo.

​	以下示例执行了一个重定向到命名位置 @foo。

```
ngx_int_t
ngx_http_foo_named_redirect(ngx_http_request_t *r)
{
    ngx_str_t  name;

    ngx_str_set(&name, "foo");

    return ngx_http_named_location(r, &name);
}
```

Both functions - `ngx_http_internal_redirect(r, uri, args)` and `ngx_http_named_location(r, name)` can be called when nginx modules have already stored some contexts in a request's `ctx` field. It's possible for these contexts to become inconsistent with the new location configuration. To prevent inconsistency, all request contexts are erased by both redirect functions.

​	这两个函数 - `ngx_http_internal_redirect(r, uri, args)` 和 `ngx_http_named_location(r, name)` 可以在 nginx 模块已经将某些上下文存储在请求的 `ctx` 字段时调用。这些上下文可能与新的位置配置不一致。为了防止不一致，这两个重定向函数会清除所有请求上下文。

Calling `ngx_http_internal_redirect(r, uri, args)` or `ngx_http_named_location(r, name)` increases the request `count`. For consistent request reference counting, call `ngx_http_finalize_request(r, NGX_DONE)` after redirecting the request. This will finalize current request code path and decrease the counter.

​	在调用 `ngx_http_internal_redirect(r, uri, args)` 或 `ngx_http_named_location(r, name)` 后，会增加请求的 `count`。为了保持一致的请求引用计数，在重定向请求后调用 `ngx_http_finalize_request(r, NGX_DONE)`。这将完成当前请求代码路径并减少计数器。

Redirected and rewritten requests become internal and can access the [internal](https://nginx.org/en/docs/http/ngx_http_core_module.html#internal) locations. Internal requests have the `internal` flag set.

​	重定向和重写后的请求变为内部请求，并且可以访问 [internal](https://nginx.org/en/docs/http/ngx_http_core_module.html#internal) 位置。内部请求设置了 `internal` 标志。



### 子请求 Subrequests

Subrequests are primarily used to insert output of one request into another, possibly mixed with other data. A subrequest looks like a normal request, but shares some data with its parent. In particular, all fields related to client input are shared because a subrequest does not receive any other input from the client. The request field `parent` for a subrequest contains a link to its parent request and is NULL for the main request. The field `main` contains a link to the main request in a group of requests.

​	子请求主要用于将一个请求的输出插入到另一个请求中，可能与其他数据混合。子请求看起来像是正常的请求，但与其父请求共享一些数据。特别是与客户端输入相关的所有字段都是共享的，因为子请求不会从客户端接收任何其他输入。子请求的 `parent` 字段包含到其父请求的链接，在主请求中该字段为 NULL。字段 `main` 包含一个链接到请求组中的主请求。

A subrequest starts in the `NGX_HTTP_SERVER_REWRITE_PHASE` phase. It passes through the same subsequent phases as a normal request and is assigned a location based on its own URI.

​	子请求从 `NGX_HTTP_SERVER_REWRITE_PHASE` 阶段开始。它通过与正常请求相同的后续阶段，并根据自己的 URI 被分配到一个位置。

The output header in a subrequest is always ignored. The `ngx_http_postpone_filter` places the subrequest's output body in the right position relative to other data produced by the parent request.

​	子请求的输出头总是被忽略。`ngx_http_postpone_filter` 将子请求的输出正文放置在与父请求生成的其他数据相对应的正确位置。

Subrequests are related to the concept of active requests. A request `r` is considered active if `c->data == r`, where `c` is the client connection object. At any given point, only the active request in a request group is allowed to output its buffers to the client. An inactive request can still send its output to the filter chain, but it does not pass beyond the `ngx_http_postpone_filter` and remains buffered by that filter until the request becomes active. Here are some rules of request activation:

​	子请求与活动请求的概念相关联。如果请求 `r` 满足条件 `c->data == r`，其中 `c` 是客户端连接对象，那么请求 `r` 被认为是活动的。在任何给定的时刻，请求组中只允许活动请求将其缓冲区输出到客户端。非活动请求仍然可以将其输出发送到过滤器链，但不会超越 `ngx_http_postpone_filter`，并会被该过滤器缓冲，直到请求变为活动状态。以下是请求激活的一些规则：

 

- Initially, the main request is active.
- 最初，主请求是活动的。
- The first subrequest of an active request becomes active right after creation.
- 活动请求的第一个子请求在创建后立即激活。
- The `ngx_http_postpone_filter` activates the next request in the active request's subrequest list, once all data prior to that request are sent.
- `ngx_http_postpone_filter` 一旦发送了该请求之前的所有数据，就会激活活动请求的子请求列表中的下一个请求。
- When a request is finalized, its parent is activated.
- 当请求完成时，其父请求被激活。

Create a subrequest by calling the function `ngx_http_subrequest(r, uri, args, psr, ps, flags)`, where `r` is the parent request, `uri` and `args` are the URI and arguments of the subrequest, `psr` is the output parameter, which receives the newly created subrequest reference, `ps` is a callback object for notifying the parent request that the subrequest is being finalized, and `flags` is bitmask of flags. The following flags are available:

​	通过调用函数 `ngx_http_subrequest(r, uri, args, psr, ps, flags)` 来创建一个子请求，其中 `r` 是父请求，`uri` 和 `args` 是子请求的 URI 和参数，`psr` 是输出参数，用于接收新创建的子请求引用，`ps` 是用于通知父请求子请求正在完成的回调对象，而 `flags` 是标志位掩码。以下标志可用：

- `NGX_HTTP_SUBREQUEST_IN_MEMORY` - Output is not sent to the client, but rather stored in memory. The flag only affects subrequests which are processed by one of the proxying modules. After a subrequest is finalized its output is available in `r->out` of type `ngx_buf_t`.
- `NGX_HTTP_SUBREQUEST_IN_MEMORY` - 输出不会发送到客户端，而是存储在内存中。该标志仅影响由代理模块之一处理的子请求。在子请求完成后，其输出在类型为 `ngx_buf_t` 的 `r->out` 中可用。
- `NGX_HTTP_SUBREQUEST_WAITED` - The subrequest's `done` flag is set even if the subrequest is not active when it is finalized. This subrequest flag is used by the SSI filter.
- `NGX_HTTP_SUBREQUEST_WAITED` - 即使子请求在完成时不是活动的，也会设置子请求的 `done` 标志。这个子请求标志被 SSI 过滤器使用。
- `NGX_HTTP_SUBREQUEST_CLONE` - The subrequest is created as a clone of its parent. It is started at the same location and proceeds from the same phase as the parent request.
- `NGX_HTTP_SUBREQUEST_CLONE` - 子请求被创建为其父请求的克隆。它从与父请求相同的位置开始，并从相同的阶段继续。

The following example creates a subrequest with the URI of `/foo`.

​	以下示例创建一个具有 URI `/foo` 的子请求。

```
ngx_int_t            rc;
ngx_str_t            uri;
ngx_http_request_t  *sr;

...

ngx_str_set(&uri, "/foo");

rc = ngx_http_subrequest(r, &uri, NULL, &sr, NULL, 0);
if (rc == NGX_ERROR) {
    /* error */
}
```

This example clones the current request and sets a finalization callback for the subrequest.

​	此示例克隆当前请求并为子请求设置了最终化回调。

```
ngx_int_t
ngx_http_foo_clone(ngx_http_request_t *r)
{
    ngx_http_request_t          *sr;
    ngx_http_post_subrequest_t  *ps;

    ps = ngx_palloc(r->pool, sizeof(ngx_http_post_subrequest_t));
    if (ps == NULL) {
        return NGX_ERROR;
    }

    ps->handler = ngx_http_foo_subrequest_done;
    ps->data = "foo";

    return ngx_http_subrequest(r, &r->uri, &r->args, &sr, ps,
                               NGX_HTTP_SUBREQUEST_CLONE);
}


ngx_int_t
ngx_http_foo_subrequest_done(ngx_http_request_t *r, void *data, ngx_int_t rc)
{
    char  *msg = (char *) data;

    ngx_log_error(NGX_LOG_INFO, r->connection->log, 0,
                  "done subrequest r:%p msg:%s rc:%i", r, msg, rc);

    return rc;
}
```

Subrequests are normally created in a body filter, in which case their output can be treated like the output from any explicit request. This means that eventually the output of a subrequest is sent to the client, after all explicit buffers that are passed before subrequest creation and before any buffers that are passed after creation. This ordering is preserved even for large hierarchies of subrequests. The following example inserts output from a subrequest after all request data buffers, but before the final buffer with the `last_buf` flag.

​	子请求通常在 body 过滤器中创建，在这种情况下，它们的输出可以像来自任何显式请求的输出一样进行处理。这意味着最终子请求的输出会发送到客户端，经过在子请求创建之前传递的所有显式缓冲区，并在创建之后传递的任何缓冲区之前。即使在子请求的大型层次结构中，此顺序也会保留。以下示例在所有请求数据缓冲区之后，但在具有 `last_buf` 标志的最终缓冲区之前，插入了子请求的输出。

```
ngx_int_t
ngx_http_foo_body_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    ngx_int_t                   rc;
    ngx_buf_t                  *b;
    ngx_uint_t                  last;
    ngx_chain_t                *cl, out;
    ngx_http_request_t         *sr;
    ngx_http_foo_filter_ctx_t  *ctx;

    ctx = ngx_http_get_module_ctx(r, ngx_http_foo_filter_module);
    if (ctx == NULL) {
        return ngx_http_next_body_filter(r, in);
    }

    last = 0;

    for (cl = in; cl; cl = cl->next) {
        if (cl->buf->last_buf) {
            cl->buf->last_buf = 0;
            cl->buf->last_in_chain = 1;
            cl->buf->sync = 1;
            last = 1;
        }
    }

    /* Output explicit output buffers */
    /* 输出显式输出缓冲区 */

    rc = ngx_http_next_body_filter(r, in);

    if (rc == NGX_ERROR || !last) {
        return rc;
    }

    /*
     * Create the subrequest.  The output of the subrequest
     * will automatically be sent after all preceding buffers,
     * but before the last_buf buffer passed later in this function.
     */
     /*
     * 创建子请求。子请求的输出将自动在所有先前的缓冲区之后发送，
     * 但在稍后在该函数中传递的 last_buf 缓冲区之前。
     */

    if (ngx_http_subrequest(r, ctx->uri, NULL, &sr, NULL, 0) != NGX_OK) {
        return NGX_ERROR;
    }

    ngx_http_set_ctx(r, NULL, ngx_http_foo_filter_module);

    /* Output the final buffer with the last_buf flag */
    /* 输出带有 last_buf 标志的最终缓冲区 */

    b = ngx_calloc_buf(r->pool);
    if (b == NULL) {
        return NGX_ERROR;
    }

    b->last_buf = 1;

    out.buf = b;
    out.next = NULL;

    return ngx_http_output_filter(r, &out);
}
```

A subrequest can also be created for other purposes than data output. For example, the [ngx_http_auth_request_module](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) module creates a subrequest at the `NGX_HTTP_ACCESS_PHASE` phase. To disable output at this point, the `header_only` flag is set on the subrequest. This prevents the subrequest body from being sent to the client. Note that the subrequest's header is never sent to the client. The result of the subrequest can be analyzed in the callback handler.

​	子请求也可以出于除了数据输出之外的其他目的而创建。例如，[ngx_http_auth_request_module](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) 模块在 `NGX_HTTP_ACCESS_PHASE` 阶段创建子请求。在此阶段禁用输出时，会在子请求上设置 `header_only` 标志。这会阻止子请求正文被发送到客户端。请注意，子请求的头部永远不会被发送到客户端。可以在回调处理程序中分析子请求的结果。

### 请求最终化 Request finalization

An HTTP request is finalized by calling the function `ngx_http_finalize_request(r, rc)`. It is usually finalized by the content handler after all output buffers are sent to the filter chain. At this point all of the output might not be sent to the client, with some of it remaining buffered somewhere along the filter chain. If it is, the `ngx_http_finalize_request(r, rc)` function automatically installs a special handler `ngx_http_writer(r)` to finish sending the output. A request is also finalized in case of an error or if a standard HTTP response code needs to be returned to the client.

​	通过调用函数 `ngx_http_finalize_request(r, rc)` 来完成 HTTP 请求。通常，在所有输出缓冲区被发送到过滤器链后，由内容处理程序来完成请求。此时，并不一定所有的输出都已发送到客户端，其中一些可能在过滤器链的某个地方保留了缓冲。如果有的话，`ngx_http_finalize_request(r, rc)` 函数会自动安装一个特殊的处理程序 `ngx_http_writer(r)` 来完成输出的发送。在发生错误或需要向客户端返回标准的 HTTP 响应代码时，请求也会被最终化。

The function `ngx_http_finalize_request(r, rc)` expects the following `rc` values:

​	函数 `ngx_http_finalize_request(r, rc)` 期望以下 `rc` 值：

- `NGX_DONE` - Fast finalization. Decrement the request `count` and destroy the request if it reaches zero. The client connection can be used for more requests after the current request is destroyed.
- `NGX_DONE` - 快速最终化。递减请求 `count` 并在计数达到零时销毁请求。当前请求被销毁后，客户端连接可以继续用于其他请求。
- `NGX_ERROR`, `NGX_HTTP_REQUEST_TIME_OUT` (`408`), `NGX_HTTP_CLIENT_CLOSED_REQUEST` (`499`) - Error finalization. Terminate the request as soon as possible and close the client connection.
- `NGX_ERROR`、`NGX_HTTP_REQUEST_TIME_OUT` (`408`)、`NGX_HTTP_CLIENT_CLOSED_REQUEST` (`499`) - 错误最终化。尽快终止请求并关闭客户端连接。
- `NGX_HTTP_CREATED` (`201`), `NGX_HTTP_NO_CONTENT` (`204`), codes greater than or equal to `NGX_HTTP_SPECIAL_RESPONSE` (`300`) - Special response finalization. For these values nginx either sends to the client a default response page for the code or performs the internal redirect to an [error_page](https://nginx.org/en/docs/http/ngx_http_core_module.html#error_page) location if that is configured for the code.
- `NGX_HTTP_CREATED` (`201`)、`NGX_HTTP_NO_CONTENT` (`204`)、大于或等于 `NGX_HTTP_SPECIAL_RESPONSE` (`300`) 的代码 - 特殊响应最终化。对于这些值，nginx 会向客户端发送用于该代码的默认响应页面，或者如果为该代码配置了 [error_page](https://nginx.org/en/docs/http/ngx_http_core_module.html#error_page) 位置，则执行内部重定向到该位置。
- Other codes are considered successful finalization codes and might activate the request writer to finish sending the response body. Once the body is completely sent, the request `count` is decremented. If it reaches zero, the request is destroyed, but the client connection can still be used for other requests. If `count` is positive, there are unfinished activities within the request, which will be finalized at a later point.
- 其他代码被视为成功的最终化代码，可能会激活请求写入器来完成发送响应正文。一旦正文完全发送，请求 `count` 将递减。如果计数达到零，请求将被销毁，但客户端连接仍然可以用于其他请求。如果 `count` 为正数，则请求内部还有未完成的活动，在稍后的某个点上将会完成。



### 请求正文 Request body

For dealing with the body of a client request, nginx provides the `ngx_http_read_client_request_body(r, post_handler)` and `ngx_http_discard_request_body(r)` functions. The first function reads the request body and makes it available via the `request_body` request field. The second function instructs nginx to discard (read and ignore) the request body. One of these functions must be called for every request. Normally, the content handler makes the call.

​	在处理客户端请求的主体时，nginx 提供了 `ngx_http_read_client_request_body(r, post_handler)` 和 `ngx_http_discard_request_body(r)` 函数。第一个函数读取请求主体并通过 `request_body` 请求字段使其可用。第二个函数指示 nginx 放弃（读取并忽略）请求主体。必须为每个请求调用其中一个函数。通常，内容处理程序会进行调用。

Reading or discarding the client request body from a subrequest is not allowed. It must always be done in the main request. When a subrequest is created, it inherits the parent's `request_body` object which can be used by the subrequest if the main request has previously read the request body.

​	从子请求中读取或丢弃客户端请求主体是不允许的。它必须始终在主请求中完成。当创建子请求时，它继承父请求的 `request_body` 对象，如果主请求之前已经读取了请求主体，则子请求可以使用它。

The function `ngx_http_read_client_request_body(r, post_handler)` starts the process of reading the request body. When the body is completely read, the `post_handler` callback is called to continue processing the request. If the request body is missing or has already been read, the callback is called immediately. The function `ngx_http_read_client_request_body(r, post_handler)` allocates the `request_body` request field of type `ngx_http_request_body_t`. The field `bufs` of this object keeps the result as a buffer chain. The body can be saved in memory buffers or file buffers, if the capacity specified by the [client_body_buffer_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) directive is not enough to fit the entire body in memory.

​	函数 `ngx_http_read_client_request_body(r, post_handler)` 开始读取请求主体的过程。当主体完全读取完毕后，将调用 `post_handler` 回调函数以继续处理请求。如果请求主体丢失或已经被读取，则立即调用回调函数。函数 `ngx_http_read_client_request_body(r, post_handler)` 分配类型为 `ngx_http_request_body_t` 的 `request_body` 请求字段。此对象的 `bufs` 字段将缓冲链作为结果保存。如果 [client_body_buffer_size](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) 指令指定的容量不足以将整个主体放入内存中，则主体可以保存在内存缓冲区或文件缓冲区中。

The following example reads a client request body and returns its size.

​	以下示例读取客户端请求主体并返回其大小。

```
ngx_int_t
ngx_http_foo_content_handler(ngx_http_request_t *r)
{
    ngx_int_t  rc;

    rc = ngx_http_read_client_request_body(r, ngx_http_foo_init);

    if (rc >= NGX_HTTP_SPECIAL_RESPONSE) {
        /* error */
        return rc;
    }

    return NGX_DONE;
}


void
ngx_http_foo_init(ngx_http_request_t *r)
{
    off_t         len;
    ngx_buf_t    *b;
    ngx_int_t     rc;
    ngx_chain_t  *in, out;

    if (r->request_body == NULL) {
        ngx_http_finalize_request(r, NGX_HTTP_INTERNAL_SERVER_ERROR);
        return;
    }

    len = 0;

    for (in = r->request_body->bufs; in; in = in->next) {
        len += ngx_buf_size(in->buf);
    }

    b = ngx_create_temp_buf(r->pool, NGX_OFF_T_LEN);
    if (b == NULL) {
        ngx_http_finalize_request(r, NGX_HTTP_INTERNAL_SERVER_ERROR);
        return;
    }

    b->last = ngx_sprintf(b->pos, "%O", len);
    b->last_buf = (r == r->main) ? 1: 0;
    b->last_in_chain = 1;

    r->headers_out.status = NGX_HTTP_OK;
    r->headers_out.content_length_n = b->last - b->pos;

    rc = ngx_http_send_header(r);

    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        ngx_http_finalize_request(r, rc);
        return;
    }

    out.buf = b;
    out.next = NULL;

    rc = ngx_http_output_filter(r, &out);

    ngx_http_finalize_request(r, rc);
}
```

The following fields of the request determine how the request body is read:

​	请求的以下字段决定如何读取请求主体：

- `request_body_in_single_buf` - Read the body to a single memory buffer.
- `request_body_in_single_buf` - 将主体读取到单个内存缓冲区中。
- `request_body_in_file_only` - Always read the body to a file, even if fits in the memory buffer.
- `request_body_in_file_only` - 始终将主体读取到文件中，即使在内存缓冲区中也适用。
- `request_body_in_persistent_file` - Do not unlink the file immediately after creation. A file with this flag can be moved to another directory.
- `request_body_in_persistent_file` - 创建文件后不立即取消链接。带有此标志的文件可以移动到另一个目录中。
- `request_body_in_clean_file` - Unlink the file when the request is finalized. This can be useful when a file was supposed to be moved to another directory but was not moved for some reason.
- `request_body_in_clean_file` - 在请求最终化时取消链接文件。当文件本应移动到另一个目录但由于某种原因未移动时，这可能会很有用。
- `request_body_file_group_access` - Enable group access to the file by replacing the default 0600 access mask with 0660.
- `request_body_file_group_access` - 通过用 0660 替换默认的 0600 访问掩码来启用组访问文件。
- `request_body_file_log_level` - Severity level at which to log file errors.
- `request_body_file_log_level` - 记录文件错误的严重级别。
- `request_body_no_buffering` - Read the request body without buffering.
- `request_body_no_buffering` - 无缓冲读取请求主体。

The `request_body_no_buffering` flag enables the unbuffered mode of reading a request body. In this mode, after calling `ngx_http_read_client_request_body()`, the `bufs` chain might keep only a part of the body. To read the next part, call the `ngx_http_read_unbuffered_request_body(r)` function. The return value `NGX_AGAIN` and the request flag `reading_body` indicate that more data is available. If `bufs` is NULL after calling this function, there is nothing to read at the moment. The request callback `read_event_handler` will be called when the next part of request body is available.

​	`request_body_no_buffering` 标志启用了无缓冲的请求主体读取模式。在此模式下，在调用 `ngx_http_read_client_request_body()` 之后，`bufs` 链可能只保留部分主体。要读取下一部分，请调用 `ngx_http_read_unbuffered_request_body(r)` 函数。返回值 `NGX_AGAIN` 和请求标志 `reading_body` 表示更多数据可用。如果在调用此函数后 `bufs` 为 NULL，则当前没有要读取的内容。当下一部分请求主体可用时，将调用请求回调 `read_event_handler`。



### 请求正文过滤器 Request body filters

After a request body part is read, it's passed to the request body filter chain by calling the first body filter handler stored in the `ngx_http_top_request_body_filter` variable. It's assumed that every body handler calls the next handler in the chain until the final handler `ngx_http_request_body_save_filter(r, cl)` is called. This handler collects the buffers in `r->request_body->bufs` and writes them to a file if necessary. The last request body buffer has nonzero `last_buf` flag.

​	在读取请求主体的某个部分后，将通过调用存储在 `ngx_http_top_request_body_filter` 变量中的第一个主体过滤器处理程序将其传递给请求主体过滤器链。假定每个主体处理程序都调用链中的下一个处理程序，直到调用最终处理程序 `ngx_http_request_body_save_filter(r, cl)`。该处理程序将缓冲区链在 `r->request_body->bufs` 中进行收集，并在需要时将它们写入文件中。最后的请求主体缓冲区具有非零的 `last_buf` 标志。

If a filter is planning to delay data buffers, it should set the flag `r->request_body->filter_need_buffering` to `1` when called for the first time.

​	如果过滤器计划延迟数据缓冲区，则在首次调用时应将标志 `r->request_body->filter_need_buffering` 设置为 `1`。

Following is an example of a simple request body filter that delays request body by one second.

​	以下是一个简单请求主体过滤器的示例，它将请求主体延迟一秒钟。

```
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>


#define NGX_HTTP_DELAY_BODY  1000


typedef struct {
    ngx_event_t   event;
    ngx_chain_t  *out;
} ngx_http_delay_body_ctx_t;


static ngx_int_t ngx_http_delay_body_filter(ngx_http_request_t *r,
    ngx_chain_t *in);
static void ngx_http_delay_body_cleanup(void *data);
static void ngx_http_delay_body_event_handler(ngx_event_t *ev);
static ngx_int_t ngx_http_delay_body_init(ngx_conf_t *cf);


static ngx_http_module_t  ngx_http_delay_body_module_ctx = {
    NULL,                          /* preconfiguration */
    ngx_http_delay_body_init,      /* postconfiguration */

    NULL,                          /* create main configuration */
    NULL,                          /* init main configuration */

    NULL,                          /* create server configuration */
    NULL,                          /* merge server configuration */

    NULL,                          /* create location configuration */
    NULL                           /* merge location configuration */
};


ngx_module_t  ngx_http_delay_body_filter_module = {
    NGX_MODULE_V1,
    &ngx_http_delay_body_module_ctx, /* module context */
    NULL,                          /* module directives */
    NGX_HTTP_MODULE,               /* module type */
    NULL,                          /* init master */
    NULL,                          /* init module */
    NULL,                          /* init process */
    NULL,                          /* init thread */
    NULL,                          /* exit thread */
    NULL,                          /* exit process */
    NULL,                          /* exit master */
    NGX_MODULE_V1_PADDING
};


static ngx_http_request_body_filter_pt   ngx_http_next_request_body_filter;


static ngx_int_t
ngx_http_delay_body_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    ngx_int_t                   rc;
    ngx_chain_t                *cl, *ln;
    ngx_http_cleanup_t         *cln;
    ngx_http_delay_body_ctx_t  *ctx;

    ngx_log_debug0(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,
                   "delay request body filter");

    ctx = ngx_http_get_module_ctx(r, ngx_http_delay_body_filter_module);

    if (ctx == NULL) {
        ctx = ngx_pcalloc(r->pool, sizeof(ngx_http_delay_body_ctx_t));
        if (ctx == NULL) {
            return NGX_HTTP_INTERNAL_SERVER_ERROR;
        }

        ngx_http_set_ctx(r, ctx, ngx_http_delay_body_filter_module);

        r->request_body->filter_need_buffering = 1;
    }

    if (ngx_chain_add_copy(r->pool, &ctx->out, in) != NGX_OK) {
        return NGX_HTTP_INTERNAL_SERVER_ERROR;
    }

    if (!ctx->event.timedout) {
        if (!ctx->event.timer_set) {

            /* cleanup to remove the timer in case of abnormal termination */

            cln = ngx_http_cleanup_add(r, 0);
            if (cln == NULL) {
                return NGX_HTTP_INTERNAL_SERVER_ERROR;
            }

            cln->handler = ngx_http_delay_body_cleanup;
            cln->data = ctx;

            /* add timer */

            ctx->event.handler = ngx_http_delay_body_event_handler;
            ctx->event.data = r;
            ctx->event.log = r->connection->log;

            ngx_add_timer(&ctx->event, NGX_HTTP_DELAY_BODY);
        }

        return ngx_http_next_request_body_filter(r, NULL);
    }

    rc = ngx_http_next_request_body_filter(r, ctx->out);

    for (cl = ctx->out; cl; /* void */) {
        ln = cl;
        cl = cl->next;
        ngx_free_chain(r->pool, ln);
    }

    ctx->out = NULL;

    return rc;
}


static void
ngx_http_delay_body_cleanup(void *data)
{
    ngx_http_delay_body_ctx_t *ctx = data;

    if (ctx->event.timer_set) {
        ngx_del_timer(&ctx->event);
    }
}


static void
ngx_http_delay_body_event_handler(ngx_event_t *ev)
{
    ngx_connection_t    *c;
    ngx_http_request_t  *r;

    r = ev->data;
    c = r->connection;

    ngx_log_debug0(NGX_LOG_DEBUG_HTTP, c->log, 0,
                   "delay request body event");

    ngx_post_event(c->read, &ngx_posted_events);
}


static ngx_int_t
ngx_http_delay_body_init(ngx_conf_t *cf)
{
    ngx_http_next_request_body_filter = ngx_http_top_request_body_filter;
    ngx_http_top_request_body_filter = ngx_http_delay_body_filter;

    return NGX_OK;
}
```



### 响应 Response

In nginx an HTTP response is produced by sending the response header followed by the optional response body. Both header and body are passed through a chain of filters and eventually get written to the client socket. An nginx module can install its handler into the header or body filter chain and process the output coming from the previous handler.

​	在 nginx 中，HTTP 响应是通过发送响应头部，然后是可选的响应主体来生成的。头部和主体都通过一系列过滤器传递，并最终写入客户端套接字。nginx 模块可以将其处理程序安装到头部或主体过滤器链中，并处理来自上一个处理程序的输出。



#### 响应头部 Response header

The `ngx_http_send_header(r)` function sends the output header. Do not call this function until `r->headers_out` contains all of the data required to produce the HTTP response header. The `status` field in `r->headers_out` must always be set. If the response status indicates that a response body follows the header, `content_length_n` can be set as well. The default value for this field is `-1`, which means that the body size is unknown. In this case, chunked transfer encoding is used. To output an arbitrary header, append the `headers` list.

​	`ngx_http_send_header(r)` 函数发送输出头部。在 `r->headers_out` 包含生成 HTTP 响应头部所需的所有数据之前，请不要调用此函数。`r->headers_out` 中的 `status` 字段必须始终设置。如果响应状态表示响应主体随头部之后，则还可以设置 `content_length_n`。此字段的默认值为 `-1`，表示主体大小未知。在这种情况下，使用分块传输编码。要输出任意头部，请将其附加到 `headers` 列表中。

```
static ngx_int_t
ngx_http_foo_content_handler(ngx_http_request_t *r)
{
    ngx_int_t         rc;
    ngx_table_elt_t  *h;

    /* send header */
    /* 发送头部 */

    r->headers_out.status = NGX_HTTP_OK;
    r->headers_out.content_length_n = 3;

    /* X-Foo: foo */

    h = ngx_list_push(&r->headers_out.headers);
    if (h == NULL) {
        return NGX_ERROR;
    }

    h->hash = 1;
    ngx_str_set(&h->key, "X-Foo");
    ngx_str_set(&h->value, "foo");

    rc = ngx_http_send_header(r);

    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        return rc;
    }

    /* send body */
    /* 发送主体 */

    ...
}
```



#### 头部过滤器 Header filters

The `ngx_http_send_header(r)` function invokes the header filter chain by calling the first header filter handler stored in the `ngx_http_top_header_filter` variable. It's assumed that every header handler calls the next handler in the chain until the final handler `ngx_http_header_filter(r)` is called. The final header handler constructs the HTTP response based on `r->headers_out` and passes it to the `ngx_http_writer_filter` for output.

​	`ngx_http_send_header(r)` 函数通过调用存储在 `ngx_http_top_header_filter` 变量中的第一个头部过滤器处理程序来调用头部过滤器链。假定每个头部处理程序都在退出之前调用新添加的处理程序，直到调用最终处理程序 `ngx_http_header_filter(r)`。最终的头部处理程序根据 `r->headers_out` 构造 HTTP 响应，并将其传递给 `ngx_http_writer_filter` 进行输出。

To add a handler to the header filter chain, store its address in the global variable `ngx_http_top_header_filter` at configuration time. The previous handler address is normally stored in a static variable in a module and is called by the newly added handler before exiting.

​	要将处理程序添加到头部过滤器链中，请在配置时将其地址存储在全局变量 `ngx_http_top_header_filter` 中。通常，前一个处理程序地址会在模块中的静态变量中存储，并在新添加的处理程序调用之前由前一个处理程序调用。

The following example of a header filter module adds the HTTP header "`X-Foo: foo`" to every response with status `200`.

​	以下示例是一个头部过滤器模块的示例，它将 HTTP 头部 "`X-Foo: foo`" 添加到每个状态为 `200` 的响应中。

```
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>


static ngx_int_t ngx_http_foo_header_filter(ngx_http_request_t *r);
static ngx_int_t ngx_http_foo_header_filter_init(ngx_conf_t *cf);


static ngx_http_module_t  ngx_http_foo_header_filter_module_ctx = {
    NULL,                                   /* preconfiguration */
    ngx_http_foo_header_filter_init,        /* postconfiguration */

    NULL,                                   /* create main configuration */
    NULL,                                   /* init main configuration */

    NULL,                                   /* create server configuration */
    NULL,                                   /* merge server configuration */

    NULL,                                   /* create location configuration */
    NULL                                    /* merge location configuration */
};


ngx_module_t  ngx_http_foo_header_filter_module = {
    NGX_MODULE_V1,
    &ngx_http_foo_header_filter_module_ctx, /* module context */
    NULL,                                   /* module directives */
    NGX_HTTP_MODULE,                        /* module type */
    NULL,                                   /* init master */
    NULL,                                   /* init module */
    NULL,                                   /* init process */
    NULL,                                   /* init thread */
    NULL,                                   /* exit thread */
    NULL,                                   /* exit process */
    NULL,                                   /* exit master */
    NGX_MODULE_V1_PADDING
};


static ngx_http_output_header_filter_pt  ngx_http_next_header_filter;


static ngx_int_t
ngx_http_foo_header_filter(ngx_http_request_t *r)
{
    ngx_table_elt_t  *h;

    /*
     * The filter handler adds "X-Foo: foo" header
     * to every HTTP 200 response
     */
     /*
     * 该过滤器处理程序将 "X-Foo: foo" 头部添加到每个 HTTP 200 响应中
     */

    if (r->headers_out.status != NGX_HTTP_OK) {
        return ngx_http_next_header_filter(r);
    }

    h = ngx_list_push(&r->headers_out.headers);
    if (h == NULL) {
        return NGX_ERROR;
    }

    h->hash = 1;
    ngx_str_set(&h->key, "X-Foo");
    ngx_str_set(&h->value, "foo");

    return ngx_http_next_header_filter(r);
}


static ngx_int_t
ngx_http_foo_header_filter_init(ngx_conf_t *cf)
{
    ngx_http_next_header_filter = ngx_http_top_header_filter;
    ngx_http_top_header_filter = ngx_http_foo_header_filter;

    return NGX_OK;
}
```



### 响应主体 Response body

To send the response body, call the `ngx_http_output_filter(r, cl)` function. The function can be called multiple times. Each time, it sends a part of the response body in the form of a buffer chain. Set the `last_buf` flag in the last body buffer.

​	要发送响应主体，请调用 `ngx_http_output_filter(r, cl)` 函数。该函数可以多次调用。每次，它都会以缓冲链的形式发送响应主体的一部分。在最后的主体缓冲区中设置 `last_buf` 标志。

The following example produces a complete HTTP response with "foo" as its body. For the example to work as subrequest as well as a main request, the `last_in_chain` flag is set in the last buffer of the output. The `last_buf` flag is set only for the main request because the last buffer for a subrequest does not end the entire output.

​	以下示例生成了一个完整的 HTTP 响应，其中主体为 "foo"。为了使示例能够作为子请求和主请求使用，输出的最后一个缓冲区中设置了 `last_in_chain` 标志。对于主请求，仅为最后一个缓冲区设置了 `last_buf` 标志，因为对于子请求，最后的缓冲区并不结束整个输出。

```
static ngx_int_t
ngx_http_bar_content_handler(ngx_http_request_t *r)
{
    ngx_int_t     rc;
    ngx_buf_t    *b;
    ngx_chain_t   out;

    /* send header */
    /* 发送头部 */

    r->headers_out.status = NGX_HTTP_OK;
    r->headers_out.content_length_n = 3;

    rc = ngx_http_send_header(r);

    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        return rc;
    }

    /* send body */
    /* 发送主体 */

    b = ngx_calloc_buf(r->pool);
    if (b == NULL) {
        return NGX_ERROR;
    }

    b->last_buf = (r == r->main) ? 1: 0;
    b->last_in_chain = 1;

    b->memory = 1;

    b->pos = (u_char *) "foo";
    b->last = b->pos + 3;

    out.buf = b;
    out.next = NULL;

    return ngx_http_output_filter(r, &out);
}
```



### 响应主体过滤器 Response body filters

The function `ngx_http_output_filter(r, cl)` invokes the body filter chain by calling the first body filter handler stored in the `ngx_http_top_body_filter` variable. It's assumed that every body handler calls the next handler in the chain until the final handler `ngx_http_write_filter(r, cl)` is called.

​	函数 `ngx_http_output_filter(r, cl)` 调用主体过滤器链，通过调用存储在 `ngx_http_top_body_filter` 变量中的第一个主体过滤器处理程序。假设每个主体处理程序都调用链中的下一个处理程序，直到调用最终处理程序 `ngx_http_write_filter(r, cl)`。

A body filter handler receives a chain of buffers. The handler is supposed to process the buffers and pass a possibly new chain to the next handler. It's worth noting that the chain links `ngx_chain_t` of the incoming chain belong to the caller, and must not be reused or changed. Right after the handler completes, the caller can use its output chain links to keep track of the buffers it has sent. To save the buffer chain or to substitute some buffers before passing to the next filter, a handler needs to allocate its own chain links.

​	主体过滤器处理程序接收一个缓冲链。处理程序应处理缓冲区并将可能的新链传递给下一个处理程序。值得注意的是，传入链的链路 `ngx_chain_t` 属于调用者，并且不能被重用或更改。在处理程序完成后，调用者可以使用其输出链路来跟踪已发送的缓冲区。要保存缓冲区链或在传递给下一个过滤器之前替换一些缓冲区，处理程序需要分配自己的链路。

Following is an example of a simple body filter that counts the number of bytes in the body. The result is available as the `$counter` variable which can be used in the access log.

​	以下是一个简单的主体过滤器示例，它计算主体中的字节数。结果以 `$counter` 变量的形式提供，可以在访问日志中使用。

```
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>


typedef struct {
    off_t  count;
} ngx_http_counter_filter_ctx_t;


static ngx_int_t ngx_http_counter_body_filter(ngx_http_request_t *r,
    ngx_chain_t *in);
static ngx_int_t ngx_http_counter_variable(ngx_http_request_t *r,
    ngx_http_variable_value_t *v, uintptr_t data);
static ngx_int_t ngx_http_counter_add_variables(ngx_conf_t *cf);
static ngx_int_t ngx_http_counter_filter_init(ngx_conf_t *cf);


static ngx_http_module_t  ngx_http_counter_filter_module_ctx = {
    ngx_http_counter_add_variables,        /* preconfiguration */
    ngx_http_counter_filter_init,          /* postconfiguration */

    NULL,                                  /* create main configuration */
    NULL,                                  /* init main configuration */

    NULL,                                  /* create server configuration */
    NULL,                                  /* merge server configuration */

    NULL,                                  /* create location configuration */
    NULL                                   /* merge location configuration */
};


ngx_module_t  ngx_http_counter_filter_module = {
    NGX_MODULE_V1,
    &ngx_http_counter_filter_module_ctx,   /* module context */
    NULL,                                  /* module directives */
    NGX_HTTP_MODULE,                       /* module type */
    NULL,                                  /* init master */
    NULL,                                  /* init module */
    NULL,                                  /* init process */
    NULL,                                  /* init thread */
    NULL,                                  /* exit thread */
    NULL,                                  /* exit process */
    NULL,                                  /* exit master */
    NGX_MODULE_V1_PADDING
};


static ngx_http_output_body_filter_pt  ngx_http_next_body_filter;

static ngx_str_t  ngx_http_counter_name = ngx_string("counter");


static ngx_int_t
ngx_http_counter_body_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    ngx_chain_t                    *cl;
    ngx_http_counter_filter_ctx_t  *ctx;

    ctx = ngx_http_get_module_ctx(r, ngx_http_counter_filter_module);
    if (ctx == NULL) {
        ctx = ngx_pcalloc(r->pool, sizeof(ngx_http_counter_filter_ctx_t));
        if (ctx == NULL) {
            return NGX_ERROR;
        }

        ngx_http_set_ctx(r, ctx, ngx_http_counter_filter_module);
    }

    for (cl = in; cl; cl = cl->next) {
        ctx->count += ngx_buf_size(cl->buf);
    }

    return ngx_http_next_body_filter(r, in);
}


static ngx_int_t
ngx_http_counter_variable(ngx_http_request_t *r, ngx_http_variable_value_t *v,
    uintptr_t data)
{
    u_char                         *p;
    ngx_http_counter_filter_ctx_t  *ctx;

    ctx = ngx_http_get_module_ctx(r, ngx_http_counter_filter_module);
    if (ctx == NULL) {
        v->not_found = 1;
        return NGX_OK;
    }

    p = ngx_pnalloc(r->pool, NGX_OFF_T_LEN);
    if (p == NULL) {
        return NGX_ERROR;
    }

    v->data = p;
    v->len = ngx_sprintf(p, "%O", ctx->count) - p;
    v->valid = 1;
    v->no_cacheable = 0;
    v->not_found = 0;

    return NGX_OK;
}


static ngx_int_t
ngx_http_counter_add_variables(ngx_conf_t *cf)
{
    ngx_http_variable_t  *var;

    var = ngx_http_add_variable(cf, &ngx_http_counter_name, 0);
    if (var == NULL) {
        return NGX_ERROR;
    }

    var->get_handler = ngx_http_counter_variable;

    return NGX_OK;
}


static ngx_int_t
ngx_http_counter_filter_init(ngx_conf_t *cf)
{
    ngx_http_next_body_filter = ngx_http_top_body_filter;
    ngx_http_top_body_filter = ngx_http_counter_body_filter;

    return NGX_OK;
}
```



### 构建过滤器模块 Building filter modules

When writing a body or header filter, pay special attention to the filter's position in the filter order. There's a number of header and body filters registered by nginx standard modules. The nginx standard modules register a number of head and body filters and it's important to register a new filter module in the right place with respect to them. Normally, modules register filters in their postconfiguration handlers. The order in which filters are called during processing is obviously the reverse of the order in which they are registered.

​	在编写主体或头部过滤器时，要特别注意过滤器在过滤器顺序中的位置。nginx 标准模块注册了许多头部和主体过滤器。nginx 标准模块注册了许多头部和主体过滤器，重要的是要在与它们相关的正确位置注册新的过滤器模块。通常，模块会在其后配置处理程序中注册过滤器。处理请求期间调用过滤器的顺序与注册顺序相反。

For third-party filter modules nginx provides a special slot `HTTP_AUX_FILTER_MODULES`. To register a filter module in this slot, set the `ngx_module_type` variable to `HTTP_AUX_FILTER` in the module's configuration.

​	对于第三方过滤器模块，nginx 提供了一个名为 `HTTP_AUX_FILTER_MODULES` 的特殊插槽。要在此插槽中注册过滤器模块，请在模块的配置中将 `ngx_module_type` 变量设置为 `HTTP_AUX_FILTER`。

The following example shows a filter module config file assuming for a module with just one source file, `ngx_http_foo_filter_module.c`.

​	以下示例显示了一个过滤器模块配置文件，假设只有一个源文件 `ngx_http_foo_filter_module.c`。

```
ngx_module_type=HTTP_AUX_FILTER
ngx_module_name=ngx_http_foo_filter_module
ngx_module_srcs="$ngx_addon_dir/ngx_http_foo_filter_module.c"

. auto/module
```



### 缓冲区重用 Buffer reuse

When issuing or altering a stream of buffers, it's often desirable to reuse the allocated buffers. A standard and widely adopted approach in nginx code is to keep two buffer chains for this purpose: `free` and `busy`. The `free` chain keeps all free buffers, which can be reused. The `busy` chain keeps all buffers sent by the current module that are still in use by some other filter handler. A buffer is considered in use if its size is greater than zero. Normally, when a buffer is consumed by a filter, its `pos` (or `file_pos` for a file buffer) is moved towards `last` (`file_last` for a file buffer). Once a buffer is completely consumed, it's ready to be reused. To add newly freed buffers to the `free` chain it's enough to iterate over the `busy` chain and move the zero size buffers at the head of it to `free`. This operation is so common that there is a special function for it, `ngx_chain_update_chains(free, busy, out, tag)`. The function appends the output chain `out` to `busy` and moves free buffers from the top of `busy` to `free`. Only the buffers with the specified `tag` are reused. This lets a module reuse only the buffers that it allocated itself.

​	在发出或更改缓冲区流时，通常希望重用已分配的缓冲区。nginx 代码中通用且广泛采用的方法是保持两个缓冲链：`free` 和 `busy`。`free` 链保存所有可以重用的空闲缓冲区。`busy` 链保存当前模块发送的仍由某些其他过滤器处理程序使用的所有缓冲区。如果缓冲区的大小大于零，则认为它正在使用中。通常，当过滤器使用缓冲区时，它的 `pos`（或文件缓冲区的 `file_pos`）会向 `last`（文件缓冲区的 `file_last`）移动。一旦缓冲区完全使用完毕，它就可以被重用。要将新释放的缓冲区添加到 `free` 链，只需遍历 `busy` 链，并将其中头部的零大小缓冲区移动到 `free` 链的头部。此操作非常常见，因此有一个专门的函数可以执行此操作，即 `ngx_chain_update_chains(free, busy, out, tag)`。此函数将输出链 `out` 附加到 `busy` 链，并将 `busy` 链的顶部的空闲缓冲区移到 `free` 链。只有具有指定 `tag` 的缓冲区才会被重用。这使得模块只能重用自己分配的缓冲区。

The following example is a body filter that inserts the string “foo” before each incoming buffer. The new buffers allocated by the module are reused if possible. Note that for this example to work properly, setting up a [header filter](https://nginx.org/en/docs/dev/development_guide.html#http_header_filters) and resetting `content_length_n` to `-1` is also required, but the relevant code is not provided here.

​	以下示例是一个主体过滤器，它在每个传入的缓冲区之前插入字符串 “foo”。模块分配的新缓冲区在可能的情况下被重用。请注意，为了使此示例正常工作，还需要设置[头部过滤器](https://nginx.org/en/docs/dev/development_guide.html#http_header_filters)并将 `content_length_n` 重置为 `-1`，但此处未提供相关代码。

```
typedef struct {
    ngx_chain_t  *free;
    ngx_chain_t  *busy;
}  ngx_http_foo_filter_ctx_t;


ngx_int_t
ngx_http_foo_body_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    ngx_int_t                   rc;
    ngx_buf_t                  *b;
    ngx_chain_t                *cl, *tl, *out, **ll;
    ngx_http_foo_filter_ctx_t  *ctx;

    ctx = ngx_http_get_module_ctx(r, ngx_http_foo_filter_module);
    if (ctx == NULL) {
        ctx = ngx_pcalloc(r->pool, sizeof(ngx_http_foo_filter_ctx_t));
        if (ctx == NULL) {
            return NGX_ERROR;
        }

        ngx_http_set_ctx(r, ctx, ngx_http_foo_filter_module);
    }

    /* create a new chain "out" from "in" with all the changes */

    ll = &out;

    for (cl = in; cl; cl = cl->next) {

        /* append "foo" in a reused buffer if possible */

        tl = ngx_chain_get_free_buf(r->pool, &ctx->free);
        if (tl == NULL) {
            return NGX_ERROR;
        }

        b = tl->buf;
        b->tag = (ngx_buf_tag_t) &ngx_http_foo_filter_module;
        b->memory = 1;
        b->pos = (u_char *) "foo";
        b->last = b->pos + 3;

        *ll = tl;
        ll = &tl->next;

        /* append the next incoming buffer */

        tl = ngx_alloc_chain_link(r->pool);
        if (tl == NULL) {
            return NGX_ERROR;
        }

        tl->buf = cl->buf;
        *ll = tl;
        ll = &tl->next;
    }

    *ll = NULL;

    /* send the new chain */

    rc = ngx_http_next_body_filter(r, out);

    /* update "busy" and "free" chains for reuse */

    ngx_chain_update_chains(r->pool, &ctx->free, &ctx->busy, &out,
                            (ngx_buf_tag_t) &ngx_http_foo_filter_module);

    return rc;
}
```



### 负载均衡 Load balancing

The [ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) provides the basic functionality needed to pass requests to remote servers. Modules that implement specific protocols, such as HTTP or FastCGI, use this functionality. The module also provides an interface for creating custom load-balancing modules and implements a default round-robin method.

​	[ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) 提供了将请求传递给远程服务器所需的基本功能。实现特定协议（如 HTTP 或 FastCGI）的模块使用此功能。该模块还提供了一个接口用于创建自定义负载均衡模块，并实现了默认的轮询方法。

The [least_conn](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn) and [hash](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) modules implement alternative load-balancing methods, but are actually implemented as extensions of the upstream round-robin module and share a lot of code with it, such as the representation of a server group. The [keepalive](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) module is an independent module that extends upstream functionality.

​	[least_conn](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn) 和 [hash](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) 模块实现了替代的负载均衡方法，但实际上是作为上游轮询模块的扩展来实现的，并且与之共享许多代码，例如服务器组的表示。[keepalive](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) 模块是一个独立的模块，扩展了上游功能。

The [ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) can be configured explicitly by placing the corresponding [upstream](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) block into the configuration file, or implicitly by using directives such as [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) that accept a URL that gets evaluated at some point into a list of servers. The alternative load-balancing methods are available only with an explicit upstream configuration. The upstream module configuration has its own directive context `NGX_HTTP_UPS_CONF`. The structure is defined as follows:

​	[ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) 可以通过将相应的 [upstream](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) 块放入配置文件中来进行显式配置，也可以通过使用接受 URL 的指令（例如 [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)）来进行隐式配置，该 URL 在某个时刻会计算为服务器列表。替代的负载均衡方法仅在显式上游配置中可用。上游模块配置具有自己的指令上下文 `NGX_HTTP_UPS_CONF`。结构定义如下：

```
struct ngx_http_upstream_srv_conf_s {
    ngx_http_upstream_peer_t         peer;
    void                           **srv_conf;

    ngx_array_t                     *servers;  /* ngx_http_upstream_server_t */

    ngx_uint_t                       flags;
    ngx_str_t                        host;
    u_char                          *file_name;
    ngx_uint_t                       line;
    in_port_t                        port;
    ngx_uint_t                       no_port;  /* unsigned no_port:1 */

#if (NGX_HTTP_UPSTREAM_ZONE)
    ngx_shm_zone_t                  *shm_zone;
#endif
};
```



- `srv_conf` — Configuration context of upstream modules.

- `srv_conf` — 上游模块的配置上下文。

- `servers` — Array of `ngx_http_upstream_server_t`, the result of parsing a set of [server](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) directives in the `upstream` block.

- `servers` — `ngx_http_upstream_server_t` 数组，这是在 `upstream` 块中解析一组 [server](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) 指令的结果。

- `flags` — Flags that mostly mark which features are supported by the load-balancing method. The features are configured as parameters of the server directive:

- `flags` — 标志，大部分标志标识负载均衡方法支持的特性。这些特性是通过服务器指令的参数进行配置的：

  - `NGX_HTTP_UPSTREAM_CREATE` — Distinguishes explicitly defined upstreams from those that are automatically created by the [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive and “friends” (FastCGI, SCGI, etc.)
  - `NGX_HTTP_UPSTREAM_CREATE` — 将显式定义的上游与由 [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) 指令和类似指令（FastCGI、SCGI 等）自动创建的上游区分开。
  - `NGX_HTTP_UPSTREAM_WEIGHT` — The “`weight`” parameter is supported
  - `NGX_HTTP_UPSTREAM_WEIGHT` — 支持 “`weight`” 参数。
  - `NGX_HTTP_UPSTREAM_MAX_FAILS` — The “`max_fails`” parameter is supported
  - `NGX_HTTP_UPSTREAM_MAX_FAILS` — 支持 “`max_fails`” 参数。
  - `NGX_HTTP_UPSTREAM_FAIL_TIMEOUT` — The “`fail_timeout`” parameter is supported
  - `NGX_HTTP_UPSTREAM_FAIL_TIMEOUT` — 支持 “`fail_timeout`” 参数。
  - `NGX_HTTP_UPSTREAM_DOWN` — The “`down`” parameter is supported
  - `NGX_HTTP_UPSTREAM_DOWN` — 支持 “`down`” 参数。
  - `NGX_HTTP_UPSTREAM_BACKUP` — The “`backup`” parameter is supported
  - `NGX_HTTP_UPSTREAM_BACKUP` — 支持 “`backup`” 参数。
  - `NGX_HTTP_UPSTREAM_MAX_CONNS` — The “`max_conns`” parameter is supported
  - `NGX_HTTP_UPSTREAM_MAX_CONNS` — 支持 “`max_conns`” 参数。

- `host` — Name of the upstream.

- `host` — 上游的名称。

- `file_name, line` — Name of the configuration file and the line where the `upstream` block is located.

- `file_name`, `line` — 配置文件的名称和 `upstream` 块所在的行号。

- `port` and `no_port` — Not used for explicitly defined upstream groups.

- `port` 和 `no_port` — 对于显式定义的上游组不使用。

- `shm_zone` — Shared memory zone used by this upstream group, if any.

- `shm_zone` — 由此上游组使用的共享内存区域（如果有的话）。

- `peer` — object that holds generic methods for initializing upstream configuration:
  
  `peer` — 对象，保存初始化上游配置的通用方法：

  > ```
  > typedef struct {
  >     ngx_http_upstream_init_pt        init_upstream;
  >     ngx_http_upstream_init_peer_pt   init;
  >     void                            *data;
  > } ngx_http_upstream_peer_t;
  > ```
  
  A module that implements a load-balancing algorithm must set these methods and initialize private `data`. If `init_upstream`was not initialized during configuration parsing, `ngx_http_upstream_module` sets it to the default `ngx_http_upstream_init_round_robin` algorithm.

  ​	实现负载均衡算法的模块必须设置这些方法并初始化私有 `data`。如果在配置解析期间未初始化 `init_upstream`，则 `ngx_http_upstream_module` 会将其设置为默认的 `ngx_http_upstream_init_round_robin` 算法。 

  - `init_upstream(cf, us)` — Configuration-time method responsible for initializing a group of servers and initializing the `init()` method in case of success. A typical load-balancing module uses a list of servers in the `upstream` block to create an efficient data structure that it uses and saves its own configuration to the `data` field.
  - `init_upstream(cf, us)` — 配置时的方法，负责初始化一组服务器并在成功的情况下初始化 `init()` 方法。典型的负载均衡模块使用 `upstream` 块中的服务器列表来创建一个它使用的有效数据结构，并将自己的配置保存到 `data` 字段。
  - `init(r, us)` — Initializes a per-request `ngx_http_upstream_peer_t.peer` structure that is used for load balancing (not to be confused with the `ngx_http_upstream_srv_conf_t.peer` described above which is per-upstream). It is passed as the `data` argument to all callbacks that deal with server selection.
  - `init(r, us)` — 初始化每个请求的 `ngx_http_upstream_peer_t.peer` 结构，该结构用于负载均衡（不要与上述的 `ngx_http_upstream_srv_conf_t.peer` 混淆，后者是每个上游的）。它作为 `data` 参数传递给所有处理服务器选择的回调函数（详见下文）。



When nginx has to pass a request to another host for processing, it uses the configured load-balancing method to obtain an address to connect to. The method is obtained from the `ngx_http_upstream_t.peer` object of type `ngx_peer_connection_t`:

​	当 nginx 需要将请求传递到另一个主机进行处理时，它会使用配置的负载均衡方法来获取要连接的地址。该方法是从类型为 `ngx_peer_connection_t` 的 `ngx_http_upstream_t.peer` 对象中获取的：

```
struct ngx_peer_connection_s {
    ...

    struct sockaddr                 *sockaddr;
    socklen_t                        socklen;
    ngx_str_t                       *name;

    ngx_uint_t                       tries;

    ngx_event_get_peer_pt            get;
    ngx_event_free_peer_pt           free;
    ngx_event_notify_peer_pt         notify;
    void                            *data;

#if (NGX_SSL || NGX_COMPAT)
    ngx_event_set_peer_session_pt    set_session;
    ngx_event_save_peer_session_pt   save_session;
#endif

    ...
};
```

The structure has the following fields:

​	结构具有以下字段：

- `sockaddr`, `socklen`, `name` — Address of the upstream server to connect to; this is the output parameter of a load-balancing method.
- `sockaddr`, `socklen`, `name` — 要连接到的上游服务器的地址；这是负载均衡方法的输出参数。
- `data` — The per-request data of a load-balancing method; keeps the state of the selection algorithm and usually includes the link to the upstream configuration. It is passed as an argument to all methods that deal with server selection (see [below](https://nginx.org/en/docs/dev/development_guide.html#lb_method_get)).
- `data` — 负载均衡方法的每个请求数据；保持选择算法的状态，通常包括指向上游配置的链接。它作为传递给处理服务器选择的所有方法的参数（参见 [下面](https://nginx.org/en/docs/dev/development_guide.html#lb_method_get)）。
- `tries` — Allowed [number](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream_tries) of attempts to connect to an upstream server.
- `tries` — 允许连接到上游服务器的尝试次数。
- `get`, `free`, `notify`, `set_session`, and `save_session` - Methods of the load-balancing module, described below.
- `get`, `free`, `notify`, `set_session`, 和 `save_session` - 负载均衡模块的方法，下面将进行描述。



All methods accept at least two arguments: a peer connection object `pc` and the `data` created by `ngx_http_upstream_srv_conf_t.peer.init()`. Note that it might differ from `pc.data` due to “chaining” of load-balancing modules.

​	所有方法至少接受两个参数：一个对等连接对象 `pc` 和 `ngx_http_upstream_srv_conf_t.peer.init()` 创建的 `data`。注意，由于负载均衡模块的 “链接”，它可能与 `pc.data` 不同。

- `get(pc, data)` — The method called when the upstream module is ready to pass a request to an upstream server and needs to know its address. The method has to fill the `sockaddr`, `socklen`, and `name` fields of `ngx_peer_connection_t` structure. The return is one of:
  
- `get(pc, data)` — 当上游模块准备将请求传递给上游服务器并且需要知道其地址时调用该方法。该方法必须填充 `ngx_peer_connection_t` 结构的 `sockaddr`、`socklen` 和 `name` 字段。返回值是以下之一：
  
  - `NGX_OK` — Server was selected.
  - `NGX_OK` — 已选择服务器。
  - `NGX_ERROR` — Internal error occurred.
  - `NGX_ERROR` — 发生内部错误。
  - `NGX_BUSY` — no servers are currently available. This can happen due to many reasons, including: the dynamic server group is empty, all servers in the group are in the failed state, or all servers in the group are already handling the maximum number of connections.
  - `NGX_BUSY` — 当前没有可用的服务器。这可能由于多种原因，包括动态服务器组为空，组中的所有服务器都处于故障状态，或者组中的所有服务器已经处理了最大数量的连接。
  - `NGX_DONE` — the underlying connection was reused and there is no need to create a new connection to the upstream server. This value is set by the `keepalive` module.
  - `NGX_DONE` — 底层连接已被重用，无需创建新的连接到上游服务器。此值由 `keepalive` 模块设置。
  
- `free(pc, data, state)` — The method called when an upstream module has finished work with a particular server. The `state` argument is the completion status of the upstream connection, a bitmask with the following possible values:
  
- `free(pc, data, state)` — 当上游模块已经完成对特定服务器的处理时调用该方法。`state` 参数是上游连接的完成状态，是一个位掩码，具有以下可能的值：
  
  - `NGX_PEER_FAILED` — Attempt was [unsuccessful](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails)
  - `NGX_PEER_FAILED` — 尝试[未成功](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails)
  - `NGX_PEER_NEXT` — A special case when upstream server returns codes `403` or `404`, which are not considered a [failure](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails).
  - `NGX_PEER_NEXT` — 特殊情况，上游服务器返回代码 `403` 或 `404`，不被视为 [失败](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails)
  - `NGX_PEER_KEEPALIVE` — Currently unused
  - `NGX_PEER_KEEPALIVE` — 当前未使用
  
  This method also decrements the `tries`  counter.
  
  该方法还会减少 `tries` 字段的值。
  
- `notify(pc, data, type)` — Currently unused in the OSS version.

- `notify(pc, data, type)` — 目前在开源版本中未使用。

- `set_session(pc, data)` and `save_session(pc, data)` — SSL-specific methods that enable caching sessions to upstream servers. The implementation is provided by the round-robin balancing method.

- `set_session(pc, data)` 和 `save_session(pc, data)` — 针对 SSL 的特定方法，用于将会话缓存到上游服务器。这些方法的实现由轮询负载均衡方法提供。





## 示例 Examples

The [nginx-dev-examples](http://hg.nginx.org/nginx-dev-examples) repository provides nginx module examples.

​	[nginx-dev-examples](http://hg.nginx.org/nginx-dev-examples) 仓库提供了 nginx 模块示例。



## 代码风格 Code style



### 通用规则 General rules



- maximum text width is 80 characters
- indentation is 4 spaces
- no tabs, no trailing spaces
- list elements on the same line are separated with spaces
- hexadecimal literals are lowercase
- file names, function and type names, and global variables have the `ngx_` or more specific prefix such as `ngx_http_` and `ngx_mail_`
- 最大文本宽度为 80 个字符
- 缩进为 4 个空格
- 不使用制表符，不留下尾随空格
- 同一行上的列表元素使用空格分隔
- 十六进制字面量使用小写字母
- 文件名、函数和类型名称以及全局变量都具有 `ngx_` 或更具体的前缀，例如 `ngx_http_` 和 `ngx_mail_`



```
size_t
ngx_utf8_length(u_char *p, size_t n)
{
    u_char  c, *last;
    size_t  len;

    last = p + n;

    for (len = 0; p < last; len++) {

        c = *p;

        if (c < 0x80) {
            p++;
            continue;
        }

        if (ngx_utf8_decode(&p, last - p) > 0x10ffff) {
            /* invalid UTF-8 */
            return n;
        }
    }

    return len;
}
```





### 文件 Files

A typical source file may contain the following sections separated by two empty lines:

​	一个典型的源文件可能包含以下通过两个空行分隔的部分：

- copyright statements
- includes
- preprocessor definitions
- type definitions
- function prototypes
- variable definitions
- function definitions
- 版权声明
- 包含
- 预处理器定义
- 类型定义
- 函数原型
- 变量定义
- 函数定义



Copyright statements look like this:

​	版权声明的格式如下：

```
/*
 * Copyright (C) Author Name
 * Copyright (C) Organization, Inc.
 */
```

If the file is modified significantly, the list of authors should be updated, the new author is added to the top.

​	如果文件经过重大修改，作者列表应进行更新，新作者将被添加到列表的顶部。

The `ngx_config.h` and `ngx_core.h` files are always included first, followed by one of `ngx_http.h`, `ngx_stream.h`, or `ngx_mail.h`. Then follow optional external header files:

​	`ngx_config.h` 和 `ngx_core.h` 文件始终首先被包含，然后是 `ngx_http.h`、`ngx_stream.h` 或 `ngx_mail.h` 中的一个。然后是可选的外部头文件：

```
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

#include <libxml/parser.h>
#include <libxml/tree.h>
#include <libxslt/xslt.h>

#if (NGX_HAVE_EXSLT)
#include <libexslt/exslt.h>
#endif
```



Header files should include the so called "header protection":

​	头文件应该包含所谓的“头部保护”：

```
#ifndef _NGX_PROCESS_CYCLE_H_INCLUDED_
#define _NGX_PROCESS_CYCLE_H_INCLUDED_
...
#endif /* _NGX_PROCESS_CYCLE_H_INCLUDED_ */
```





### 注释 Comments



- “`//`” comments are not used

- text is written in English, American spelling is preferred

- multi-line comments are formatted like this:

- 不使用“`//`”注释

- 注释使用英语编写，美式拼写首选

- 多行注释的格式如下：

  > ```
  > /*
  >  * The red-black tree code is based on the algorithm described in
  >  * the "Introduction to Algorithms" by Cormen, Leiserson and Rivest.
  >  */
  >  /*
  >  * 红黑树代码基于 Cormen、Leiserson 和 Rivest 在《算法导论》中描述的算法。
  >  */
  > ```

  > ```
  > /* find the server configuration for the address:port */
  > /* 查找地址：端口 的服务器配置 */
  > ```





### 预处理器 Preprocessor

Macro names start from `ngx_` or `NGX_` (or more specific) prefix. Macro names for constants are uppercase. Parameterized macros and macros for initializers are lowercase. The macro name and value are separated by at least two spaces:

​	宏名以 `ngx_` 或 `NGX_`（或更具体的前缀）开头。用于常量的宏名使用大写字母。带参数的宏和初始化器的宏使用小写字母。宏名和值之间至少有两个空格：

```
#define NGX_CONF_BUFFER  4096

#define ngx_buf_in_memory(b)  (b->temporary || b->memory || b->mmap)

#define ngx_buf_size(b)                                                      \
    (ngx_buf_in_memory(b) ? (off_t) (b->last - b->pos):                      \
                            (b->file_last - b->file_pos))

#define ngx_null_string  { 0, NULL }
```

Conditions are inside parentheses, negation is outside:

​	条件放在括号内，否定放在括号外：

```
#if (NGX_HAVE_KQUEUE)
...
#elif ((NGX_HAVE_DEVPOLL && !(NGX_TEST_BUILD_DEVPOLL)) \
       || (NGX_HAVE_EVENTPORT && !(NGX_TEST_BUILD_EVENTPORT)))
...
#elif (NGX_HAVE_EPOLL && !(NGX_TEST_BUILD_EPOLL))
...
#elif (NGX_HAVE_POLL)
...
#else /* select */
...
#endif /* NGX_HAVE_KQUEUE */
```





### 类型 Types

Type names end with the “`_t`” suffix. A defined type name is separated by at least two spaces:

​	类型名以“`_t`”后缀结束。定义的类型名之间至少有两个空格：

```
typedef ngx_uint_t  ngx_rbtree_key_t;
```



Structure types are defined using `typedef`. Inside structures, member types and names are aligned:

​	使用 `typedef` 定义结构类型。在结构内部，成员类型和名称是对齐的：

```
typedef struct {
    size_t      len;
    u_char     *data;
} ngx_str_t;
```

Keep alignment identical among different structures in the file. A structure that points to itself has the name, ending with “`_s`”. Adjacent structure definitions are separated with two empty lines:

​	在文件中保持不同结构之间的对齐相同。指向自身的结构以“`_s`”结尾的名称。相邻的结构定义之间用两个空行分隔：

```
typedef struct ngx_list_part_s  ngx_list_part_t;

struct ngx_list_part_s {
    void             *elts;
    ngx_uint_t        nelts;
    ngx_list_part_t  *next;
};


typedef struct {
    ngx_list_part_t  *last;
    ngx_list_part_t   part;
    size_t            size;
    ngx_uint_t        nalloc;
    ngx_pool_t       *pool;
} ngx_list_t;
```

Each structure member is declared on its own line:

​	每个结构成员单独声明一行：

```
typedef struct {
    ngx_uint_t        hash;
    ngx_str_t         key;
    ngx_str_t         value;
    u_char           *lowcase_key;
} ngx_table_elt_t;
```

Function pointers inside structures have defined types ending with “`_pt`”:

​	结构内的函数指针具有以“`_pt`”结尾的定义类型：

```
typedef ssize_t (*ngx_recv_pt)(ngx_connection_t *c, u_char *buf, size_t size);
typedef ssize_t (*ngx_recv_chain_pt)(ngx_connection_t *c, ngx_chain_t *in,
    off_t limit);
typedef ssize_t (*ngx_send_pt)(ngx_connection_t *c, u_char *buf, size_t size);
typedef ngx_chain_t *(*ngx_send_chain_pt)(ngx_connection_t *c, ngx_chain_t *in,
    off_t limit);

typedef struct {
    ngx_recv_pt        recv;
    ngx_recv_chain_pt  recv_chain;
    ngx_recv_pt        udp_recv;
    ngx_send_pt        send;
    ngx_send_pt        udp_send;
    ngx_send_chain_pt  udp_send_chain;
    ngx_send_chain_pt  send_chain;
    ngx_uint_t         flags;
} ngx_os_io_t;
```



Enumerations have types ending with “`_e`”:

​	枚举类型以“`_e`”结尾的类型名：

```
typedef enum {
    ngx_http_fastcgi_st_version = 0,
    ngx_http_fastcgi_st_type,
    ...
    ngx_http_fastcgi_st_padding
} ngx_http_fastcgi_state_e;
```





### 变量 Variables

Variables are declared sorted by length of a base type, then alphabetically. Type names and variable names are aligned. The type and name “columns” are separated with two spaces. Large arrays are put at the end of a declaration block:

​	变量按基本类型的长度进行排序，然后按字母顺序排列。类型名和变量名是对齐的。类型和名称“列”之间用两个空格分隔。大型数组放在声明块的末尾：

```
u_char                      |  | *rv, *p;
ngx_conf_t                  |  | *cf;
ngx_uint_t                  |  |  i, j, k;
unsigned int                |  |  len;
struct sockaddr             |  | *sa;
const unsigned char         |  | *data;
ngx_peer_connection_t       |  | *pc;
ngx_http_core_srv_conf_t    |  |**cscfp;
ngx_http_upstream_srv_conf_t|  | *us, *uscf;
u_char                      |  |  text[NGX_SOCKADDR_STRLEN];
```



Static and global variables may be initialized on declaration:

​	静态和全局变量可以在声明时初始化：

```
static ngx_str_t  ngx_http_memcached_key = ngx_string("memcached_key");
```



```
static ngx_uint_t  mday[] = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
```



```
static uint32_t  ngx_crc32_table16[] = {
    0x00000000, 0x1db71064, 0x3b6e20c8, 0x26d930ac,
    ...
    0x9b64c2b0, 0x86d3d2d4, 0xa00ae278, 0xbdbdf21c
};
```



There is a bunch of commonly used type/name combinations:

​	存在一些常用的类型/名称组合：

```
u_char                        *rv;
ngx_int_t                      rc;
ngx_conf_t                    *cf;
ngx_connection_t              *c;
ngx_http_request_t            *r;
ngx_peer_connection_t         *pc;
ngx_http_upstream_srv_conf_t  *us, *uscf;
```





### 函数 Functions

All functions (even static ones) should have prototypes. Prototypes include argument names. Long prototypes are wrapped with a single indentation on continuation lines:

​	所有函数（即使是静态函数）都应该有原型。原型包括参数名称。长的原型在续行时会有单个缩进：

```
static char *ngx_http_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
static ngx_int_t ngx_http_init_phases(ngx_conf_t *cf,
    ngx_http_core_main_conf_t *cmcf);

static char *ngx_http_merge_servers(ngx_conf_t *cf,
    ngx_http_core_main_conf_t *cmcf, ngx_http_module_t *module,
    ngx_uint_t ctx_index);
```

The function name in a definition starts with a new line. The function body opening and closing braces are on separate lines. The body of a function is indented. There are two empty lines between functions:

​	定义中的函数名以新行开头。函数体的开头和结尾大括号在不同的行上。函数体缩进。函数之间有两个空行：

```
static ngx_int_t
ngx_http_find_virtual_server(ngx_http_request_t *r, u_char *host, size_t len)
{
    ...
}


static ngx_int_t
ngx_http_add_addresses(ngx_conf_t *cf, ngx_http_core_srv_conf_t *cscf,
    ngx_http_conf_port_t *port, ngx_http_listen_opt_t *lsopt)
{
    ...
}
```

There is no space after the function name and opening parenthesis. Long function calls are wrapped such that continuation lines start from the position of the first function argument. If this is impossible, format the first continuation line such that it ends at position 79:

​	在函数名和开括号之间没有空格。将长的函数调用包装起来，使得续行从第一个函数参数的位置开始。如果不可能这样做，请将第一行续行格式化，使其在第79个位置结束：

```
ngx_log_debug2(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,
               "http header: \"%V: %V\"",
               &h->key, &h->value);

hc->busy = ngx_palloc(r->connection->pool,
                  cscf->large_client_header_buffers.num * sizeof(ngx_buf_t *));
```

The `ngx_inline` macro should be used instead of `inline`:

​	应该使用 `ngx_inline` 宏而不是 `inline`：

```
static ngx_inline void ngx_cpuid(uint32_t i, uint32_t *buf);
```





## 表达式 Expressions

Binary operators except “`.`” and “`−>`” should be separated from their operands by one space. Unary operators and subscripts are not separated from their operands by spaces:

​	二元运算符，除了“`.`”和“`−>`”，应该与其操作数之间有一个空格分隔。一元运算符和下标不与其操作数之间有空格分隔：

```
width = width * 10 + (*fmt++ - '0');
```



```
ch = (u_char) ((decoded << 4) + (ch - '0'));
```



```
r->exten.data = &r->uri.data[i + 1];
```



Type casts are separated by one space from casted expressions. An asterisk inside type cast is separated with space from type name:

​	类型转换与转换的表达式之间用一个空格分隔。类型转换内部的星号与类型名之间用空格分隔：

```
len = ngx_sock_ntop((struct sockaddr *) sin6, p, len, 1);
```



If an expression does not fit into single line, it is wrapped. The preferred point to break a line is a binary operator. The continuation line is lined up with the start of expression:

​	如果一个表达式不能放入单行中，则将其换行。换行的首选位置是二元运算符。续行与表达式的起始对齐：

```
if (status == NGX_HTTP_MOVED_PERMANENTLY
    || status == NGX_HTTP_MOVED_TEMPORARILY
    || status == NGX_HTTP_SEE_OTHER
    || status == NGX_HTTP_TEMPORARY_REDIRECT
    || status == NGX_HTTP_PERMANENT_REDIRECT)
{
    ...
}
```



```
p->temp_file->warn = "an upstream response is buffered "
                     "to a temporary file";
```

As a last resort, it is possible to wrap an expression so that the continuation line ends at position 79:

​	作为最后的选择，可以将一个表达式包装起来，以使续行在第79个位置结束：

```
hinit->hash = ngx_pcalloc(hinit->pool, sizeof(ngx_hash_wildcard_t)
                                     + size * sizeof(ngx_hash_elt_t *));
```

The above rules also apply to sub-expressions, where each sub-expression has its own indentation level:

​	上述规则也适用于子表达式，其中每个子表达式都有自己的缩进级别：

```
if (((u->conf->cache_use_stale & NGX_HTTP_UPSTREAM_FT_UPDATING)
     || c->stale_updating) && !r->background
    && u->conf->cache_background_update)
{
    ...
}
```

Sometimes, it is convenient to wrap an expression after a cast. In this case, the continuation line is indented:

​	有时，在类型转换之后将表达式包装起来是方便的。在这种情况下，续行是缩进的：

```
node = (ngx_rbtree_node_t *)
           ((u_char *) lr - offsetof(ngx_rbtree_node_t, color));
```



Pointers are explicitly compared to `NULL` (not `0`):

​	指针与 `NULL` 进行显式比较（而不是 `0`）：

```
if (ptr != NULL) {
    ...
}
```





### 条件和循环 Conditionals and Loops

The “`if`” keyword is separated from the condition by one space. Opening brace is located on the same line, or on a dedicated line if the condition takes several lines. Closing brace is located on a dedicated line, optionally followed by “`else if` / `else`”. Usually, there is an empty line before the “`else if` / `else`” part:

​	“`if`”关键字与条件之间用一个空格分隔。开括号位于同一行上，或者位于一个专用的行上，如果条件占据多行。闭括号位于一个专用的行上，可选地跟在“`else if` / `else`”之后。通常，在“`else if` / `else`”部分之前有一个空行：

```
if (node->left == sentinel) {
    temp = node->right;
    subst = node;

} else if (node->right == sentinel) {
    temp = node->left;
    subst = node;

} else {
    subst = ngx_rbtree_min(node->right, sentinel);

    if (subst->left != sentinel) {
        temp = subst->left;

    } else {
        temp = subst->right;
    }
}
```



Similar formatting rules are applied to “`do`” and “`while`” loops:

​	类似的格式化规则适用于“`do`”和“`while`”循环：

```
while (p < last && *p == ' ') {
    p++;
}
```



```
do {
    ctx->node = rn;
    ctx = ctx->next;
} while (ctx);
```



The “`switch`” keyword is separated from the condition by one space. Opening brace is located on the same line. Closing brace is located on a dedicated line. The “`case`” keywords are lined up with “`switch`”:

​	“`switch`”关键字与条件之间用一个空格分隔。开括号位于同一行上。闭括号位于一个专用的行上。与“`switch`”对齐的“`case`”关键字：

```
switch (ch) {
case '!':
    looked = 2;
    state = ssi_comment0_state;
    break;

case '<':
    copy_end = p;
    break;

default:
    copy_end = p;
    looked = 0;
    state = ssi_start_state;
    break;
}
```



Most “`for`” loops are formatted like this:

​	大多数“`for`”循环的格式如下：

```
for (i = 0; i < ccf->env.nelts; i++) {
    ...
}
```



```
for (q = ngx_queue_head(locations);
     q != ngx_queue_sentinel(locations);
     q = ngx_queue_next(q))
{
    ...
}
```

If some part of the “`for`” statement is omitted, this is indicated by the “`/* void */`” comment:

​	如果“`for`”语句的某个部分被省略，这可以通过“`/* void */`”注释来表示：

```
for (i = 0; /* void */ ; i++) {
    ...
}
```

A loop with an empty body is also indicated by the “`/* void */`” comment which may be put on the same line:

​	一个空主体的循环也通过“`/* void */`”注释来表示，可以放在同一行上：

```
for (cl = *busy; cl->next; cl = cl->next) { /* void */ }
```

An endless loop looks like this:

​	无限循环如下所示：

```
for ( ;; ) {
    ...
}
```





### 标签 Labels

Labels are surrounded with empty lines and are indented at the previous level:

​	标签用空行包围，并在前一级进行缩进：

```
    if (i == 0) {
        u->err = "host not found";
        goto failed;
    }

    u->addrs = ngx_pcalloc(pool, i * sizeof(ngx_addr_t));
    if (u->addrs == NULL) {
        goto failed;
    }

    u->naddrs = i;

    ...

    return NGX_OK;

failed:

    freeaddrinfo(res);
    return NGX_ERROR;
```





## 调试内存问题 Debugging memory issues

To debug memory issues such as buffer overruns or use-after-free errors, you can use the [AddressSanitizer](https://en.wikipedia.org/wiki/AddressSanitizer) (ASan) supported by some modern compilers. To enable ASan with `gcc` and `clang`, use the `-fsanitize=address` compiler and linker option. When building nginx, this can be done by adding the option to `--with-cc-opt` and `--with-ld-opt` parameters of the `configure` script.

​	要调试诸如缓冲区溢出或使用后释放错误等内存问题，您可以使用一些现代编译器支持的 [AddressSanitizer](https://en.wikipedia.org/wiki/AddressSanitizer) (ASan)。要在 `gcc` 和 `clang` 中启用 ASan，使用 `-fsanitize=address` 编译器和链接器选项。在构建 nginx 时，可以通过将该选项添加到 `configure` 脚本的 `--with-cc-opt` 和 `--with-ld-opt` 参数中来实现。

Since most allocations in nginx are made from nginx internal [pool](https://nginx.org/en/docs/dev/development_guide.html#pool), enabling ASan may not always be enough to debug memory issues. The internal pool allocates a big chunk of memory from the system and cuts smaller allocations from it. However, this mechanism can be disabled by setting the `NGX_DEBUG_PALLOC` macro to `1`. In this case, allocations are passed directly to the system allocator giving it full control over the buffers boundaries.

​	由于 nginx 中的大多数分配都是从 nginx 内部的 [pool](https://nginx.org/en/docs/dev/development_guide.html#pool) 进行的，启用 ASan 可能并不总是足够以调试内存问题。内部池从系统中分配了一大块内存，并从中削减了较小的分配。然而，这种机制可以通过将 `NGX_DEBUG_PALLOC` 宏设置为 `1` 来禁用。在这种情况下，分配直接传递给系统分配器，使其对缓冲区边界具有完全控制权。

The following configuration line summarizes the information provided above. It is recommended while developing third-party modules and testing nginx on different platforms.

​	下面的配置行总结了上面提供的信息。在开发第三方模块并在不同平台上测试 nginx 时，建议使用此配置：

```
auto/configure --with-cc-opt='-fsanitize=address -DNGX_DEBUG_PALLOC=1'
               --with-ld-opt=-fsanitize=address
```



## 常见陷阱 Common Pitfalls



### 编写 C 模块 Writing a C module

The most common pitfall is an attempt to write a full-fledged C module when it can be avoided. In most cases your task can be accomplished by creating a proper configuration. If writing a module is inevitable, try to make it as small and simple as possible. For example, a module can only export some [variables](https://nginx.org/en/docs/dev/development_guide.html#http_variables).

​	最常见的陷阱是在可以避免的情况下尝试编写一个完整的 C 模块。在大多数情况下，您的任务可以通过创建适当的配置来完成。如果编写模块是不可避免的，尽量使它尽可能小而简单。例如，模块只能导出一些[变量](https://nginx.org/en/docs/dev/development_guide.html#http_variables)。

Before starting a module, consider the following questions:

​	在开始编写模块之前，请考虑以下问题：

- Is it possible to implement a desired feature using already [available modules](https://nginx.org/en/docs/index.html)?
- Is it possible to solve an issue using built-in scripting languages, such as [Perl](https://nginx.org/en/docs/http/ngx_http_perl_module.html) or [njs](https://nginx.org/en/docs/njs/index.html)?
- 是否可以使用已经[可用的模块](https://nginx.org/en/docs/index.html)来实现所需的功能？
- 是否可以使用内置的脚本语言来解决问题，例如 [Perl](https://nginx.org/en/docs/http/ngx_http_perl_module.html) 或 [njs](https://nginx.org/en/docs/njs/index.html)？





### C 字符串 C Strings

The most used string type in nginx, [ngx_str_t](https://nginx.org/en/docs/dev/development_guide.html#string_overview) is not a C-Style zero-terminated string. You cannot pass the data to standard C library functions such as `strlen()` or `strstr()`. Instead, nginx [counterparts](https://nginx.org/en/docs/dev/development_guide.html#string_overview) that accept either `ngx_str_t` should be used or pointer to data and a length. However, there is a case when `ngx_str_t` holds a pointer to a zero-terminated string: strings that come as a result of configuration file parsing are zero-terminated.

​	在 nginx 中，最常用的字符串类型是 [ngx_str_t](https://nginx.org/en/docs/dev/development_guide.html#string_overview)，它不是 C 风格的零终止字符串。不能将数据传递给标准的 C 库函数，如 `strlen()` 或 `strstr()`。而应该使用接受 `ngx_str_t` 或数据指针和长度的 nginx [对应函数](https://nginx.org/en/docs/dev/development_guide.html#string_overview)。但是，在某些情况下，`ngx_str_t` 可以保存指向零终止字符串的指针：解析配置文件产生的字符串是以零终止的。



### 全局变量 Global Variables

Avoid using global variables in your modules. Most likely this is an error to have a global variable. Any global data should be tied to a [configuration cycle](https://nginx.org/en/docs/dev/development_guide.html#cycle) and be allocated from the corresponding [memory pool](https://nginx.org/en/docs/dev/development_guide.html#pool). This allows nginx to perform graceful configuration reloads. An attempt to use global variables will likely break this feature, because it will be impossible to have two configurations at the same time and get rid of them. Sometimes global variables are required. In this case, special attention is needed to manage reconfiguration properly. Also, check if libraries used by your code have implicit global state that may be broken on reload.

​	在您的模块中避免使用全局变量。很可能这是一个错误，因为拥有全局变量是不合适的。任何全局数据都应与[配置周期](https://nginx.org/en/docs/dev/development_guide.html#cycle)关联，并从相应的[内存池](https://nginx.org/en/docs/dev/development_guide.html#pool)中分配。这使得 nginx 能够执行优雅的配置重载。尝试使用全局变量很可能会破坏此功能，因为不可能同时拥有两个配置并摆脱它们。有时确实需要全局变量。在这种情况下，需要特别注意以正确管理重新配置。另外，请检查您的代码使用的库是否具有可能在重新加载时破坏的隐式全局状态。

### 手动内存管理 Manual Memory Management

Instead of dealing with malloc/free approach which is error prone, learn how to use nginx [pools](https://nginx.org/en/docs/dev/development_guide.html#pool). A pool is created and tied to an object - [configuration](https://nginx.org/en/docs/dev/development_guide.html#http_conf), [cycle](https://nginx.org/en/docs/dev/development_guide.html#cycle), [connection](https://nginx.org/en/docs/dev/development_guide.html#connection), or [HTTP request](https://nginx.org/en/docs/dev/development_guide.html#http_request). When the object is destroyed, the associated pool is destroyed too. So when working with an object, it is possible to allocate the amount needed from the corresponding pool and don't care about freeing memory even in case of errors.

​	与其使用容易出错的 malloc/free 方法，不如学习如何使用 nginx 的[内存池](https://nginx.org/en/docs/dev/development_guide.html#pool)。池是与对象（[配置](https://nginx.org/en/docs/dev/development_guide.html#http_conf)、[周期](https://nginx.org/en/docs/dev/development_guide.html#cycle)、[连接](https://nginx.org/en/docs/dev/development_guide.html#connection)或[HTTP 请求](https://nginx.org/en/docs/dev/development_guide.html#http_request)）关联并创建的。当销毁对象时，关联的池也会被销毁。因此，在使用对象时，可以从相应的池中分配所需的数量，甚至在出现错误的情况下也不需要关心释放内存。



### 线程 Threads

It is recommended to avoid using threads in nginx because it will definitely break things: most nginx functions are not thread-safe. It is expected that a thread will be executing only system calls and thread-safe library functions. If you need to run some code that is not related to client request processing, the proper way is to schedule a timer in the `init_process` module handler and perform required actions in timer handler. Internally nginx makes use of [threads](https://nginx.org/en/docs/dev/development_guide.html#threads) to boost IO-related operations, but this is a special case with a lot of limitations.

​	建议避免在 nginx 中使用线程，因为它肯定会破坏事物：大多数 nginx 函数都不是线程安全的。预期线程仅会执行系统调用和线程安全库函数。如果需要运行与客户端请求处理无关的某些代码，则正确的方法是在 `init_process` 模块处理程序中调度一个计时器，并在计时器处理程序中执行所需的操作。在内部，nginx 使用[线程](https://nginx.org/en/docs/dev/development_guide.html#threads)来提升与 IO 相关的操作，但这是一种具有很多限制的特殊情况。

### 阻塞式库 Blocking Libraries

A common mistake is to use libraries that are blocking internally. Most libraries out there are synchronous and blocking by nature. In other words, they perform one operation at a time and waste time waiting for response from other peer. As a result, when a request is processed with such library, whole nginx worker is blocked, thus destroying performance. Use only libraries that provide asynchronous interface and don't block whole process.

​	常见的错误是使用内部是阻塞的库。大多数库都是同步且阻塞的。换句话说，它们一次只执行一个操作，并浪费时间等待其他对等方的响应。结果，当使用此类库处理请求时，整个 nginx 工作进程都被阻塞，从而破坏了性能。只使用提供异步接口且不会阻塞整个进程的库。

### 对外部服务的 HTTP 请求 HTTP Requests to External Services

Often modules need to perform an HTTP call to some external service. A common mistake is to use some external library, such as libcurl, to perform the HTTP request. It is absolutely unnecessary to bring a huge amount of external (probably [blocking](https://nginx.org/en/docs/dev/development_guide.html#using_libraries)!) code for the task which can be accomplished by nginx itself.

​	通常，模块需要对某些外部服务执行 HTTP 调用。常见的错误是使用一些外部库，例如 libcurl，来执行 HTTP 请求。为了完成 nginx 本身可以实现的任务，完全没有必要引入大量的外部（可能是[阻塞的](https://nginx.org/en/docs/dev/development_guide.html#using_libraries)）代码。

There are two basic usage scenarios when an external request is needed:

​	在需要外部请求的两种基本使用场景：

- in the context of processing a client request (for example, in content handler)
- in the context of a worker process (for example, timer handler)
- 在处理客户端请求的上下文中（例如，在内容处理程序中）
- 在工作进程的上下文中（例如，在计时器处理程序中）



In the first case, the best is to use [subrequests API](https://nginx.org/en/docs/dev/development_guide.html#http_subrequests). Instead of directly accessing external service, you declare a location in nginx configuration and direct your subrequest to this location. This location is not limited to [proxying](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) requests, but may contain other nginx directives. An example of such approach is the [auth_request](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request) directive implemented in [ngx_http_auth_request module](http://hg.nginx.org/nginx/file/tip/src/http/modules/ngx_http_auth_request_module.c).

​	在第一种情况下，最好使用[子请求 API](https://nginx.org/en/docs/dev/development_guide.html#http_subrequests)。而不是直接访问外部服务，您可以在 nginx 配置中声明一个位置，并将子请求定向到该位置。此位置不限于[代理](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)请求，还可以包含其他 nginx 指令。这种方法的一个示例是 [ngx_http_auth_request 模块](http://hg.nginx.org/nginx/file/tip/src/http/modules/ngx_http_auth_request_module.c) 中实现的 [auth_request](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request) 指令。

For the second case, it is possible to use basic HTTP client functionality available in nginx. For example, [OCSP module](http://hg.nginx.org/nginx/file/tip/src/event/ngx_event_openssl_stapling.c) implements simple HTTP client.

​	对于第二种情况，可以在 nginx 中使用基本的 HTTP 客户端功能。例如，[OCSP 模块](http://hg.nginx.org/nginx/file/tip/src/event/ngx_event_openssl_stapling.c) 实现了简单的 HTTP 客户端。