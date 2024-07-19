---
title: 快速了解反向代理
---

<h1 id="reverse-proxy-quick-start">
	快速了解反向代理
</h1>

本教程将让您快速了解如何搭建一个反向代理服务器，并按照您的需求开启或关闭 HTTPS 功能。

This guide will show you how to get a production-ready reverse proxy with or without HTTPS up and running quickly.

**您需要准备：**
- 使用 终端/命令行
- 将 `caddy` 配置到您的 PATH
- 一个需要被代理的后台服务

---

本教程假设您有一个 HTTP 后台服务运行在 `127.0.0.1:9000`，教程中的环境为 Linux 系统，但大部分命令在其他操作系统中也可以使用。

您可以在不适用配置文件的情况下搭建一个简单的反向代理服务器，或是通过配置文件创造一个拥有更多功能的服务。

<h2 id="command-line">
  命令行
</h2>

通过普通的反向代理服务将 9000 端口代理到您服务器上的 2080 端口：

<pre><code class="cmd bash">caddy reverse-proxy --from :2080 --to :9000</code></pre>

之后测试效果：

<pre><code class="cmd bash">curl -v 127.0.0.1:2080</code></pre>

[`reverse-proxy` 命令](/docs/command-line#reverse-proxy) 专门用于快速开启反向代理服务。（如果您的生产环境需求很简单，您可以直接使用这个命令。）

## Caddyfile

在当前工作路径中，创建名为 `Caddyfile` 的文件，并输入内容：

```caddy
:2080

reverse_proxy :9000
```

这个配置的效果和上文中的 `caddy reverse-proxy` 命令相同。

之后，依然在此路径下，运行：

<pre><code class="cmd bash">caddy run</code></pre>

并测试您的代理服务：

<pre><code class="cmd bash">curl -v 127.0.0.1:2080</code></pre>

如果您修改了 Caddyfile 文件，记得 [重载](/docs/command-line#caddy-reload) Caddy。

这是一个非常简单的示例，实际上您可以利用 [`reverse_proxy` 指令](/docs/caddyfile/directives/reverse_proxy) 实现更多的功能。

<h2 id="https-from-client-to-proxy">
  从客户端到代理的 HTTPS
</h2>

如果向 Caddy 配置了您的主机名（或域名），Caddy 会 [自动为您提供HTTPS](/docs/automatic-https) 代理服务。如果使用 `caddy reverse-proxy` 命令时不添加 `--from` 标志，`caddy` 会自动使用 `localhost` 作为其内容，或者您也可以修改 Caddyfile 的首行，将其替换为代理服务的域名。

- 当您使用 `localhost` 或是任何以 `.localhost` 结尾的域名时，Caddy 会创建一个自动更新的自签证书。当您第一次进行这个操作时，由于 Caddy 会尝试将证书安装到您服务器的 trust store，您可能需要输入密码进行授权。
- 如果您使用其他的域名，Caddy 会尝试创建一个对公众可用的证书，您需要确定您的 DNS 记录正确指向这台机器，并且 80 和 443 端口开放，并由 Caddy 提供服务。

如果您未指定一个端口，Caddy 将默认使用 443 端口提供 HTTPS 服务，在这种情况下可能您仍然需要提供权限（以便使用 80 和 443 端口号），在 Linux 中实现的方式有两种：

- 提升到 root 权限 (例如 `sudo -E`)。
- 执行 `sudo setcap cap_net_bind_service=+ep $(which caddy)` 为 Caddy 授权。

能够提供 HTTPS 服务的最基础的命令方式是这样：

<pre><code class="cmd bash">caddy reverse-proxy --to :9000</code></pre>

您可以这样测试：

<pre><code class="cmd bash">curl -v https://localhost</code></pre>

您可以通过 `--from` 标志指定域名：

<pre><code class="cmd bash">caddy reverse-proxy --from example.com --to :9000</code></pre>

如果您没有使用较低端口号的权限，您可以将服务代理到一个更高的端口号：

<pre><code class="cmd bash">caddy reverse-proxy --from example.com:8443 --to :9000</code></pre>

如果您使用 Caddyfile，只需要更改首行即可更换提供代理服务的域名，例如：

```caddy
example.com

reverse_proxy :9000
```

<h2 id="https-from-proxy-to-backend">
  从代理到后端服务的 HTTPS
</h2>

当后端服务支持 TLS 的时候，Caddy 也可以在代理和后端服务之间使用 HTTPS，您只需要在后端服务地址前添加 `https://`：

<pre><code class="cmd bash">caddy reverse-proxy --from :2080 --to https://localhost:9000</code></pre>

这要求后端拥有 Caddy 运行所在的系统认可的证书。（除非特意配置，否则 Caddy 并不会认可自签名证书。）

当然，您也可以在反向代理的两端都使用 HTTPS：

<pre><code class="cmd bash">caddy reverse-proxy --from example.com --to https://example.com:9000</code></pre>

这将会在客户端到反向代理服务，以及反向代理服务到后端的两条路径上都启用 HTTPS。

如果您代理服务的域名和后端服务的域名并不相同，您需要使用 `--change-host-header` 标志：

<pre><code class="cmd bash">caddy reverse-proxy \
	--from example.com \
	--to https://localhost:9000 \
	--change-host-header</code></pre>

默认情况下，Caddy 会透明传递所有的 HTTP 头，当然也包括 `Host`，而且 Caddy 通过 `Host` 头获取 TLS 服务名称。`--change-host-header` 标志会将 `Host` 头修改为后端服务的域名，从而使 TLS 握手成功。在上面的例子中，Host 头的内容会从 `example.com` 变更为 `localhost:9000`（随后在进行 TLS 握手时会使用 `localhost`）。