---
title: bind (Caddyfile directive)
---

# bind

覆盖服务器 socket 应绑定的接口。

通常来说，监听器绑定在空（通配符）接口。然而，您也可以指定监听绑定到某个主机名或 IP 地址。本指令只接收主机名，不接收端口，端口由 [站点地址](/docs/caddyfile/concepts#addresses) 决定（默认值为 `443`）。

请注意，将站点绑定在不同的接口上可能会导致意料之外的后果。例如，有两个都期望向 `127.0.0.1` 服务并具有相同端口的站点，其中一个配置了 `bind 127.0.0.1`，另一个则会无法访问，因为另一个站点将绑定到没有特定主机名的端口。操作系统将根据情况选择一个更加具体的接口。（虚拟主机名并不会在不同的监听器之间共享。）

`bind` 接受 [网络地址](/docs/conventions#network-addresses) 参数，但不可携带端口。

<h2 id="syntax">
	语法
</h2>

```caddy-d
bind <hosts...>
```

- **&lt;hosts...&gt;** 要绑定的主机接口列表，用于绑定侦听器。

<h2 id="examples">
	示例
</h2>

使 socket 只能在当前计算机上访问，绑定到环回接口（localhost）：

```caddy
example.com {
	bind 127.0.0.1
}
```

包括 IPv6：

```caddy
example.com {
	bind 127.0.0.1 [::1]
}
```

绑定到 `10.0.0.1:8080`：

```caddy
example.com:8080 {
	bind 10.0.0.1
}
```

绑定到 Unix domain socket `/run/caddy`：

```caddy
example.com {
	bind unix//run/caddy
}
```

修改文件权限，使所有人都可以写入（[默认](/docs/conventions#network-addresses) 情况下是 `0200`，意味着只有文件持有者可以写入）：

```caddy
example.com {
	bind unix//run/caddy|0222
}
```

将一个域名绑定到两个接口，并使用不同的响应：

```caddy
example.com {
	bind 10.0.0.1
	respond "One"
}

example.com {
	bind 10.0.0.2
	respond "Two"
}
```
