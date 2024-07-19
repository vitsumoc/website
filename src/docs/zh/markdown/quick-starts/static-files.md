---
title: 静态资源服务
---

<h1 id="static-files-quick-start">
	快速了解静态资源服务
</h1>

本教程将让您快速了解如何搭建一个静态资源服务器。

**您需要准备：**
- 使用 终端/命令行
- 将 `caddy` 配置到您的 PATH
- 包含您站点资源的文件夹

---

有两种方式启动一个静态资源服务器。

<h2 id="command-line">
  命令行
</h2>

通过您的终端，在您站点资源的根目录执行：

<pre><code class="cmd bash">caddy file-server</code></pre>

如果您遇到了权限错误，可能是因为您的服务器禁止您使用较低的端口号 —— 可以更改为一个较高的端口号比如：

<pre><code class="cmd bash">caddy file-server --listen :2015</code></pre>

之后，在您的浏览器访问 [localhost](http://localhost)（或 [localhost:2015](http://localhost:2015)），即可看到您的站点。

如果您没有 index 文件，而是想通过列表形式显示您的静态资源内容，可以添加 `--browse` 标志：

<pre><code class="cmd bash">caddy file-server --browse</code></pre>

您也可以在命令中指定根目录的位置：

<pre><code class="cmd bash">caddy file-server --root ~/mysite</code></pre>

## Caddyfile

在您站点的根目录，创建 `Caddyfile` 并包括以下内容：

```caddy
localhost

file_server
```

如果您没有权限使用较低的端口号，请将 `localhost` 替换为 `localhost:2015` （或其他端口号）。

之后，在此路径下，运行：

<pre><code class="cmd bash">caddy run</code></pre>

您已经可以通过 [localhost](https://localhost) （或是其他您所配置的地址） 访问您的站点！

[`file_server` 指令](/docs/caddyfile/directives/file_server) 包含很多可配置指令用来定制您的站点，当您完成配置修改后，请记得 [重载](/docs/command-line#caddy-reload) Caddy （或是重启 Caddy）。

如果您没有 index 文件，而是想通过列表形式显示您的静态资源内容，可以添加 `browse` 标志：

```caddy
localhost

file_server browse
```

您也可以指定其他文件夹作为根目录：

```caddy
localhost

root * /var/www/mysite
file_server
```