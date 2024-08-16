---
title: log_append (Caddyfile directive)
---

# log_append

给当前请求的接入日志尾部追加字段。

此指令应与 [`log`](log) 指令一起使用，首先使用 log 指令开启接入日志。

追加内容的值可以是静态字符串，也可以是 [占位符](/docs/caddyfile/concepts#placeholders)，占位符会在请求时被替换为对应的值。

<h2 id="syntax">
	语法
</h2>

```caddy-d
log_append [<matcher>] <key> <value>
```

<h2 id="examples">
	示例
</h2>

在日志中显示正在处理请求的 area（`static` 或 `dynamic`）：

```caddy
example.com {
	log

	handle /static* {
		log_append area "static"
		respond "Static response!"
	}

	handle {
		log_append area "dynamic"
		reverse_proxy localhost:9000
	}
}
```
