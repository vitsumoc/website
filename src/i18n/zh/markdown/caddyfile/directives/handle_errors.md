---
title: handle_errors (Caddyfile directive)
---

# handle_errors

配置错误处理器。

当普通的 HTTP 请求处理器返回错误时，普通的处理进程将会结束，随后调用错误处理器。Error handlers 提供了和普通处理器类似的路由，可以执行所有和普通处理器相同的行为。这为 HTTP 错误处理带来了很强的控制和定制能力。例如，您可以提供静态错误页面、使用模板化的错误页面，或是把请求反向代理到其他的错误处理后端。

此指令可以重复配置，配置不同的状态码参数，用来匹配不同的错误。如果没有指定错误状态码，则表示匹配所有类型的错误，这种表现方式就像是一种退路路由，如果其他错误处理器没有匹配，则都通过此处理器处理。

请求的内容会被带到错误路由中，因此在请求中设置的所有值例如 [site root](root) 或是 [vars](vars) 也都会被保存到错误路由中。此外， [新占位符](#placeholders) 也可以在错误处理中使用。

请注意，某些指令，例如 [`reverse_proxy`](reverse_proxy) 可能会写入一个 HTTP 状态码为错误类型的响应，但 _并不会_ 触发错误路由（只改写了状态码，没有实际触发错误）。

您可以根据您的路由需要，显示的使用 [`error`](error) 指令触发错误。

<h2 id="syntax">
	语法
</h2>

```caddy-d
handle_errors [<status_codes...>] {
	<directives...>
}
```

- **<status_codes...>** 是一个或多个用来匹配错误的 HTTP 状态码。状态码可以是 3 位数字，或是 `4xx`，`5xx` 这样的格式，这种格式表示 400-499 或是 500-599 范围的错误。如果没有选择状态码，则表示可以匹配所有错误，这将匹配所有其他错误匹配器都没有捕捉到的错误，表现形式类似于退路路由。

- **<directives...>** HTTP 处理 [指令](/docs/caddyfile/directives) 列表和 [匹配器](/docs/caddyfile/matchers)，每行一条。

<h2 id="placeholders">
	占位符
</h2>

错误处理中可以使用下述占位符，这些都是对完整占位符的 [Caddyfile 简写](/docs/caddyfile/concepts#placeholders)，完整的占位符可以在 [the JSON docs for an HTTP server's error routes](/docs/json/apps/http/servers/errors/#routes) 中查看。

| 占位符 | 描述 |
|---|---|
| `{err.status_code}` | 推荐的 HTTP 状态代码 |
| `{err.status_text}` | 与推荐状态代码关联的状态文本 |
| `{err.message}` | 错误信息 |
| `{err.trace}` | 错误起源 |
| `{err.id}` | 本次错误发生的 id |

<h2 id="examples">
	示例
</h2>

基于状态码指定错误页（例如对 `404` 错误则调用 `404.html` 页面）。请注意此处在 `handle_errors` 中运行 [`file_server`](file_server) 会保留错误的 HTTP 状态码（此处假设您已经在之前的站点配置中设置了 [site root](root)）：

```caddy-d
handle_errors {
	rewrite * /{err.status_code}.html
	file_server
}
```

使用 [`模板`](templates) 编写自定义错误消息的单个错误页面：

```caddy-d
handle_errors {
	rewrite * /error.html
	templates
	file_server
}
```

如果您只想为某些错误提供定制的错误页面，您可以使用 [`file`](/docs/caddyfile/matchers#file) 匹配器先检查错误页面是否存在。

```caddy-d
handle_errors {
	@custom_err file /err-{err.status_code}.html /err.html
	handle @custom_err {
		rewrite * {file_match.relative}
		file_server
	}
	respond "{err.status_code} {err.status_text}"
}
```

将错误反向代理到专业的错误处理服务器：

```caddy-d
handle_errors {
	rewrite * /{err.status_code}
	reverse_proxy https://http.cat {
		header_up Host {upstream_hostport}
		replace_status {err.status_code}
	}
}
```

用 [`respond`](respond) 返回错误码和名称

```caddy-d
handle_errors {
	respond "{err.status_code} {err.status_text}"
}
```

分别处理不同的错误码：

```caddy-d
handle_errors 404 410 {
	respond "It's a 404 or 410 error!"
}

handle_errors 5xx {
	respond "It's a 5xx error."
}

handle_errors {
	respond "It's another error"
}
```

和上述示例相同的功能，使用 [`表达式`](/docs/caddyfile/matchers#expression) 匹配状态码，使用 [`handle`](handle) 进行互斥分组：

```caddy-d
handle_errors {
	@404-410 `{err.status_code} in [404, 410]`
	handle @404-410 {
		respond "It's a 404 or 410 error!"
	}

	@5xx `{err.status_code} >= 500 && {err.status_code} < 600`
	handle @5xx {
		respond "It's a 5xx error."
	}

	handle {
		respond "It's another error"
	}
}
```
