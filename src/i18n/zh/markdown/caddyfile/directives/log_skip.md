---
title: log_skip (Caddyfile directive)
---

# log_skip

跳过匹配到请求的接入日志。

此指令应与 [`log`](log) 指令一起使用，以跳过与您的需求不相关的日志记录请求。

在 v2.8.0 之前，此指令被命名为 `skip_log`，但为了与其他指令保持一致，已重命名。

<h2 id="syntax">
	语法
</h2>

```caddy-d
log_skip [<matcher>]
```

<h2 id="examples">
	示例
</h2>

跳过存储在子路径中的静态文件的接入日志：

```caddy
example.com {
	root * /srv

	log
	log_skip /static*

	file_server
}
```

跳过与模式匹配的请求的接入日志；此案例中，匹配具有特定扩展名的文件：

```caddy-d
@skip path_regexp \.(js|css|png|jpe?g|gif|ico|woff|otf|ttf|eot|svg|txt|pdf|docx?|xlsx?)$
log_skip @skip
```

如果指令在路由块中，且已经进行了匹配，则不需要再在指令中使用匹配器。例如，本例匹配某个特定子路径下的静态文件服务器：

```caddy-d
handle_path /static* {
	root * /srv/static
	log_skip
	file_server
}
```
