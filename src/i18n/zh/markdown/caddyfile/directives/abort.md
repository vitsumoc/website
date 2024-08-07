---
title: abort (Caddyfile directive)
---

# abort

通过立即中止 HTTP 处理程序链并关闭连接来阻止对客户端的任何响应，同一连接上的任何并发活动 HTTP 流都会被中断。

<h2 id="syntax">
	格式
</h2>

```caddy-d
abort [<matcher>]
```

<h2 id="examples">
	示例
</h2>

使用通配符强制关闭所有发起未知路径请求的连接：

```caddy
*.example.com {
    @foo host foo.example.com
    handle @foo {
        respond "This is foo!" 200
    }

    handle {
		# 未匹配到路径的请求都会发往这里
		# 不处理这些请求并断开连接
        abort
    }
}
```
