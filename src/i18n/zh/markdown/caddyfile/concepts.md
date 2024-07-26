---
title: Caddyfile 概念
---

<h1 id="caddyfile-concepts">
	Caddyfile 概念
</h1>

此文档将会帮助您详细学习 HTTP Caddyfile。

1. [结构(structure)](#structure)
	- [块(blocks)](#blocks)
	- [指令(directives)](#directives)
	- [词元和引号(tokens and quotes)](#tokens-and-quotes)
2. [全局选项(global options)](#global-options)
3. [地址(addresses)](#addresses)
4. [匹配器(matchers)](#matchers)
5. [占位符(placeholders)](#placeholders)
6. [配置片段(snippets)](#snippets)
7. [命名路由(named routes)](#named-routes)
8. [注释(comments)](#comments)
9. [环境变量(environment variables)](#environment-variables)

<h2 id="structure">
	结构(structure)
</h2>

Caddyfile 的结构可以直观的描述：

![Caddyfile structure](/i18n/zh/old/resources/images/caddyfile-visual.png)

关键点：

- [**全局选项**](#global-options) 应该出现在配置顶端。

- [配置片段](#snippets) 或 [命名路由](#named-routes) 应该尽量集中在一起。 

- 除此之外， Caddyfile 的首行 **通常** 是 [站点地址](#addresses)。

- 所有的 [指令](#directives) 和 [匹配器](#matchers) 都 **必须** 在站点块中。Caddyfile 不支持对这两项的全局配置或者向站点的继承。

- 如果仅有一个站点，那么站点块的 `{ }` 可以省略。

Caddyfile 必须包含一个或多个站点，站点必须由一个或多个 [地址](#addresses) 作为定义的开始。所有在地址前出现的指令都会让解析器报错。

<h3 id="blocks">
	块(blocks)
</h3>

使用大括号标记 **块** 的打开和关闭：

```
... {
	...
}
```

- 左大括号 `{` 必须位于行尾，而且和前文至少有一个空格（如果有前文）。

- 右大括号 `}` 必须位于独立的一行。

当只有一个站点时，大括号（和缩进）是可选的。这个特性用来定义一些非常简单（但实用）的站点，比如：

```caddy
localhost

reverse_proxy /api/* localhost:9001
file_server
```

这等同于：

```caddy
localhost {
	reverse_proxy /api/* localhost:9001
	file_server
}
```

当您只需要配置一个站点块时，两种方式皆可，随您喜好。

而当需要在一个 Caddyfile 中配置多个站点块时，您 **必须** 使用大括号包裹每一个站点，从而区分他们的配置内容：

```caddy
example1.com {
	root * /www/example.com
	file_server
}

example2.com {
	reverse_proxy localhost:9000
}
```

如果一个请求能够匹配到多个站点块地址，那么具有最具体地址的站点块会被执行，一个请求不会同时被多个站点块执行。

<h3 id="directives">
	指令(directives)
</h3>

[**指令**](/docs/zh/caddyfile/directives) 描述了站点的功能，**必须**被放置在站点块中。例如，文件服务器应该这样：

```caddy
localhost {
	file_server
}
```

而反向代理服务器：

```caddy
localhost {
	reverse_proxy localhost:9000
}
```

在上面的例子中，[`file_server`](/docs/caddyfile/directives/file_server) 和 [`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy) 都是指令。指令在站点块中总是位于行首。

在第二个例子中，`localhost:9000` 是一个 **实际参数**，因为他出现在指令之后。

有些情况下指令可以有属于自己的块。**子指令** 在指令块中总是位于行首：

```caddy
localhost {
	reverse_proxy localhost:9000 localhost:9001 {
		lb_policy first
	}
}
```

此处的 `lb_policy` 是 [`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy) 的子指令（用来设置多个后端间的负载均衡策略）。

**除非另有说明，否则指令无法引用在其他指令块中。**例如：[`basic_auth`](/docs/caddyfile/directives/basic_auth) 不能用在 [`file_server`](/docs/caddyfile/directives/file_server) 中，因为文件服务器并不了解如何进行认证；但是您可以在 [`route`](/docs/caddyfile/directives/route)、[`handle`](/docs/caddyfile/directives/handle) 和 [`handle_path`](/docs/caddyfile/directives/handle_path) 块中使用指令，因为他们的设计目的就是用于将指令聚合在一起。

请注意，在适配 HTTP Caddyfile 时，HTTP 句柄指令按照一个特定的 [指令顺序](/docs/caddyfile/directives#directive-order) 存储，除非是在 [`route`](/docs/caddyfile/directives/route) 块中。因此，除了 `route` 块的情况外，其他情况下指令的书写顺序和实际程序运行是无关的。

<h3 id="tokens-and-quotes">
	词元和引号
</h3>

Caddyfile 在被解析前需要首先被拆分成词元，空格在 Caddyfile 中的重要性也是源于此，因为词元由空格分割。

通常情况下，指令需要固定数量的实参，如果一个实参中使用了空格，那么他会被视为两个独立的词元：

```caddy-d
directive abc def
```

这会导致一些问题，返回错误，也可能产生不受预期的行为。

如果实参的内容确实应该是 `abc def`，那么可以使用引号包裹：

```caddy-d
directive "abc def"
```

在被引号包裹的词元中，可以使用转义表示引号：

```caddy-d
directive "\"abc def\""
```

如果不想使用引号转义，您也可以使用反引号 <code>\` \`</code> 包裹词元，例如：

```caddy-d
directive `{"foo": "bar"}`
```

在引号词元中，所有的其他字符都会被视为字面量，包括空格、制表符、换行符等。这样的多行词元也是可行的：

```caddy-d
directive "first line
	second line"
```

还支持 Heredocs <span id="heredocs"/>：

```caddy
example.com {
	respond <<HTML
		<html>
		  <head><title>Foo</title></head>
		  <body>Foo</body>
		</html>
		HTML 200
}
```

heredoc 的起始标记是 `<<`，随后跟随任何文本（推荐使用大写字母），终止标记是和开放标记相同的文本（在上述的例子中，是 `HTML`）。如需使用 `<<` 字面量（而非开启 heredoc）可以使用 `\<<` 转义。

终止标记可以缩进，这会导致内容中的每一行都会被删除同样数量的缩进（受 [PHP](https://www.php.net/manual/en/language.types.string.php#language.types.string.syntax.heredoc) 启发），这对于块内的可读性有好处，也方便了控制内容中的缩进。内容结尾处的换行符会被删除，但如果需要的话可以在内容中添加一个新行来解决（在终止标记之前）。

终止标记后跟随的词元视为指令的实参（在上文的例子中，实参是返回状态码 200）。

## Global options

A Caddyfile may optionally start with a special block that has no keys, called a [global options block](/docs/caddyfile/options):

```caddy
{
	...
}
```

If present, it must be the very first block in the config.

It is used to set options that apply globally, or not to any one site in particular. Inside, only global options can be set; you cannot use regular site directives in them.

For example, to enable the `debug` global option, which is commonly used to produce verbose logs for troubleshooting:

```caddy
{
	debug
}
```

**[Read the Global Options page](/docs/caddyfile/options) to learn more.**



## Addresses

An address always appears at the top of the site block, and is usually the first thing in the Caddyfile.

These are examples of valid addresses:

| Address              | Effect                            |
|----------------------|-----------------------------------|
| `example.com`        | HTTPS with managed [publicly-trusted certificate](/docs/automatic-https#hostname-requirements) |
| `*.example.com`      | HTTPS with managed [wildcard publicly-trusted certificate](/docs/caddyfile/patterns#wildcard-certificates) |
| `localhost`          | HTTPS with managed [locally-trusted certificate](/docs/automatic-https#local-https) |
| `http://`            | HTTP catch-all, affected by [`http_port`](/docs/caddyfile/options#http-port) |
| `https://`           | HTTPS catch-all, affected by [`https_port`](/docs/caddyfile/options#http-port) |
| `http://example.com` | HTTP explicitly, with a `Host` matcher |
| `example.com:443`    | HTTPS due to matching the [`https_port`](/docs/caddyfile/options#http-port) default |
| `:443`               | HTTPS catch-all due to matching the [`https_port`](/docs/caddyfile/options#http-port) default |
| `:8080`              | HTTP on non-standard port, no `Host` matcher |
| `localhost:8080`     | HTTPS on non-standard port, due to having a valid domain |
| `https://example.com:443` | HTTPS, but both `https://` and `:443` are redundant |
| `127.0.0.1` | HTTPS, with a locally-trusted IP certificate |
| `http://127.0.0.1` | HTTP, with an IP address `Host` matcher (rejects `localhost`) |


<aside class="tip">

[Automatic HTTPS](/docs/automatic-https) is enabled if your site's address contains a hostname or IP address. This behavior is purely implicit, however, so it never overrides any explicit configuration.

For example, if the site's address is `http://example.com`, auto-HTTPS will not activate because the scheme is explicitly `http://`.

</aside>


From the address, Caddy can potentially infer the scheme, host and port of your site. If the address is without a port, the Caddyfile will choose the port matching the scheme if specified, or the default port of 443 will be assumed.

If you specify a hostname, only requests with a matching `Host` header will be honored. In other words, if the site address is `localhost`, then Caddy will not match requests to `127.0.0.1`.

Wildcards (`*`) may be used, but only to represent precisely one label of the hostname. For example, `*.example.com` matches `foo.example.com` but not `foo.bar.example.com`, and `*` matches `localhost` but not `example.com`. See the [wildcard certificates pattern](/docs/caddyfile/patterns#wildcard-certificates) for a practical example.

To catch all hosts, omit the host portion of the address, for example, simply `https://`. This is useful when using [On-Demand TLS](/docs/automatic-https#on-demand-tls), when you don't know the domains ahead of time.

If multiple sites share the same definition, you can list all of them together, either with spaces or commas. The following three examples are equivalent:

```caddy
# Comma separated site addresses
localhost:8080, example.com, www.example.com {
	...
}
```

or

```caddy
# Space separated site addresses
localhost:8080 example.com www.example.com {
	...
}
```

or

```caddy
# Comma and new-line separated site addresses
localhost:8080,
example.com,
www.example.com {
	...
}
```

An address must be unique; you cannot specify the same address more than once.

[Placeholders](#placeholders) **cannot** be used in addresses, but you may use Caddyfile-style [environment variables](#environment-variables) in them:

```caddy
{$DOMAIN:localhost} {
	...
}
```

By default, sites bind on all network interfaces. If you wish to override this, use the [`bind` directive](/docs/caddyfile/directives/bind) or the [`default_bind` global option](/docs/caddyfile/options#default-bind) to do so.



## Matchers

HTTP handler [directives](#directives) apply to all requests by default (unless otherwise documented).

[Request matchers](/docs/caddyfile/matchers) can be used to classify requests by a given criteria. With matchers, you can specify exactly which requests a certain directive applies to.

For directives that support matchers, the first argument after the directive is the **matcher token**. Here are some examples:

```caddy-d
root *           /var/www  # matcher token: *
root /index.html /var/www  # matcher token: /index.html
root @post       /var/www  # matcher token: @post
```

Matcher tokens can be omitted entirely to match all requests; for example, `*` does not need to be given if the next argument does not look like a path matcher.

**[Read the Request Matchers page](/docs/caddyfile/matchers) to learn more.**




## Placeholders

You can use any [Caddy placeholders](/docs/conventions#placeholders) in the Caddyfile, but for convenience you can also use some equivalent shorthand ones:

| Shorthand       | Replaces                          |
|-----------------|-----------------------------------|
| `{cookie.*}`    | `{http.request.cookie.*}`         |
| `{client_ip}`   | `{http.vars.client_ip}`           |
| `{dir}`         | `{http.request.uri.path.dir}`     |
| `{err.*}`       | `{http.error.*}`                  |
| `{file_match.*}` | `{http.matchers.file.*}`         |
| `{file.base}`   | `{http.request.uri.path.file.base}` |
| `{file.ext}`    | `{http.request.uri.path.file.ext}`  |
| `{file}`        | `{http.request.uri.path.file}`    |
| `{header.*}`    | `{http.request.header.*}`         |
| `{host}`        | `{http.request.host}`             |
| `{hostport}`    | `{http.request.hostport}`         |
| `{labels.*}`    | `{http.request.host.labels.*}`    |
| `{method}`      | `{http.request.method}`           |
| `{path.*}`      | `{http.request.uri.path.*}`       |
| `{path}`        | `{http.request.uri.path}`         |
| `{port}`        | `{http.request.port}`             |
| `{query.*}`     | `{http.request.uri.query.*}`      |
| `{query}`       | `{http.request.uri.query}`        |
| `{re.*}`        | `{http.regexp.*}`                 |
| `{remote_host}` | `{http.request.remote.host}`      |
| `{remote_port}` | `{http.request.remote.port}`      |
| `{remote}`      | `{http.request.remote}`           |
| `{rp.*}`        | `{http.reverse_proxy.*}`          |
| `{scheme}`      | `{http.request.scheme}`           |
| `{tls_cipher}`  | `{http.request.tls.cipher_suite}` |
| `{tls_client_certificate_der_base64}` | `{http.request.tls.client.certificate_der_base64}` |
| `{tls_client_certificate_pem}`        | `{http.request.tls.client.certificate_pem}` |
| `{tls_client_fingerprint}`            | `{http.request.tls.client.fingerprint}`     |
| `{tls_client_issuer}`                 | `{http.request.tls.client.issuer}`          |
| `{tls_client_serial}`                 | `{http.request.tls.client.serial}`          |
| `{tls_client_subject}`                | `{http.request.tls.client.subject}`         |
| `{tls_version}` | `{http.request.tls.version}`      |
| `{upstream_hostport}` | `{http.reverse_proxy.upstream.hostport}` |
| `{uri}`         | `{http.request.uri}`              |
| `{vars.*}`      | `{http.vars.*}` |



## Snippets

You can define special blocks called snippets by giving them a name surrounded in parentheses:

```caddy
(logging) {
	log {
		output file /var/log/caddy.log
		format json
	}
}
```

And then you can reuse this anywhere you need, using the special [`import`](/docs/caddyfile/directives/import) directive:

```caddy
example.com {
	import logging
}

www.example.com {
	import logging
}
```

The [`import`](/docs/caddyfile/directives/import) directive can also be used to include other files in its place. If the argument does not match a defined snippet, it will be tried as a file. It also supports globs to import multiple files. As a special case, it can appear anywhere within the Caddyfile (except as an argument to another directive), including outside of site blocks:

```caddy
{
	email admin@example.com
}

import sites/*
```

You can pass arguments to an imported configuration (snippets or files) and use them like so:

```caddy
(snippet) {
	respond "Yahaha! You found {args[0]}!"
}

a.example.com {
	import snippet "Example A"
}

b.example.com {
	import snippet "Example B"
}
```

**[Read the `import` directive page](/docs/caddyfile/directives/import) to learn more.**


## Named Routes

⚠️ <i>Experimental</i>

Named routes use syntax similar to [snippets](#snippets); they're a special block defined outside of site blocks, prefixed with `&(` and ending in `)` with the name in between.

```caddy
&(app-proxy) {
	reverse_proxy app-01:8080 app-02:8080 app-03:8080
}
```

And then you can reuse this named route within any site:

```caddy
example.com {
	invoke app-proxy
}

www.example.com {
	invoke app-proxy
}
```

This is particularly useful to reduce memory usage if the same route is needed in many different sites, or if multiple different matcher conditions are needed to invoke the same route.

**[Read the `invoke` directive page](/docs/caddyfile/directives/invoke) to learn more.**



## Comments

Comments start with `#` and proceed until the end of the line:

```caddy-d
# Comments can start a line
directive  # or go at the end
```

The hash character `#` for a comment cannot appear in the middle of a token (i.e. it must be preceded by a space or appear at the beginning of a line). This allows the use of hashes within URIs or other values without requiring quoting.



## Environment variables

If your configuration relies on environment variables, you can use them in the Caddyfile:

```caddy
{$ENV}
```

Environment variables in this form are substituted **before Caddyfile parsing begins**, so they can expand to empty values (i.e. `""`), partial tokens, complete tokens, or even multiple tokens and lines.

For example, a environement variable `UPSTREAMS="app1:8080 app2:8080 app3:8080"` would expand to multiple [tokens](#tokens-and-quotes):

```caddy
example.com {
	reverse_proxy {$UPSTREAMS}
}
```

A default value can be specified for when the environment variable is not found, by using `:` as the delimiter between the variable name and the default value:

```caddy
{$DOMAIN:localhost} {

}
```

If you want to **defer the substitution** of an environment variable until runtime, you can use the [standard `{env.*}` placeholders](/docs/conventions#placeholders). Note that not all config parameters support these placeholders though, since module developers need to add a line of code to perform the replacement. If it doesn't seem to work, please file an issue to request support for it.

For example, if you have the [`caddy-dns/cloudflare` plugin <img src="/old/resources/images/external-link.svg" class="external-link">](https://github.com/caddy-dns/cloudflare) installed and wish to configure the [DNS challenge](/docs/automatic-https#dns-challenge), you can pass your `CLOUDFLARE_API_TOKEN` environment variable to the plugin like this:

```caddy
{
	acme_dns cloudflare {env.CLOUDFLARE_API_TOKEN}
}
```

If you're running Caddy as a systemd service, see [these instructions](/docs/running#overrides) for setting service overrides to define your environment variables.
