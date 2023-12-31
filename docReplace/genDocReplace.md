+++
title = "生成文档前的替换"
date = 2023-08-15T08:37:31+08:00
description = ""
isCJKLanguage = true
draft = false

+++

# 生成文档前的替换

## 替换生成指令的标题

```
// 查找匹配如下正则表达式
\| Syntax:  \| `([\S]+)

// 替换成
### $1

| Syntax:  | `$1
```



## 替换掉代码块外的边框

```
// 查找匹配如下正则表达式
^>(\s){1}

// 替换成空

// 查找匹配字符串(不包含单引号)
'  > '
  
// 替换成(不包含单引号)
'  '
```



## 替换表格渲染出问题的地方

```
// 1. 查找匹配如下正则表达式
\| \:-{1,} \| -{1,} \|\n(\| Default\: \|)

// 替换成
$1

// 2. 查找匹配如下正则表达式
\| (Syntax:  )\| (`[^\n]+`)\s+\|

// 替换成
	$1$2\n
	
// 3. 查找匹配如下正则表达式
\| (Default: )\|\s+(`[^\n]+`)\s+\|

或
\| (Default: )\|\s+(\—{1,})\s+\|

//替换成
	$1$2\n

// 4.查找匹配如下正则表达式
\| (Context: )\| (`[^\n]+`)\s+\|

//替换成
	$1$2\n

// 5. 查找匹配如下正则表达式
(#{1,}\s+[a-zA-Z\-]+);`

或
(###\s{1,}[^;`]{1,});`

// 替换成
$1


```

## 替换modules reference 中的模块链接

```
// 查找匹配如下正则表达式
\[(ngx_[a-z\_]+_module)\]\([^\)]+\)

// 替换成
[$1](../$1)
```



## 替换非modules reference 中的模块链接

```
// 查找匹配如下正则表达式
\[(ngx_[a-z\_]+_module)\]\([^\)]+\)

// 替换成
[$1]({{< ref "/mod_ref/$1" >}})
```



## 替换指令链接

```
//1. 查找匹配如下正则表达式
\(https:\/\/nginx\.org\/en\/docs\/(http\/)?(ngx_[a-z\_]+_module)\.html(#[a-z\_]+)\)

// 替换成
({{< ref "/mod_ref/$2$3">}})
```



## 替换其他一些链接

```
// 查找匹配如下正则表达式
\(https:\/\/nginx\.org\/en\/docs\/events.html(#[a-z\_]+)\)

// 替换成
({{< ref "/introduction/connectionProcessingMethods$1">}})

// 查找匹配如下字符串
\(https:\/\/nginx\.org\/en\/docs\/http\/server_names.html(#[a-z\_]+)\)

// 替换成
({{< ref "/introduction/serverNames$1">}})

// 查找匹配如下字符串
\(https:\/\/nginx\.org\/en\/docs\/http\/server_names.html\)

// 替换成
({{< ref "/introduction/serverNames">}})

// 查找匹配如下字符串
\(https:\/\/nginx\.org\/en\/docs\/debugging_log\.html\)

// 替换成
({{< ref "/introduction/aDebuggingLog">}})

// 查找匹配如下字符串
\(https:\/\/nginx\.org\/en\/docs\/http\/request_processing\.html\)

// 替换成
({{< ref "/introduction/howNginxProcessesARequest">}})
```

