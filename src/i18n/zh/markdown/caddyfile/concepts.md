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
6. [引用片段(snippets)](#snippets)
7. [具名路由(named routes)](#named-routes)
8. [注释(comments)](#comments)
9. [环境变量(environment variables)](#environment-variables)

<h2 id="structure">
	结构(structure)
</h2>

Caddyfile 的结构可以直观的描述：

![Caddyfile structure](/i18n/zh/old/resources/images/caddyfile-visual.png)

关键点：

- [**全局选项**](#global-options) 应该出现在配置顶端。

- [引用片段](#snippets) 或 [具名路由](#named-routes) 应该尽量集中在一起。 

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

<h2 id="global-options">
	全局选项
</h2>

Caddyfile 可以由一个没有键的特殊块开始，这被称为 [全局选项块](/docs/caddyfile/options)：

```caddy
{
	...
}
```

如果使用全局选项，那么请将他放置在配置文件顶部。

全局选项被用于设置全局性的，而非针对某个站的设置。在块中只能使用全局关键字，不能使用普通站点中使用的关键字。

例如，为了开启全局 `debug` 选项（用于打印详细日志，进行故障排查）：

```caddy
{
	debug
}
```

**查看 [全局选项](/docs/caddyfile/options) 了解更多。**

<h2 id="addresses">
	地址
</h2>

地址是站点块的键，而且大部分情况下是 Caddyfile 的首行内容。

这些是合法地址的示例：

| Address              | Effect                            |
|----------------------|-----------------------------------|
| `example.com`        | 基于 [公共信任证书](/docs/automatic-https#hostname-requirements) 的 HTTPS |
| `*.example.com`      | 基于 [公共信任通配符证书](/docs/caddyfile/patterns#wildcard-certificates) 的 HTTPS |
| `localhost`          | 基于 [本地信任证书](/docs/automatic-https#local-https) 的 HTTPS |
| `http://`            | HTTP 通配，端口使用 [`http_port`](/docs/caddyfile/options#http-port) |
| `https://`           | HTTPS 通配，端口使用 [`https_port`](/docs/caddyfile/options#http-port) |
| `http://example.com` | 使用 `Host` 显示指定 HTTP |
| `example.com:443`    | 由于端口号等于 [`https_port`](/docs/caddyfile/options#http-port) 默认值，使用 HTTPS |
| `:443`               | 由于端口号等于 [`https_port`](/docs/caddyfile/options#http-port) 默认值，HTTPS 通配 |
| `:8080`              | 没有 `Host`，启动非默认端口的 HTTP |
| `localhost:8080`     | 由于有域名，启动非默认端口的 HTTPS |
| `https://example.com:443` | HTTPS，但 `https://` 和 `:443` 都不是必须的 |
| `127.0.0.1` | 基于本地信任 IP 证书的 HTTPS |
| `http://127.0.0.1` | HTTP，将 IP 地址作为 `Host`（不可用 `localhost` 访问） |

<aside class="tip">

当您的站点地址包含主机名或 IP 地址时，[自动 HTTPS](/docs/automatic-https) 会被默认开启，此行为是完全隐式的，当然，可以被显式的配置覆盖。

例如，如果站点地址为 `http://example.com`，则自动 HTTPS 不会生效，因为显示指定了 `http://` 协议。

</aside>

Caddy 会通过您的地址推断您站点的协议、域名和端口。如果地址中没有指定端口，Caddy 会在配置中寻找对应该协议的端口，或者使用默认端口 443。

如果您制定了域名，那么仅有携带匹配该域名 `Host` 头的请求才会被处理。换句话说，如果站点地址是 `localhost`，那么 Caddy 并不会处理访问 `127.0.0.1` 的请求。

您可以使用通配符(`*`)，但记住他只能用来表示域名中的一个标签。例如，`*.example.com` 可以匹配 `foo.example.com` 但不能匹配 
`foo.bar.example.com`，`*` 可以匹配 `localhost` 但不能匹配 `example.com`。查阅 [通配符认证模板](/docs/caddyfile/patterns#wildcard-certificates) 了解实例。

省略域名或 IP 地址，可以用来匹配所有地址，例如 `https://`。这在 [按需 TLS](/docs/automatic-https#on-demand-tls) 的情况下很有用，您也不知道随后会使用什么域名访问。

如果多个站点共享相同的定义，您可以将他们列举在一起，使用空格或逗号分隔。下方的三个例子是完全等价的：

```caddy
# Comma separated site addresses
localhost:8080, example.com, www.example.com {
	...
}
```

```caddy
# Space separated site addresses
localhost:8080 example.com www.example.com {
	...
}
```

```caddy
# Comma and new-line separated site addresses
localhost:8080,
example.com,
www.example.com {
	...
}
```

地址必须是唯一的，您不能将同一地址配置两次。

[占位符](#placeholders) **不能**用于地址，但您可以使用 Caddyfile 风格的环境变量 [环境变量](#environment-variables)：

```caddy
{$DOMAIN:localhost} {
	...
}
```

默认情况下，站点将会绑定到所有的网络接口。如果您向覆盖此配置，请使用 [`bind` 指令](/docs/caddyfile/directives/bind) 或 [`default_bind` 全局选项](/docs/caddyfile/options#default-bind)。

<h2 id="matchers">
	匹配器
</h2>

默认情况下，[指令](#directives) 会对所有的请求生效。

[请求匹配器](/docs/caddyfile/matchers) 可用于按给定标准对请求进行分类。使用匹配器，您可以准确指定某个指令适用于哪些请求。

对于适用匹配器的指令，指令的第一个实参就是 **匹配词元**。下面是一些例子：

```caddy-d
root *           /var/www  # matcher token: *
root /index.html /var/www  # matcher token: /index.html
root @post       /var/www  # matcher token: @post
```

不指定匹配词元意味着适配所有请求，比如，如果后续的实参看起来并不是路径匹配器的话，则可以省略 `*` 实参。

**阅读 [匹配器文档](/docs/caddyfile/matchers) 了解更多。**

<h2 id="placeholders">
	占位符
</h2>

您可以在 Caddyfile 中使用 [占位符](/docs/conventions#placeholders)，为了方便起见，您也可以使用简写：

| 简写       | 等效于                          |
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

<h2 id="snippets">
	引用片段
</h2>

您可以通过使用小括号包裹名称，来定义一个引用片段：

```caddy
(logging) {
	log {
		output file /var/log/caddy.log
		format json
	}
}
```

随后，您就可以使用 [`import`](/docs/caddyfile/directives/import) 指令在其他需要的地方复用这个片段：

```caddy
example.com {
	import logging
}

www.example.com {
	import logging
}
```

[`import`](/docs/caddyfile/directives/import) 指令也可用于引入其他文件中的内容，当该指令的参数无法匹配任何先前定义的复用片段时，就会把参数视为一个文件，指令还支持使用星号匹配多个文件。作为一种特例，import 指令可以出现在 Caddyfile 的任何地方，无论是站点块的内部还是外部（但不能作为其他指令的实际参数）。

```caddy
{
	email admin@example.com
}

import sites/*
```

您可以向配置片段或引用文件传递参数，就像这样：

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

**[参考 `import` 指令](/docs/caddyfile/directives/import)了解更多。**


<h2 id="named-routes">
	具名路由
</h2>

⚠️ <i>实验性</i>

具名路由的语法于 [引用片段](#snippets) 相似；都是在站点块之外定义的特殊块，具名路由的定义方式是使用 `&(` 和 `)` 包裹具名路由名称。

```caddy
&(app-proxy) {
	reverse_proxy app-01:8080 app-02:8080 app-03:8080
}
```

随后您就可以在任何站点中使用上述定义的具名路由：

```caddy
example.com {
	invoke app-proxy
}

www.example.com {
	invoke app-proxy
}
```

在多个不同的站点都需要进行相同的路由配置的时候，或是多个铍铜的匹配器条件都需要执行相同的路由的时候，具名路由功能非常实用。

**[参考 `invoke` 指令](/docs/caddyfile/directives/invoke)了解更多。**

<h2 id="comments">
	注释
</h2>

注释从 `#` 开始，持续到行尾：

```caddy-d
# Comments can start a line
directive  # or go at the end
```

注释的井号字符 `#` 不能出现在词元中（即它前面必须有空格或必须出现在行首）。这个特性允许用户直接在 URI 或其他值当中直接使用 `#` 而无需使用引号。

<h2 id="environment-variables">
	环境变量
</h2>

如果您的配置依赖环境变量，您可以在 Caddyfile 中这样引用：

```caddy
{$ENV}
```

**环境变量会在 Caddyfile 解析前完成替换**，因此他们可以是空值（例如 `""`）、次元的一部分、完整的词元、多个词元、甚至是一整行或是多行配置。

例如，环境变量 `UPSTREAMS="app1:8080 app2:8080 app3:8080"` 就会被替换为多个 [词元](#tokens-and-quotes)：

```caddy
example.com {
	reverse_proxy {$UPSTREAMS}
}
```

未找到环境变量时，可以使用默认值，使用 `:` 分割环境变量名称和默认值：

```caddy
{$DOMAIN:localhost} {

}
```

如果您想要把环境变量的替换动作**推迟到运行时**，您可以使用 [标准的 `{env.*}` 占位符](/docs/conventions#placeholders)。注意并非所有的配置参数都支持这些占位符，因为模组的开发者需要添加一行代码来执行替换动作。如果占位符没法正常使用，请您提交一个 issue 来请求支持。

例如，您正在使用 [`caddy-dns/cloudflare` plugin <img src="/old/resources/images/external-link.svg" class="external-link">](https://github.com/caddy-dns/cloudflare) 模组并想要配置 [DNS challenge](/docs/automatic-https#dns-challenge)，您可以像这样将 `CLOUDFLARE_API_TOKEN` 环境变量传入您的插件：

```caddy
{
	acme_dns cloudflare {env.CLOUDFLARE_API_TOKEN}
}
```

如果您将 Caddy 作为 systemd 服务运行，请参阅这些 [设置服务覆盖的说明](/docs/zh/running#overrides) 来定义您的环境变量。
