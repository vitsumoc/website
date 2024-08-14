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

  Prefix with `-` to delete the field. The field may use prefix or suffix `*` wildcards to delete all matching fields.

  Prefix with `?` to set a default value for the field. The field is only written if it doesn't yet exist.

  Prefix with `>` to set the field, and enable `defer`, as a shortcut.

- **&lt;value&gt;** is the header field value, when adding or setting a field.

- **&lt;find&gt;** is the substring or regular expression to search for.

- **&lt;replace&gt;** is the replacement value; required if performing a search-and-replace.

- **defer** will force the header operations to be deferred until the response is being written out to the client. This is automatically enabled if any of the header fields are being deleted with `-`, when setting a default value with `?`, or when having used the `>` prefix.

For multiple header manipulations, you can open a block and specify one manipulation per line in the same way.

When using the `?` prefix to set a default header value, it is automatically separated into its own `header` handler, if it was in a `header` block with multiple header operations. [Under the hood](/docs/modules/http.handlers.headers#response/require), using `?` configures a response matcher which applies to the directive's entire handler, which only applies the header operations (like `defer`), but only if the field is not yet set.


## Examples

Set a custom header field on all requests:

```caddy-d
header Custom-Header "My value"
```

Strip the "Hidden" header field:

```caddy-d
header -Hidden
```

Replace `http://` with `https://` in any Location header:

```caddy-d
header Location http:// https://
```

Set security and privacy headers on all pages: (**WARNING:** only use if you understand the implications!)

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

Multiple header directives that are intended to be mutually-exclusive:

```caddy-d
route {
	header           Cache-Control max-age=3600
	header /static/* Cache-Control max-age=31536000
}
```

Set a default cache expiration if upstream doesn't define one:

```caddy-d
header ?Cache-Control "max-age=3600"
reverse_proxy upstream:443
```

To override the cache expiration that a proxy upstream had set for paths starting with `/no-cache`; enabling `defer` is necessary to ensure the header is set _after_ the proxy writes its headers:

```caddy-d
header /no-cache* >Cache-Control nocache
reverse_proxy upstream:443
```

To perform a deferred update of a `Set-Cookie` header to add `SameSite=None`; a regexp capture is used to grab the existing value, and `$1` re-inserts it at the start with the additional option appended:

```caddy-d
header >Set-Cookie (.*) "$1; SameSite=None;"
```
