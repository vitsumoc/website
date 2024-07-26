---
title: 快速了解 Caddyfile
---

<h1 id="caddyfile-quick-start">
	快速了解 Caddyfile
</h1>

创建一个名为 `Caddyfile` 的文件（不带扩展名）。

首先写入您的站点地址：

```caddy
localhost
```

<aside class="tip">

如果您系统中的 HTTP 和 HTTPS 端口（分别是 80 和 443）有安全策略，您可能需要提升操作权限或是尝试使用别的端口号。提升权限的方式是使用 `sudo -E` 获得 root 权限，或是 `sudo setcap cap_net_bind_service=+ep $(which caddy)`。如果使用别的端口号的话，例如您可以将上述的站点地址改为 `localhost:2080`，同时需要修改 Caddyfile 内容中的 `http_port` [配置项](/docs/caddyfile/options)。

</aside>

然后按 Enter 并输入您想要他执行的操作，比如：

```caddy
localhost

respond "Hello, world!"
```

保存文件，然后在文件所在路径运行 Caddy。

<pre><code class="cmd bash">caddy start</code></pre>

系统可能会提示您输入密码，因为 Caddy 提供的所有服务 —— 包括本地服务 —— 都默认支持 HTTPS。（您只需要在第一次运行的时候输入密码！）

<aside class="tip">

为了实现本地 HTTPS，Caddy 自动生成了证书和私钥。根证书需要安装到您的系统信任仓库，因此需要输入密码进行确认。这将会让您可以在本地就拥有 HTTPS 的开发环境，而不必担心证书错误。

</aside>

（如果您遇到了权限错误，您可能需要提权运行，或是选择一个 1023 以上的端口号。）

打开您的 [浏览器](http://localhost) 或是使用 `curl` 访问：

<pre><code class="cmd"><span class="bash">curl https://localhost</span>
Hello, world!</code></pre>

您可以利用大括号 `{ }` 在 Caddyfile 中区分多个站点，例如：

```caddy
localhost {
	respond "Hello, world!"
}

localhost:2016 {
	respond "Goodbye, world!"
}
```

您有两种更新 Caddy 配置的方式，比如使用 API：

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: text/caddyfile" \
	--data-binary @Caddyfile
</code></pre>

或是使用重载命令，实际上 caddy 会在内部调用 API：

<pre><code class="cmd bash">caddy reload</code></pre>

可以在 [浏览器](https://localhost:2016) 或是 `curl` 中测试您新配置的 “goodbye” 服务：

<pre><code class="cmd"><span class="bash">curl https://localhost:2016</span>
Goodbye, world!</code></pre>

当您不再使用 Caddy，记得关闭：

<pre><code class="cmd bash">caddy stop</code></pre>

<h2 id="further-reading">
  延伸阅读
</h2>

- [Caddyfile concepts](/docs/caddyfile/concepts)
- [Directives](/docs/caddyfile/directives)
- [Common patterns](/docs/caddyfile/patterns)
