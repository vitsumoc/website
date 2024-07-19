---
title: 快速了解 HTTPS
---

<h1 id="https-quick-start">
	快速了解 HTTPS
</h1>

本教程将让您了解如何快速搭建一个 [自动管理的 HTTPS](/docs/automatic-https) 服务器。

<aside class="tip">
	只要您在配置中提供了域名，Caddy 就会默认为站点启动 HTTPS 服务。本教程假定您想要搭建一个互联网环境（而非 localhost）的 HTTPS 服务器，因此我们需要准备一个域名和具备互联网端口的服务器。
</aside>

**您需要准备：**
- 使用 终端/命令行
- 了解 DNS
- 一个经过注册的域名
- 面向互联网的 80 和 443 端口
- 将 `caddy` 和 `curl` 配置到您的 PATH

---

在本教程中，我们假设您的域名是 `example.com`。

将您域名的 A 或 AAAA 记录指向您的服务器，您可以在您的域名提供商的页面中完成这项操作。

在继续之前，需要通过一次权威查询确认您的 DNS 记录生效，请将下方命令中的 `example.com` 替换为您的实际域名，如果您使用 IPv6，请记得将 `type=A` 替换为 `type=AAAA`：

<pre><code class="cmd bash">curl "https://cloudflare-dns.com/dns-query?name=example.com&type=A" \
  -H "accept: application/dns-json"</code></pre>

之后，同样要检查并确认您的服务器的 80 和 443 端口可以被互联网访问。

<aside class="tip">
	如果您使用的是家庭网络或是其他受限制的网络，您可能需要配置端口映射，或是调整防火墙策略。
</aside>

接下来只需要将您的域名配置到 Caddy，并且运行 Caddy 即可，我们提供了若干种配置方式。

## Caddyfile

这是搭建 HTTPS 服务器最常用的方式。

创建一个名为 `Caddyfile`（没有扩展名）的文件，在其首行写入您的域名，例如：

```caddy
example.com

respond "Hello, privacy!"
```

之后，依然在此路径下，运行：

<pre><code class="cmd bash">caddy run</code></pre>

由于您的配置中已经包括了域名，接下来您将会看到 Caddy 提供了 TLS 证书并且以 HTTPS 形式运行了您的站点。

<h2 id="the-file-server-command">
  <code>file-server</code> 命令
</h2>

如果您想做的只是通过 HTTPS 提供一个静态资源站或是静态网站，您可以使用下述命令（记得替换您自己的域名）：

<pre><code class="cmd bash">caddy file-server --domain example.com</code></pre>

您将会发现 Caddy 提供了 TLS 证书并且以 HTTPS 形式运行了您的站点。

<h2 id="the-reverse-proxy-command">
  <code>the-reverse-proxy-command</code> 命令
</h2>

如果您想要的只是通过 Caddy 提供 HTTPS 的反向代理，运行下述命令（记得替换您自己的域名和自己的后端服务地址）：

<pre><code class="cmd bash">caddy reverse-proxy --from example.com --to localhost:9000</code></pre>

您将会发现 Caddy 提供了 TLS 证书并且以 HTTPS 形式运行了您的站点。

## JSON config

任何 [host matcher](/docs/json/apps/http/servers/routes/match/host/) 都会触发自动 HTTPS。

因此，如下所示的 JSON 配置将启用 [自动 HTTPS](/docs/automatic-https)：

```json
{
	"apps": {
		"http": {
			"servers": {
				"hello": {
					"listen": [":443"],
					"routes": [
						{
							"match": [{
								"host": ["example.com"]
							}],
							"handle": [{
								"handler": "static_response",
								"body": "Hello, privacy!"
							}]
						}
					]
				}
			}
		}
	}
}
```
