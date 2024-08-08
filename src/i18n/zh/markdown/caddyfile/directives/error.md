---
title: error (Caddyfile directive)
---

# error

在 HTTP 处理程序链中触发错误，并带有可选消息和推荐的 HTTP 状态代码。

该处理不会进行响应，而是应该与 [`handle_errors`](handle_errors) 指令配对来调用您的自定义错误处理逻辑。

<h2 id="syntax">
	语法
</h2>

```caddy-d
error [<matcher>] <status>|<message> [<status>] {
    message <text>
}
```

- **&lt;status&gt;** HTTP 状态码，默认 `500`。
- **&lt;message&gt;** 错误消息，默认没有消息。
- **message** 另外一种写错误消息的方式，可以用来处理多行文本。

更清晰的描述是这样的，第一个非匹配器参数可以是 3 位数字的状态码，也可以是错误消息字符串，如果是错误消息字符串的话，那么后面的参数就应该是状态码。

<h2 id="examples">
	示例
</h2>

在某些请求路径上触发错误，并使用 [`handle_errors`](handle_errors) 写入响应：

```caddy
example.com {
	root * /srv

	# 在特定路径上触发错误
    error /private* "Unauthorized" 403
	error /hidden* "Not found" 404

    # 捕捉错误并返回 HTML 页面 
    handle_errors {
        rewrite * /{err.status_code}.html
		file_server
    }

	file_server
}
```
