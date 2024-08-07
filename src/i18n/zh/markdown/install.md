---
title: "安装"
---

<h1 id="install">
  安装
</h1>

本页面介绍了多种安装 Caddy 的方法。

**官方渠道:**

- [Static binaries](#static-binaries)
- [Debian, Ubuntu, Raspbian packages](#debian-ubuntu-raspbian)
- [Fedora, RedHat, CentOS packages](#fedora-redhat-centos)
- [Arch Linux, Manjaro, Parabola packages](#arch-linux-manjaro-parabola)
- [Docker image](#docker)

<aside class="tip">

我们的 [official packages](https://github.com/caddyserver/dist) 仅包含了基本模组，如果您需要第三方插件，[build from source with `xcaddy`](/docs/build#xcaddy) 或是访问我们的 [下载页](/download)。

</aside>

**社区维护:**

- [Gentoo](#gentoo)
- [Homebrew (Mac)](#homebrew-mac)
- [Chocolatey (Windows)](#chocolatey-windows)
- [Scoop (Windows)](#scoop-windows)
- [Webi](#webi)
- [Ansible](#ansible)
- [Termux](#termux)
- [Nix/Nixpkgs/NixOS](#nixnixpkgsnixos)
- [Unikraft](#unikraft)
- [OPNsense](#opnsense)

## Static binaries

**在生产系统部署时，我们推荐您使用下方的官方软件包。**

1. 获得 Caddy 可执行文件:
	- [GitHub Release 页](https://github.com/caddyserver/caddy/releases) (展开 "Assets")
		- 参考 [Verifying Asset Signatures](/docs/signature-verification)，了解如何验证资源签名
	- 本站 [下载页](/download)
	- [从源码构建](/docs/build) (通过 `go` 或是 `xcaddy`)
2. [将 Caddy 安装为系统服务](/docs/running#manual-installation) 在生产环境强烈推荐

将可执行文件置于您的 `$PATH` (或 Windows 中的 `%PATH%`) 中的某条目录中，这样您就可以在任何路径下直接运行 `caddy` 且不需要输入完整的可执行文件路径。(运行 `echo $PATH` 可以查看目录列表。)

您可以通过替换更新版本的可执行文件来更新 Caddy，而 [`caddy upgrade` command](/docs/command-line#caddy-upgrade) 则可以更简单的做到这一点。

## Debian, Ubuntu, Raspbian

此软件包安装后系统会自动启动名为 `caddy` 的 [systemd service](/docs/running#linux-service)。同时会安装一个 `caddy-api` 服务，但默认情况下 _不_ 会自动启动，如果您需要通过 API 配置 Caddy，您可以自行切换到该服务。

安装完成后，您可以参考 [service usage instructions](/docs/running#using-the-service)。

**稳定版:**

<pre><code class="cmd"><span class="bash">sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list</span>
<span class="bash">sudo apt update</span>
<span class="bash">sudo apt install caddy</span></code></pre>

**测试版** (包含测试和预发布内容):

<pre><code class="cmd"><span class="bash">sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/caddy/testing/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-testing-archive-keyring.gpg</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/caddy/testing/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-testing.list</span>
<span class="bash">sudo apt update</span>
<span class="bash">sudo apt install caddy</span></code></pre>

[**查看 Cloudsmith 仓库**](https://cloudsmith.io/~caddy/repos/)

如果您想在自行构建的 Caddy 中使用软件包支持文件（系统服务，命令补全和默认配置），您可以从[这里](/docs/build#package-support-files-for-custom-builds-for-debianubunturaspbian)获得参考。

## Fedora, RedHat, CentOS

此软件包安装后会同时配置两种 [systemd service](/docs/running#linux-service)，但并不会默认启动。推荐使用服务管理 Caddy，您可以参考 [使用服务](/docs/zh/running#using-the-service)。

Fedora or RHEL/CentOS 8:

<pre><code class="cmd"><span class="bash">dnf install 'dnf-command(copr)'</span>
<span class="bash">dnf copr enable @caddy/caddy</span>
<span class="bash">dnf install caddy</span></code></pre>

RHEL/CentOS 7:

<pre><code class="cmd"><span class="bash">yum install yum-plugin-copr</span>
<span class="bash">yum copr enable @caddy/caddy</span>
<span class="bash">yum install caddy</span></code></pre>

[**查看 Caddy COPR**](https://copr.fedorainfracloud.org/coprs/g/caddy/caddy/)

## Arch Linux, Manjaro, Parabola

此软件包提供了两种 [systemd service](/docs/running#linux-service) 经过大量修改的版本，但并不会默认启动。
此软件包提供的修改版包括了定制的 启动/停止 行为，和一个额外的沙箱标志，您可参考 [systemd's exec documentation](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Sandboxing)，这可能会导致 Caddy 无法访问服务器上的某些路径。

<pre><code class="cmd"><span class="bash">pacman -Syu caddy</span></code></pre>

[**查看 Arch Linux 仓库中的 Caddy**](https://archlinux.org/packages/extra/x86_64/caddy/) 和 [**Arch Linux Wiki**](https://wiki.archlinux.org/title/Caddy)

## Docker

<pre><code class="cmd bash">docker pull caddy</code></pre>

[**查看 Docker Hub**](https://hub.docker.com/_/caddy)

参考我们的 [recommended Docker Compose configuration](/docs/running#docker-compose) 和使用引导.

## Gentoo

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd">emerge www-servers/caddy</code></pre>

[**查看 Gentoo Package**](https://packages.gentoo.org/packages/www-servers/caddy)

## Homebrew (Mac)

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd bash">brew install caddy</code></pre>

[**查看 Homebrew formula**](https://formulae.brew.sh/formula/caddy)

## Chocolatey (Windows)

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd">choco install caddy</code></pre>

[**查看 Chocolatey package**](https://chocolatey.org/packages/caddy)

## Scoop (Windows)

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd">scoop install caddy</code></pre>

[**查看 Scoop manifest**](https://github.com/ScoopInstaller/Main/blob/master/bucket/caddy.json)

## Webi

_提示: 这是社区维护的安装方式。_

Linux 和 macOS:

<pre><code class="cmd bash">curl -sS https://webi.sh/caddy | sh</code></pre>

Windows:

<pre><code class="cmd">curl.exe https://webi.ms/caddy | powershell</code></pre>

您可能需要配置 Windows 防火墙从而允许接收非本机链接。

[**查看 Webi**](https://webinstall.dev/caddy)

## Ansible

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd bash">ansible-galaxy install nvjacobo.caddy</code></pre>

[**查看 Ansible role repository**](https://github.com/nvjacobo/caddy)

## Termux

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd">pkg install caddy</code></pre>

[**查看 Termux build.sh file**](https://github.com/termux/termux-packages/blob/master/packages/caddy/build.sh)

## Nix/Nixpkgs/NixOS

_提示: 这是社区维护的安装方式。_

- Package name: [`caddy`](https://search.nixos.org/packages?channel=unstable&show=caddy&query=caddy)
- NixOS module: [`services.caddy`](https://search.nixos.org/options?channel=unstable&show=services.caddy.enable&query=services.caddy)

[**查看 Nixpkgs search 中的 Caddy**](https://search.nixos.org/packages?channel=unstable&show=caddy&query=caddy) and [**the NixOS options search**](https://search.nixos.org/options?channel=unstable&show=services.caddy.enable&query=services.caddy)

## Unikraft

_提示: 这是社区维护的安装方式。_

先安装 Unikraft 的配套工具 [`kraft`](https://unikraft.org/docs/cli):

<pre><code class="cmd">curl --proto '=https' --tlsv1.2 -sSf https://get.kraftkit.sh | sh</code></pre>

之后通过 Unikraft 运行 Caddy：

<pre><code class="cmd">kraft run --rm -p 2015:2015 --plat qemu --arch x86_64 -M 256M caddy:2.7</code></pre>

为了支持非本机的链接，您应该 [connect the unikernel instance to a network](https://unikraft.org/docs/cli/running#connecting-a-unikernel-instance-to-a-network)。

[**查看 Unikraft application catalog**](https://github.com/unikraft/catalog/tree/main/examples/caddy) 和 [**the KraftCloud platform examples (powered by Unikraft)**](https://github.com/kraftcloud/examples/tree/main/caddy).

## OPNsense

_提示: 这是社区维护的安装方式。_

<pre><code class="cmd">pkg install os-caddy</code></pre>

[**查看 FreeBSD caddy-custom makefile**](https://github.com/opnsense/ports/blob/master/www/caddy-custom/Makefile) 和 [**the os-caddy plugin source**](https://github.com/opnsense/plugins/tree/master/www/caddy)
