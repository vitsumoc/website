---
title: invoke (Caddyfile directive)
---

# invoke

<i>⚠️ 实验性</i>

调用 [具名路由](/docs/caddyfile/concepts#named-routes)。

在一些 HTTP 处理过程指令都拥有自己的内存状态，或是他们的加载成本很高的情况下，此指令非常有效。例如您拥有超过一百个站点，此此时调用具名路由将会减少内存开销。

<aside class="tip">
	
与 [`import`](/docs/caddyfile/directives/import) 不同，`invoke` 不支持参数，但您仍然可以使用 [`vars`](/docs/caddyfile/directives/vars) 定义具名路由中使用的变量。

</aside>

<h2 id="syntax">
	语法
</h2>

```caddy-d
invoke [<matcher>] <route-name>
```

- **&lt;route-name&gt;** 先前定义的，需要被调用的路由。如果路由未找到到，则会报错。

<h2 id="examples">
	示例
</h2>

定义一个带有 [`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy) 的 [具名路由](/docs/caddyfile/concepts#named-routes)，可以在多个站点间复用，在每个站点间复用同一个内存中的负载均衡状态。

```caddy
&(app-proxy) {
	reverse_proxy app-01:8080 app-02:8080 app-03:8080 {
		lb_policy least_conn
		health_uri /healthz
		health_interval 5s
	}
}

# 可以通过 /app 子路径访问应用
# 其他情况则访问主站
example.com {
	handle_path /app* {
		invoke app-proxy
	}

	handle {
		root * /srv
		file_server
	}
}

# 使用子域名直接访问应用
app.example.com {
	invoke app-proxy
}
```
