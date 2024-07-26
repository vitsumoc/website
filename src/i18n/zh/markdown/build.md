---
title: "从源码构建"
---

<h1 id="build-from-source">
  从源码构建
</h1>

构建 Caddy 时有很多可选项（例如添加插件），如果您需要自行构建 Caddy 时，您可以：
- [Git](#git): 通过 Git 仓库构建
- [`xcaddy`](#xcaddy): 使用 `xcaddy` 构建
- [Docker](#docker): 构建一个自定义 Docker 镜像

您需要准备：

- [Go](https://golang.org/doc/install) 1.20 或以上

[Debian/Ubuntu/Raspbian 自构建时使用软件包支持文件](#package-support-files-for-custom-builds-for-debianubunturaspbian) 部分适用于通过 APT 命令在 Debian 系操作系统上安装了 Caddy，但仍需要使用自定义构建的可执行文件进行操作的用户。

## Git

您需要准备：

- Go (同上)

克隆仓库:

<pre><code class="cmd bash">git clone "https://github.com/caddyserver/caddy.git"</code></pre>

如果您没有 git，您也可以从 [GitHub](https://github.com/caddyserver/caddy) 下载源码压缩包。同时，每个 [release](https://github.com/caddyserver/caddy/releases) 也会带有改版本的源码。 

构建:

<pre><code class="cmd"><span class="bash">cd caddy/cmd/caddy/</span>
<span class="bash">go build</span></code></pre>

<aside class="tip">

由于 [Go 中的一个 bug](https://github.com/golang/go/issues/29228)，这种安装方式并不会携带版本信息。如果您想运行 (`caddy version`) 命令，您必须将 Caddy 作为一个依赖而非主模块来编译，这部分在 Caddy 的 [main.go](https://github.com/caddyserver/caddy/blob/master/cmd/caddy/main.go) 文件中有所说明。或者，您可以使用 [`xcaddy`](#xcaddy) 解决这个问题。

</aside>

Go 程序可以很方便的进行交叉编译 ，只需要配置 `GOOS`，`GOARCH` 和 `GOARM` 环境变量。（[查看 Go 文档了解更多](https://golang.org/doc/install/source#environment)）

例如，当您在非 Windows 环境中想要编译 Windows 环境的 Caddy 时：

<pre><code class="cmd bash">GOOS=windows go build</code></pre>

或者类似的，当您不处于 Linux ARMv6 环境时需要编译 Linux ARMv6 的包时：

<pre><code class="cmd bash">GOOS=linux GOARCH=arm GOARM=6 go build</code></pre>

## xcaddy

[`xcaddy` 命令](https://github.com/caddyserver/xcaddy) 是构建带有版本信息或是带有插件的 Caddy 的最简单方式。

您需要准备：

- Go (同上)
- 将 [`xcaddy`](https://github.com/caddyserver/xcaddy/releases) 配置到您的 `PATH` 中

您 **无需** 下载 Caddy 源码（xcaddy 会自动帮您下载）。

构建带有版本号的 Caddy 很简单：

<pre><code class="cmd bash">xcaddy build</code></pre>

如果需要和插件一起构建，则使用 `--with`：

<pre><code class="cmd bash">xcaddy build \
    --with github.com/caddyserver/nginx-adapter
	--with github.com/caddyserver/ntlm-transport@v0.1.1</code></pre>

如您所见，您可以通过 `@` 指定插件版本号，版本号可以是一个 tag，提交的 SHA，或是一个 branch。

使用 `xcaddy` 交叉编译的方式和使用 `go` 命令相同，例如，编译 macOS 版本：

<pre><code class="cmd bash">GOOS=darwin xcaddy build</code></pre>

## Docker

您可以将 `:builder` 镜像作为构建自定义 Caddy 包的快捷方式：

```Dockerfile
FROM caddy:<version>-builder AS builder

RUN xcaddy build \
    --with github.com/caddyserver/nginx-adapter \
    --with github.com/hairyhenderson/caddy-teapot-module@v0.0.3-0

FROM caddy:<version>

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

记得将 `<version>` 替换为 Caddy 的最新版本号。

注意第二个 `FROM` 指令，使用新的二进制文件覆盖 `caddy` 镜像，可以生成一个更小的镜像文件。

这种构建方式使用 `xcaddy` 构建带有模块的 Caddy，如同 [前文描述](#xcaddy) 的那样。

要使用 Docker Compose，可以参考我们推荐的 [`compose.yml`](/docs/running#docker-compose) 和使用说明。

<h2 id="package-support-files-for-custom-builds-for-debianubunturaspbian">
  Debian/Ubuntu/Raspbian 自构建时使用软件包支持文件
</h2>

此操作的目标是通过 `caddy` 包的软件包支持文件来运行自制的 `caddy` 程序，从而让用户可以使用官方包的默认配置，服务管理文件和命令补全。

您需要准备：
- 通过 [安装引导](/docs/install#debian-ubuntu-raspbian) 完成了 `caddy` 包的安装
- 完成了您的自定义 `caddy` 程序（参考上文），或是已经 [下载](/download) 一个自定义版本
- 已经将您的 `caddy` 程序放置到正确的路径中

操作:
<pre><code class="cmd"><span class="bash">sudo dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy</span>
<span class="bash">sudo mv ./caddy /usr/bin/caddy.custom</span>
<span class="bash">sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10</span>
<span class="bash">sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom 50</span>
<span class="bash">sudo systemctl restart caddy</span>
</code></pre>

说明:

- `dpkg-divert` 将把 `/usr/bin/caddy` 可执行文件移动到 `/usr/bin/caddy.default`，并在此位置配置一个转向，以防任何软件包想在此位置安装文件。

- `update-alternatives` 会创建一个软链接，将 `/usr/bin/caddy` 指向需要的 caddy程序

- `systemctl restart caddy` 会停止原有的 Caddy 服务，并重新启动新的、自定义的服务。

您可以通过这个命令在自定义或是默认的 `caddy` 之间切换，跟随命令指引操作并随后重启 Caddy 服务。

<pre><code class="cmd bash">update-alternatives --config caddy</code></pre>

在此之后，如果您想要更新 Caddy，您可以使用 [`caddy upgrade`](/docs/command-line#caddy-upgrade) 命令。这将会尝试从 [download](/download) 中下载一个与您当前构建插件配置相同的，最新版本的 caddy，随后替换原有的文件。
