---
title: header (Caddyfile directive)
---

# header

操作 HTTP 响应头，可以进行设置、添加或删除，或使用正则表达式替换。

默认情况下，和头相关的操作会立刻执行；但如果有删除（`-` 前缀）或是设置默认值（`?` 前缀），则和头相关的操作会自动延迟（deffer）到需要写入到客户端的时刻执行。

如果需要操作 HTTP 请求头，您可以使用 [`request_header`](request_header) 指令。

<h2 id="syntax">
	语法
</h2>

```caddy-d
header [<matcher>] [[+|-|?|>]<field> [<value>|<find>] [<replace>]] {
	# 添加
	+<field> <value>

	# 设置
	<field> <value>

	# 设置并 defer
	><field> <value>

	# 删除
	-<field>

	# 替换
	<field> <find> <replace>

	# 替换并 defer
	><field> <find> <replace>

	# 默认
	?<field> <value>

	[defer]
}
```

- **&lt;field&gt;** 是头的字段名称。

	不使用前缀时，表示设置（覆盖）该字段。

	使用 `+` 前缀表示在字段存在时，添加而非覆盖（设置）该字段；响应头中的同名字段可以重复出现多次。

	使用 `-` 前缀表示删除该字段，可以使用前缀或 `*` 通配符删除所有匹配的字段。

	使用 `?` 前缀表示为字段设置默认值，只有当字段本身不存在时才会进行设置。

	使用 `>` 前缀表示设置字段，并启用 `defer`，这是一个快捷方式。

- **&lt;value&gt;**  当添加或设置字段是，表示字段的值。

- **&lt;find&gt;** 用于搜索的子字符串或正则表达式。

- **&lt;replace&gt;** 在进行搜索和替换操作时使用的替换值。

- **defer** 会强制将头操作延迟到向客户端写入响应时执行。如果有任何字段被删除 `-`、设置默认值 `?` 或是带有 `>` 前缀，则此子指令默认被添加。

对于多个头部操作，您可以打开一个块，在其中逐行配置需要的操作。

当使用 `?` 前缀设置头的默认值时，如果他和其他的头部操作共同处在同一个 `header` 块中，那么他实际上会转入自己独立的 `header` 处理过程。[Under the hood](/docs/modules/http.handlers.headers#response/require)，使用 `?` 配置的响应匹配器会应用到一整个指令处理过程。这只会在匹配的字段不存在时生效，也只会应用到和头部相关的操作（例如 `defer`）。

<h2 id="examples">
	示例
</h2>

对所有的响应都设置自定义头：

```caddy-d
header Custom-Header "My value"
```

删除 "Hidden" 头：

```caddy-d
header -Hidden
```

在 Location 头中将 `http://` 替换为 `https://`：

```caddy-d
header Location http:// https://
```

对所有页面设置安全和隐私头：（**警告：** 您必须理解这些配置造成的影响！）

```caddy-d
header {
	# disable FLoC tracking
	Permissions-Policy interest-cohort=()

	# enable HSTS
	Strict-Transport-Security max-age=31536000;

	# disable clients from sniffing the media type
	X-Content-Type-Options nosniff

	# clickjacking protection
	X-Frame-Options DENY
}
```

被配置为互斥的两个 header 指令：

```caddy-d
route {
	header           Cache-Control max-age=3600
	header /static/* Cache-Control max-age=31536000
}
```

如果上游没有定义，则添加一个默认的缓存超时时间：

```caddy-d
header ?Cache-Control "max-age=3600"
reverse_proxy upstream:443
```

即使上游已经进行了设置，仍然覆盖 `/no-cache` 路径下的缓存过期配置；启用 `defer` 用来确保响应头的覆盖操作是在上游写入头部 _之后_ 进行：

```caddy-d
header /no-cache* >Cache-Control nocache
reverse_proxy upstream:443
```

执行一个 deffered 的设置，在 `Set-Cookie` 头的尾部添加 `SameSite=None`；使用正则表达式抓取现有的值，随后的 `$1` 重新插入值，并添加我们需要的内容：

```caddy-d
header >Set-Cookie (.*) "$1; SameSite=None;"
```
