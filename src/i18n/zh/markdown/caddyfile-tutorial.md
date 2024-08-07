---
title: Caddyfile 教程
---

<h1 id="caddyfile-tutorial">
  Caddyfile 教程
</h1>

本教程指导您 [HTTP Caddyfile](/docs/caddyfile) 的基本用法，让您可以快速便捷的进行配置。

**目标：**
- 🔲 第一个站点
- 🔲 静态文件服务
- 🔲 模板
- 🔲 压缩
- 🔲 多站点
- 🔲 匹配器
- 🔲 环境变量
- 🔲 注释

**您需要准备：**
- 使用 终端/命令行
- 文本编辑器
- 将 `caddy` 配置到您的 PATH

---

创建名为 `Caddyfile` 的文件（不带扩展名）。

在首行输入您的站点 [地址](/docs/caddyfile/concepts#addresses)：

```caddy
localhost
```

<aside class="tip">

如果在您的系统中，HTTP 和 HTTPS 端口（通常是 80 和 443）是受到安全策略保护的，您需要提升权限或是换用更高端口号的端口。如果需要换用更高的端口号，您需要将刚才配置的地址修改为类似于 `localhost:2015` 这样的带端口配置，并且在 Caddyfile 中使用 [http_port](/docs/caddyfile/options) 配置切换 HTTP 服务端口。

</aside>

之后换行并配置站点的功能，在本教程中，请将您的 Caddyfile 配置为：

```caddy
localhost

respond "Hello, world!"
```

保存并运行 Caddy（在本教程中，我们使用 `--watch` 标签，这会监听 Caddyfile 的变更并自动改变配置）：

<pre><code class="cmd bash">caddy run --watch</code></pre>

<aside class="tip">

如果您遇到权限报错，请尝试使用更高的端口号比如 `localhost:2015` 并在 Caddyfile 中使用 [http_port](/docs/caddyfile/options) 配置切换 HTTP 服务端口，或者您也可以提升权限，在更高的权限下运行 Caddy。

</aside>

初次运行时，系统将会要求您输入密码，这是为了让 Caddy 能以 HTTPS 方式运行站点的必须操作。

<aside class="tip">

只要您在站点地址中配置了主机名、域名或者 IP，Caddy 就会默认以 HTTPS 的方式运行站点。您可以通过手动添加 `http://` 前缀来关闭 [自动 HTTPS](/docs/automatic-https) 功能。

</aside>

<aside class="complete">第一个站点</aside>

在浏览器中打开 [localhost](https://localhost)，您将会看到您的网站在 HTTPS 环境下运行！

<aside class="tip">
	如果您初次打开的时候遇到证书报错，您可以需要重启您的浏览器。
</aside>

以上的能力很好，但是还不够好，接下来我们把 static response 替换为 [file server](/docs/caddyfile/directives/file_server)，让 Caddy 成为一个静态文件服务器：

```caddy
localhost

file_server browse
```

保存 Caddyfile 之后刷新浏览器标签页，您将会看到一个文件列表，如果您的文件夹中有 index.html 文件的话，您则会看到一个网页。

<aside class="complete">静态文件服务</aside>

<h2 id="adding-functionality">
  添加功能
</h2>

接下来让我们对文件服务器做一些更有趣的事：提供一个带页面模板功能的服务。新建一个文件并复制这些内容：

```html
<!DOCTYPE html>
<html>
	<head>
		<title>Caddy tutorial</title>
	</head>
	<body>
		Page loaded at: {{`{{`}}now | date "Mon Jan 2 15:04:05 MST 2006"{{`}}`}}
	</body>
</html>
```

在当前路径下，将文件保存为 `caddy.html`，通过浏览器打开他：[https://localhost/caddy.html](https://localhost/caddy.html)

您将会看到：

```
Page loaded at: {{`{{`}}now | date "Mon Jan 2 15:04:05 MST 2006"{{`}}`}}
```

稍等，我们不是应该看到今天的日期吗？为什么模板没有生效呢？这是因为服务器尚未添加模板处理的配置！很简单，只需要在 Caddyfile 中添加一行：

```caddy
localhost

templates
file_server browse
```

保存，之后刷新浏览器的标签页，您将会看到：

```
Page loaded at: {{now | date "Mon Jan 2 15:04:05 MST 2006"}}
```

通过 Caddy 的 [模板模组](/docs/modules/http.handlers.templates)，您可以在静态文件中实现很多功能，例如包括其他 HTML，发送子请求，设置响应头，使用数据结构，等等！

<aside class="complete">模板</aside>

使用现代的压缩算法来处理响应是一个非常好的实践，我们通过 [`encode`](/docs/caddyfile/directives/encode) 指令来启用 Gzip 和 Zstandard 支持。

```caddy
localhost

encode zstd gzip
templates
file_server browse
```

<aside class="complete">压缩</aside>

这可是创建比较先进的，生产标准网站的必备过程！

当您准备好使用 [自动 HTTPS](/docs/automatic-https) 后，您只需要将站点地址配置（在本教程中是 `localhost`）替换为您的域名。参考 [快速了解 HTTPS](/docs/zh/quick-starts/https)。

<h2 id="multiple-sites">
  多站点
</h2>

如果使用当前的 Caddyfile 格式，我们只能配置一个站点！因为我们将 Caddyfile 的首行作为站点名称，所有其他的指令都用来描述站点功能。

但是，有很简单的方法可以让我们配置多个站点！

当前的 Caddyfile 是这样的：

```caddy
localhost

encode zstd gzip
templates
file_server browse
```

他实际上等价于：

```caddy
localhost {
	encode zstd gzip
	templates
	file_server browse
}
```

下面的这种配置方式使用大括号 `{ }` 包裹了每一个站点配置，因此我们可以在单个 Caddyfile 中配置多个站点。

例如：

```caddy
:8080 {
	respond "I am 8080"
}

:8081 {
	respond "I am 8081"
}
```

当使用大括号进行多个站点配置时，在大括号外侧的只能是 [站点地址](/docs/caddyfile/concepts#addresses)，在大括号内侧的则只能是关于该站点的 [指令](/docs/caddyfile/directives)。

如果需要为多个站点应用同一套配置，您可以同时使用多个地址，例如：

```caddy
:8080, :8081 {
	...
}
```

您可以定义任意多个站点，只要确保他们的地址是唯一的。

<aside class="complete">多站点</aside>

<h2 id="matchers">
  匹配器
</h2>

有时候我们只想将指令适用于一部分特定的请求。例如，假设我们的服务器同时支持文件服务和反向代理，但是很显然我们不可能使用这两种方式同时响应某一个请求！要么我们使用静态文件服务，提供一个文件给他，要么就使用反向代理服务，把请求丢到后端处理。

这样的配置并不会按照我们的预期工作：

```caddy
localhost

file_server
reverse_proxy 127.0.0.1:9005
```

在实践中，我们一般会想要把对后台接口的请求交给反向代理处理，例如以 `/api/` 开头的请求。我们可以通过添加 [匹配器 token](/docs/caddyfile/matchers#syntax) 来轻松实现这一点。

```caddy
localhost

file_server
reverse_proxy /api/* 127.0.0.1:9005
```

行了，这样 `/api/` 的请求会交给反向代理服务优先处理。

刚才我们添加的 `/api/*` 被称为 **匹配器 token**。您可以这样理解，因为他由 `/` 起始而且位于指令之后，所以他是匹配器 token（当然您也可以随时查阅 [指令文档](/docs/caddyfile/directives) 来进行确认）。

匹配器非常强大，您甚至可以使用 `@name` 的方式来对多条路径进行一次匹配！请您在继续阅读之前花一点时间 [了解匹配器](/docs/caddyfile/matchers)。

<aside class="complete">匹配器</aside>

<h2 id="environment-variables">
  环境变量
</h2>

Caddyfile 适配器可以在加载 Caddyfile 时获得并使用 [环境变量](/docs/caddyfile/concepts#environment-variables)。

首先，设置一个环境变量（在 Caddy 运行的终端上）：

<pre><code class="cmd bash">export SITE_ADDRESS=localhost:9055</code></pre>

之后，在 Caddyfile 中这样使用环境变量：

```caddy
{$SITE_ADDRESS}

file_server
```

在 Caddyfile 被解析前，他会被替换成：

```caddy
localhost:9055

file_server
```

您可以在 Caddyfile 的任何位置使用环境变量，无论多少次，无论是什么字符。

<aside class="complete">环境变量</aside>

<h2 id="comments">
  注释
</h2>

最后，使用 `#` 表示注释：

```caddy
# this starts a comment
```

<aside class="complete">注释</aside>

<h2 id="further-reading">
  延申阅读
</h2>

- [Caddyfile concepts](/docs/caddyfile/concepts)
- [Directives](/docs/caddyfile/directives)
- [Common patterns](/docs/caddyfile/patterns)
