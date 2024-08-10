---
title: handle (Caddyfile directive)
---

# handle

一组与其他同级 `handle` 块互斥的指令。

换句话说，当有多个 `handle` 指令顺序出现时，只有第一个 _匹配到_ 的 `handle` 块会执行。一个没有匹配器的 handle 可以被视为 _退路_ 路由（最后的选择，可以匹配一切）。

`handle` 指令按照他们匹配器的 [指令排序算法](/docs/caddyfile/directives#sorting-algorithm) 顺序执行。[`handle_path`](handle_path) 是与 `handle` 有共同排序优先级的一种特殊指令，只是对路径的配置不同。

如需要，Handle 块可以嵌套使用，但是 handle 块中只能使用 HTTP 处理指令。

<h2 id="syntax">
	语法
</h2>

```caddy-d
handle [<matcher>] {
	<directives...>
}
```

- **<directives...>** Http 处理指令列表，每行一个，使用方式和在 handle 块外面时相同。

<h2 id="similar-directives">
	相似指令
</h2>

还有一些其他可以包裹 HTTP 处理指令的指令，根据您需要的表现方式来选择：

- [`handle_path`](handle_path) 和 `handle` 相似，但会在处理前删除请求前缀。

- [`handle_errors`](handle_errors) 和 `handle` 相似，但只会在 Caddy 处理请求报错时调用。

- [`route`](route) 和 `handle` 一样，可以包裹其他指令，但是有两个区别：
  1. route 指令不会排斥其他 route，
  2. route 中的指令不会被 [重新排序](/docs/caddyfile/directives#directive-order)，这使得您拥有更多的控制能力。

<h2 id="examples">
	示例
</h2>

对 `/foo/` 前缀的请求使用静态文件服务器响应，其他请求则使用反向代理：

```caddy
example.com {
	handle /foo/* {
		file_server
	}

	handle {
		reverse_proxy 127.0.0.1:8080
	}
}
```

您可以在同一个站点下混用 `handle` 和 [`handle_path`](handle_path)，他们依然会保持互斥（匹配某一条后则不会进入其他）：

```caddy
example.com {
	handle_path /foo/* {
		# 此处会剥去 "/foo" 前缀
	}

	handle /bar/* {
		# 此处将保留 "/bar"
	}
}
```

您可以嵌套 `handle` 块，用来满足一些复杂的路由逻辑：

```caddy
example.com {
	handle /foo* {
		handle /foo/bar* {
			# 此块只匹配 /foo/bar 下的内容
		}

		handle {
			# 此块匹配 /foo/ 中的剩余内容
		}
	}

	handle {
		# 此块匹配所有其余内容（退路）
	}
}
```
