---
title: file_server (Caddyfile directive)
---

<script>
window.$(function() {
	// Fix inline browse arg
	window.$('pre.chroma .s:contains("browse")').first()
		.wrapAll('<span class="k">').parent()
		.html('<a href="#browse" style="color: inherit;" title="browse">browse</a>')

	// We'll add links to all the subdirectives if a matching anchor tag is found on the page.
	addLinksToSubdirectives();
});
</script>

# file_server

支持真实或虚拟文件系统的静态文件服务器。通过将请求的 URI 路径附加到 [站点根路径](root) 来形成文件路径。


默认情况下，强制执行规范 URI；意味着当访问目录时会自动添加 `/` 结尾，访问文件时会自动删除 `/` 结尾。但是，如果内部重写（internal rewrite）修改了路径的最后一个元素（文件名），则不会进行重定向。

大部分情况下，和 `file_server` 共同出现的还会有 [`root`](root) 指令，用来指定整站根目录的位置。本指令也同样拥有一个名为 `root` 的子指令（见下方）用来单独指定本指令的根目录（不推荐）。请注意，站点根目录不提供沙箱保证：文件服务器确实会阻止路径组件的目录遍历，但根目录内的符号链接仍然可以允许根目录外部的访问。

当故障发生时（例如 文件不存在 `404`，权限拒绝 `403`），将会调用故障路由。使用 [`handle_errors`](handle_errors) 指令来定义故障路由，展示自定义故障页。

<h2 id="syntax">
	语法
</h2>

```caddy-d
file_server [<matcher>] [browse] {
	fs            <backend...>
	root          <path>
	hide          <files...>
	index         <filenames...>
	browse        [<template_file>] {
		reveal_symlinks
	}
	precompressed <formats...>
	status        <status>
	disable_canonical_uris
	pass_thru
}
```

- **fs** <span id="fs"/> 指定一个备用的（可能是虚拟的）文件系统。`caddy.fs` 命名空间中的任何模组都可以在此处使用。任何的根路径或者前缀都仍然可以用于备用文件系统模块。默认情况下使用本地磁盘。

	[`xcaddy`](/docs/build#xcaddy) v0.4.0 引入了 [`--embed`](https://github.com/caddyserver/xcaddy#custom-builds) 标志，可以用来在构建 Caddy 时嵌入一个文件系统树，并注册一个名为 `embedded` 的 `fs` 模块，让您可以通过可执行 Caddy 文件来分发您的静态站点。

- **root** <span id="root"/> 设置站点的根目录。用法上和 [`root`](root) 指令很相似，区域在于此处的子指令出现在文件服务器配置块中，并且会覆盖所有其他的站点根目录配置。默认值是 `{http.vars.root}` 或当前运行路径。注意：此子指令只会改变站点服务器指令的根目录，而不会改变同一站点下其他指令（例如 [`try_files`](try_files) 或是 [`templates`](templates)）的根目录的值，应使用 [`root`](root) 来改变整个站点根目录的值。

- **hide** <span id="hide"/> 设置隐藏文件和目录列表，当收到对隐藏内容的请求时，文件服务器会假装内容不存在。支持占位符和全局模板。注意此处配置的是 _文件系统_ 路径，而非请求路径。话句话说，是从当前运行路径下的相对路径，**而非**站点根目录的相对路径；所有的路径都会在匹配前被转换为绝对路径（如果有需要的话）。指定不带路径分隔符的文件名或模式将隐藏具有匹配名称的所有文件，无论其位置如何；否则，将尝试路径前缀匹配，然后进行全局匹配。由于这是 Caddyfile 配置，因此默认情况下就会把活动的配置文件加入其中。

- **index** <span id="index"/> 被视为 index 页面的文件列表，默认为：`index.html index.txt`。

- **browse** <span id="browse"/> 在没有 index 页面时提供文件列表。

  - **<template_file>** <span id="template_file"/> 选择文件列表的自定义模板页面。默认的模板页面可以使用 `caddy file-server export-template` 输出到 stdout。嵌入式的模板源码也可以 [在这里 ![external link](/old/resources/images/external-link.svg)](https://github.com/caddyserver/caddy/blob/master/modules/caddyhttp/fileserver/browse.html) 找到。浏览模板同样可以使用 [标准模板模组](/docs/modules/http.handlers.templates#docs) 中的各种 action。

  - **reveal_symlinks** <span id="reveal_symlinks"/> 允许在列表中显示符号链接的目标，默认情况下符号链接的目标是隐藏的，仅显示链接文件本身。

- **precompressed** <span id="precompressed"/> 用来搜索预压缩 side car 文件的编码格式列表。参数是搜索 [side car 文件](https://en.wikipedia.org/wiki/Sidecar_file) 编码格式的有序列表。支持的格式包括 `gzip`（`.gz`），`zstd` （`.zst`）和 `br`（`.br`）。

	所有文件查找都会首先查找未压缩文件是否存在。一旦找到，Caddy 将查找带有每种启用格式的文件扩展名的 sidecar 文件。如果找到预压缩的 sidecar 文件，Caddy 将使用预压缩文件进行响应，并适当设置 `Content-Encoding` 响应头。否则，Caddy 将正常响应未压缩的文件。如果启用了 [`encode` 指令](encode)，那么则可能会在没有预压缩的情况下动态压缩文件。

- **status** <span id="status"/> 可选的在写入响应时使用的状态码。和 [自定义错误页面](handle_errors) 搭配使用效果较好。格式是三位数字例如：`404`，支持占位符。默认情况下的状态码通常为 `200`，部分内容是 `206`。

- **disable_canonical_uris** <span id="disable_canonical_uris"/> 关闭默认重定向（如果请求路径是目录，则添加尾部斜杠；如果请求路径是文件，则删除尾部斜杠）。请注意，默认情况下，如果请求路径的最后一个元素（文件名）经过内部重写，则不会发生规范化，以避免用隐式行为破坏显式重写。

- **pass_thru** <span id="pass_thru"/> 启用直通模式，如果未找到请求的文件，该模式将继续到路由中的下一个 HTTP 处理程序，而不是触发 `404` 错误（调用 [`handle_errors`](handle_errors) 路由）。实际上，这仅在 `file_server` 之后具有其他处理程序指令的 [`route`](route) 块内部有用，因为文件服务器指令实际上 [排在指令的末尾](/docs/caddyfile/directives#directive-order)。

<h2 id="examples">
	示例
</h2>

当前目录之外的静态文件服务器：

```caddy-d
file_server
```

启用文件列表：

```caddy-d
file_server browse
```

仅提供 /static 文件夹中的静态文件：

```caddy-d
file_server /static/*
```

`file_server` 指令通常与 [root](root) 指令配对以设置文件服务的根目录：

```caddy
example.com {
	root * /srv
	file_server
}
```

<aside class="tip">

如果您将 Caddy 作为 systemd 服务运行，则从 `/home` 读取文件将不起作用，因为 `caddy` 用户没有 `/home` 目录的“可执行”权限（遍历所必需的）。建议您将文件放在 `/srv` 或 `/var/www/html` 中。

</aside>

隐藏所有 `.git` 文件夹及其内容：

```caddy-d
file_server {
	hide .git
}
```

如果客户端支持（通过 `Accept-Encoding` 头），则检查请求的文件旁边是否存在预压缩文件。因此，如果请求 `/path/to/file`，它会按顺序检查 `/path/to/file.zst`、`/path/to/file.br` 和 `/path/to/file.gz` 并提供第一个可用文件和设置相应的 `Content-Encoding`：

```caddy-d
file_server {
	precompressed zstd br gzip
}
```
