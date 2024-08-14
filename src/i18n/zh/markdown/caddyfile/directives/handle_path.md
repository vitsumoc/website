---
title: handle_path (Caddyfile directive)
---

<script>
window.$(function() {
	// Add a link to [<path_matcher>] as a special case for this directive.
	// The matcher text includes <> characters which are parsed as HTML,
	// so we must use text() to change the link text.
	window.$('pre.chroma .s:contains("<path_matcher>")')
		.map(function(k, item) {
			let text = item.innerText.replace(/</g, '&lt;').replace(/>/g, '&gt;');
			window.$(item)
				.html('<a href="/docs/caddyfile/matchers#path-matchers" style="color: inherit;" title="Matcher token">' + text + '</a>')
				.removeClass('s')
				.addClass('nd');
		});
});
</script>

# handle_path

与 [`handle`](handle) 指令相同，但会显示的删除 [`uri strip_prefix`](uri) 匹配的路径前缀。

处理与特定路径匹配的请求（同时从请求 URI 中剥离该路径）是一个足够常见的用例，为了方便起见，他有自己的指令。

<h2 id="syntax">
	语法
</h2>

```caddy-d
handle_path <path_matcher> {
	<directives...>
}
```

- **<directives...>** Http 处理指令列表，每行一个，使用方式和在 `handle_path` 块外面时相同。

必须配置唯一的一个 [path matcher](/docs/caddyfile/matchers#path-matchers)，您不能在 `handle_path` 中使用命名匹配器。

<h2 id="examples">
	示例
</h2>

此配置：

```caddy-d
handle_path /prefix/* {
	...
}
```

👆 和下方配置等效 👇，但上方的 `handle_path` 👆 更加简洁

```caddy-d
handle /prefix/* {
	uri strip_prefix /prefix
	...
}
```

完整的 Caddyfile 示例，其中的 `handle_path` 和 `handle` 不会同时进入；但需要小心 [subfolder problem <img src="/old/resources/images/external-link.svg" class="external-link">](https://caddy.community/t/the-subfolder-problem-or-why-cant-i-reverse-proxy-my-app-into-a-subfolder/8575)

```caddy
example.com {
	# API 服务，去除 /api 前缀
	handle_path /api/* {
		reverse_proxy localhost:9000
	}

	# 静态站点服务
	handle {
		root * /srv
		file_server
	}
}
```
