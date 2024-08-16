---
title: encode (Caddyfile directive)
---

<script>
window.$(function() {
	// We'll add links to all the subdirectives if a matching anchor tag is found on the page.
	addLinksToSubdirectives();
});
</script>

# encode

使用配置的方式编码响应，典型的用法是压缩响应。

<h2 id="syntax">
	语法
</h2>

```caddy-d
encode [<matcher>] <formats...> {
	# 编码格式
	gzip [<level>]
	zstd [<level>]
	
	minimum_length <length>

	# 单行编码匹配器
	match [header <field> [<value>]] | [status <code...>]
	# 或多种匹配条件的匹配块
	match {
		status <code...>
		header <field> [<value>]
	}
}
```

- **&lt;formats...&gt;** 启用的编码格式列表。如果同时启用了多种编码格式，则基于请求中的 Accept-Encoding 头内容来选用编码格式；如果客户端没有偏好（q-factor），则使用第一个支持的编码方式。

- **gzip** <span id="gzip"/> 启用 Gzip 压缩，可选压缩等级。

- **zstd** <span id="zstd"/> 启用 Zstandard 压缩，可选压缩等级（可选值为：default，fastest，better，best）。默认压缩级别大致等同于默认的 Zstandard 模式（level 3）。

- **minimum_length** <span id="minimum_length"/> 需要压缩的响应的最小长度（默认值：512）。

- **match** <span id="match"/> 是一个 [响应匹配器](#response-matcher)。只有被匹配的响应才会被编码，匹配器的默认值如下：

  ```caddy-d
  match {
  	header Content-Type application/atom+xml*
  	header Content-Type application/eot*
  	header Content-Type application/font*
  	header Content-Type application/geo+json*
  	header Content-Type application/graphql+json*
  	header Content-Type application/javascript*
  	header Content-Type application/json*
  	header Content-Type application/ld+json*
  	header Content-Type application/manifest+json*
  	header Content-Type application/opentype*
  	header Content-Type application/otf*
  	header Content-Type application/rss+xml*
  	header Content-Type application/truetype*
  	header Content-Type application/ttf*
  	header Content-Type application/vnd.api+json*
  	header Content-Type application/vnd.ms-fontobject*
  	header Content-Type application/wasm*
  	header Content-Type application/x-httpd-cgi*
  	header Content-Type application/x-javascript*
  	header Content-Type application/x-opentype*
  	header Content-Type application/x-otf*
  	header Content-Type application/x-perl*
  	header Content-Type application/x-protobuf*
  	header Content-Type application/x-ttf*
  	header Content-Type application/xhtml+xml*
  	header Content-Type application/xml*
  	header Content-Type font/*
  	header Content-Type image/svg+xml*
  	header Content-Type image/vnd.microsoft.icon*
  	header Content-Type image/x-icon*
  	header Content-Type multipart/bag*
  	header Content-Type multipart/mixed*
  	header Content-Type text/*
  }
  ```

<h2 id="response-matcher">
	响应匹配器
</h2>

**响应匹配器** 可以按照一定的标准来过滤（或分类）响应。

### status

```caddy-d
status <code...>
```

按照 HTTP 状态码匹配。

- **&lt;code...&gt;** HTTP 状态码列表。支持 `2xx`，`3xx`... 这样的格式，可以用来进行 200-299，300-399...这样的范围匹配。

### header

参考请求匹配器中的 [header](/docs/caddyfile/matchers#header) 语法。

<h2 id="examples">
	示例
</h2>

启用 Gzip 压缩：

```caddy-d
encode gzip
```

启用 Zstandard 和 Gzip 压缩（Zstandard 优先，因为他配置在前面）：

```caddy-d
encode zstd gzip
```

对 [`file_server`](file_server) 启动的文件站点全站启用压缩：

```caddy
example.com {
	root * /srv
	encode zstd gzip
	file_server
}
```
