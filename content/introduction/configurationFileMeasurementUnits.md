+++
title = "配置文件的度量单位"
date = 2023-08-14T16:52:29+08:00
weight = 100
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# Configuration file measurement units - 配置文件的度量单位

https://nginx.org/en/docs/syntax.html

Sizes can be specified in bytes, kilobytes (suffixes `k` and `K`) or megabytes (suffixes `m` and `M`), for example, “`1024`”, “`8k`”, “`1m`”.

​	大小可以用字节、千字节（后缀 `k` 和 `K`）或兆字节（后缀 `m` 和 `M`）来指定，例如，“`1024`”、“`8k`”、“`1m`”。

Offsets may be also specified in gigabytes using `g` or `G` suffixes.

​	偏移量也可以用 `g` 或 `G` 后缀来指定为千兆字节。

Time intervals can be specified in milliseconds, seconds, minutes, hours, days and so on, using the following suffixes:

​	时间间隔可以用毫秒、秒、分钟、小时、天等来指定，使用以下后缀：

| ms   | milliseconds    |
| ---- | --------------- |
| s    | seconds         |
| m    | minutes         |
| h    | hours           |
| d    | days            |
| w    | weeks           |
| M    | months, 30 days |
| y    | years, 365 days |



Multiple units can be combined in a single value by specifying them in the order from the most to the least significant, and optionally separated by whitespace. For example, “`1h 30m`” specifies the same time as “`90m`” or “`5400s`”. A value without a suffix means seconds. It is recommended to always specify a suffix.

​	多个单位可以通过从最高到最低的顺序指定在单个值中，可以选择用空格分隔。例如，“`1h 30m`”指定的时间与“`90m`”或“`5400s`”相同。没有后缀的值表示秒。建议始终指定后缀。

Some of the time intervals can be specified only with a seconds resolution.

​	某些时间间隔只能以秒为分辨率进行指定。