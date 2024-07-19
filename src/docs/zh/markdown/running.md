---
title: 服务化运行
---

<h1 id="running">
  服务化运行
</h1>

虽然可以通过 [命令行](/docs/command-line) 直接运行 Caddy，但是将 Caddy 注册为服务来运行则会有更多的便利，例如可以确保在系统重启后启动 Caddy 并捕捉 stdout/stderr 日志。

- [Linux 服务](#linux-service)
  - [Unit Files](#unit-files)
  - [手动安装](#manual-installation)
  - [使用服务](#using-the-service)
  - [本地 HTTPS](#local-https-with-systemd)
  - [重写](#overrides)
  - [SELinux 注意事项](#selinux-considerations)
- [Windows 服务](#windows-service)
  - [sc.exe](#scexe)
  - [WinSW](#winsw)
- [Docker Compose](#docker-compose)
  - [安装](#setup)
  - [使用](#usage)
  - [本地 HTTPS](#local-https-with-docker)

<h3 id="linux-service">
  Linux 服务
</h3>

在 Linux 发行版中通过 systemd 运行 Caddy 的推荐做法是使用我们提供的官方 unit files。

### Unit Files

我们提供了两个不同的 systemd unit files 供您选择。

- [**`caddy.service`**](https://github.com/caddyserver/dist/blob/master/init/caddy.service) 适于您使用 [Caddyfile](/docs/caddyfile) 进行配置的场景. 如果您想要使用其他的配置格式或是 JSON 文件，您可以 [override](#overrides) `ExecStart` 和 `ExecReload` 命令。

- [**`caddy-api.service`**](https://github.com/caddyserver/dist/blob/master/init/caddy-api.service) 适于您仅使用 [API](/docs/api) 进行配置的场景。此服务将以 [`--resume`](/docs/command-line#caddy-run) 标志启动，默认将配置内容 [persisted](/docs/json/admin/config/) 保存到 `autosave.json` 中。

这两种启动方式非常相似，只是在 `ExecStart` 和 `ExecReload` 命令方面有所不同。

如果您需要在两种运行方式间切换，您应当停止服务并启动另一个服务。例如，当您需要从 `caddy` 服务切换到 `caddy-api` 服务时：
<pre><code class="cmd"><span class="bash">sudo systemctl disable --now caddy</span>
<span class="bash">sudo systemctl enable --now caddy-api</span></code></pre>

<h3 id="manual-installation">
  手动安装
</h3>

有些 [安装方式](/docs/zh/install) 会自行将 Caddy 注册为服务. 但如果您选择手动安装，您可以参考下述流程:

**您需要准备：**

- 您 [下载](/download) 或 [从源码构建](/docs/build) 的 `caddy` 包
- 232 或更高版本的 `systemctl --version`
- `sudo` 权限

将 caddy 包置入您的 `$PATH`，例如：
<pre><code class="cmd bash">sudo mv caddy /usr/bin/</code></pre>

测试其是否能工作：
<pre><code class="cmd bash">caddy version</code></pre>

创建名为 `caddy` 的 group：
<pre><code class="cmd bash">sudo groupadd --system caddy</code></pre>

创建拥有 home 路径的名为 `caddy` 的用户：
<pre><code class="cmd bash">sudo useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy</code></pre>

确保您的 `caddy` 用户具备对配置文件的读权限。

基于您的使用方式，选择一个 [unit file](#unit-files)。

**再次确认 `ExecStart` 和 `ExecReload` 中的命令。**确保其中的软件包位置和命令行标志与您实际安装的情况相同！例如：当您使用配置文件时，检查 `--config` 配置的路径是否与您配置文件的路径相同。

一般情况下，service 文件保存在：`/etc/systemd/system/caddy.service`

service file 保存无误后，您就可以启动 caddy 服务：
<pre><code class="cmd"><span class="bash">sudo systemctl daemon-reload</span>
<span class="bash">sudo systemctl enable --now caddy</span></code></pre>

并检查运行状态:
<pre><code class="cmd bash">systemctl status caddy</code></pre>

现在，您已经准备好 [使用服务](#using-the-service)了!

<h3 id="using-the-service">
  使用服务
</h3>

当您使用 Caddyfile 时，您可以通过 `nano`，`vi` 或者其他文本编辑工具来修改配置：
<pre><code class="cmd bash">sudo nano /etc/caddy/Caddyfile</code></pre>

您可以将您的静态资源放置在 `/var/www/html` 或是 `/srv`，记得确保您的 `caddy` 用户有权读取这些文件。

为了确认服务正在运行：
<pre><code class="cmd bash">systemctl status caddy</code></pre>
status 命令同时会展示当前运行的服务文件的路径。

当使用我们提供的官方服务文件运行时，Caddy 的输出会被重定向到 `journalctl`。当您想要查看完整的日志时：
<pre><code class="cmd bash">journalctl -u caddy --no-pager | less +G</code></pre>

您可以在配置文件改变后热重启Caddy：
<pre><code class="cmd bash">sudo systemctl reload caddy</code></pre>

您可以终止服务：
<pre><code class="cmd bash">sudo systemctl stop caddy</code></pre>

<aside class="advice">

不要通过终止服务来重载配置文件，终止服务会导致业务中断。请使用 reload 命令。

</aside>

Caddy 进程会在 `caddy` 身份下运行，而 `caddy` 用户的 `$HOME` 被设置为 `/var/lib/caddy`，这意味着：
- 默认的 [数据存储路径](/docs/conventions#data-directory)（保存证书或者其他状态信息）将会是 `/var/lib/caddy/.local/share/caddy`。
- 默认的 [配置存储路径](/docs/conventions#configuration-directory)（主要用于 `caddy-api` 服务的自动保存的 JSON 配置文件）将会是 `/var/lib/caddy/.config/caddy`。

<h3 id="local-https-with-systemd">
  本地 HTTPS
</h3>

当使用 Caddy 进行本机 HTTPS 环境的开发时，您必须使用一个 [主机名](/docs/caddyfile/concepts#addresses) 例如 `localhost` 或 `app.localhost`。这将会启用 Caddy 的本地 CA 颁发证书，参考 [本地 HTTPS](/docs/automatic-https#local-https)。

由于 Caddy 是在 `caddy` 用户下运行的服务，他无权将根 CA 证书安装到 system trust store，您需要运行 [`sudo caddy trust`](/docs/command-line#caddy-trust) 来进行授权。

如果您希望在其他设备上也使用这种 [`internal` issuer](/docs/caddyfile/directives/tls#internal) 访问 caddy，您还需要在这些设备上安装根 CA 证书。根 CA 证书位于 `/var/lib/caddy/.local/share/caddy/pki/authorities/local/root.crt`。现在很多浏览器都拥有自己的 trust store（而不再使用 system trust store），因此您可能还需要手动将证书安装到这些浏览器的 trust store。

### Overrides

修改服务文件的最佳方法是使用如下命令：
<pre><code class="cmd bash">sudo systemctl edit caddy</code></pre>

这将会使用您默认的文本编辑器打开一个空白文件，您可以在其中个红血或是添加任何服务配置，这个文件被称为 “drop-in” 文件。

例如，如果您需要定义服务配置文件中的 environment 变量，您可以这样：

```systemd
[Service]
Environment="CF_API_TOKEN=super-secret-cloudflare-tokenvalue"
```

相似的，如果您想通过一个独立的文件来维护 environment 变量（这被称为 envfile），您可以使用 [`EnvironmentFile`](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#EnvironmentFile=) 配置，就像这样：

```systemd
[Service]
EnvironmentFile=/etc/caddy/.env
```

又或者，您不再使用 Caddyfile 作为配置文件，转而使用 JSON 格式（注意，`Exec*` 类的配置在修改前 [必须先被重置为空字符串](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=)）：

```systemd
[Service]
ExecStart=
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/caddy.json
ExecReload=
ExecReload=/usr/bin/caddy reload --config /etc/caddy/caddy.json
```

又或者，您希望 caddy 能在每次挂掉的五秒后重启自己：

```systemd
[Service]
# Automatically restart caddy if it crashes except if the exit code was 1
RestartPreventExitStatus=1
Restart=on-failure
RestartSec=5s
```

修改配置后，保存文件，退出文本编辑器，重启服务即可获得效果：
<pre><code class="cmd bash">sudo systemctl restart caddy</code></pre>

<h3 id="selinux-considerations">
  SELinux 注意事项
</h3>

在启用了 SELinux 的系统中，您拥有两个可选方案：
1. 使用 [COPR repo](/docs/install#fedora-redhat-centos) 安装 Caddy，您的系统配置文件和 caddy 软件都会被正确的创建和标签标记（因此您可以跳过此章节）。但如果您希望运行一个自构建的 Caddy，您则需要按照下面说的方式给 caddy 软件打上标签。

2. 从 [此处](/download) 下载，或是通过 [`xcaddy`](https://github.com/caddyserver/xcaddy) 自行编译 caddy。无论是哪种方案，您都需要自己进行打标签的动作。

系统配置文件必须被标记为 `systemd_unit_file_t`，caddy 程序必须被标记为 `bin_t`，否则他们就无法运行。

在 `/etc/systemd/...` 路径下创建的文件会自动被添加 `systemd_unit_file_t` 标签，因此只要您将 `caddy.service` 文件创建在此处，之后按照 [手动安装](#manual-installation) 步骤操作即可。

您可以使用下述命令为 `caddy` 程序打 `bin_t` 标签：

<pre>
<code class="cmd bash">semanage fcontext -a -t bin_t /usr/bin/caddy && restorecon -Rv /usr/bin/caddy</code>
</pre>

<h2 id="windows-service">
  Windows 服务
</h2>

有两种方案可以让 Caddy 在 Windows 系统中作为服务运行：[sc.exe](#scexe) 和 [WinSW](#winsw)。

### sc.exe

创建服务:

<pre><code class="cmd bash">sc.exe create caddy start= auto binPath= "YOURPATH\caddy.exe run"</code></pre>

(记得将 `YOURPATH` 替换为您的 `caddy.exe` 的文件夹路径)

启动服务:

<pre><code class="cmd bash">sc.exe start caddy</code></pre>

停止服务:

<pre><code class="cmd bash">sc.exe stop caddy</code></pre>

### WinSW

按照下述说明将 Caddy 注册为 Windows 服务。

**您需要准备：**

- [下载](/download) 或 [自行构建](/docs/build) 的 `caddy.exe` 文件
- 最新的 [WinSW](https://github.com/winsw/winsw/releases/latest) 发布包中的可执行文件（下方的服务配置是基于 v2.x 版本编写的）

将所有的文件放入同一个文件夹中，在下方的例子里，我们使用的是 `C:\caddy`。

将 `WinSW-x64.exe` 重命名为 `caddy-service.exe`。

在文件夹中创建名为 `caddy-service.xml` 的配置文件。

```xml
<service>
  <id>caddy</id>
  <!-- Display name of the service -->
  <name>Caddy Web Server (powered by WinSW)</name>
  <!-- Service description -->
  <description>Caddy Web Server (https://caddyserver.com/)</description>
  <executable>%BASE%\caddy.exe</executable>
  <arguments>run</arguments>
  <log mode="roll-by-time">
    <pattern>yyyy-MM-dd</pattern>
  </log>
</service>
```

安装服务:
<pre><code class="cmd bash">caddy-service install</code></pre>

打开 Windows 服务页面，检查服务是否已经运行:
<pre><code class="cmd bash">services.msc</code></pre>

注意，Windows 服务无法重新加载，所以您可以直接让 caddy 重新加载：
<pre><code class="cmd bash">caddy reload</code></pre>

当然也可以使用 Windows 系统直接重启服务，例如通过任务管理器中的 “服务” 标签。

如果需要自定义更多服务属性，请参考 [WinSW documentation](https://github.com/winsw/winsw/tree/master#usage)。

## Docker Compose

通过 Docker 运行 caddy 最简单的方式就是使用 Docker compose。您可以参考 [Docker Hub](https://hub.docker.com/_/caddy) 中 Caddy 镜像的官方文档来了解更多信息。

<aside class="tip">

此处假设您在使用 [Docker Compose V2](https://docs.docker.com/compose/reference/)，其命令是 `docker compose`，而非 V1 版本的 `docker-compose`。

</aside>

<h3 id="setup">
  安装
</h3>

首先，创建一个 `compose.yml` 文件（或将下方的服务配置添加到您现有的文件中）：

```yaml
services:
  caddy:
    image: caddy:<version>
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - caddy_data:/data
      - caddy_config:/config

volumes:
  caddy_data:
  caddy_config:
```

检查并确认您的 `<version>` 处填写的是最新的版本号，您可以在 [Docker Hub](https://hub.docker.com/_/caddy) 中的 “Tags” 标签页下找到最新版本号。

配置文件的含义:

- 使用 `unless-stopped` 重启策略，可以确保您的机器重启后，Caddy 容器可以自动启动。
- 分别为 HTTP 和 HTTPS 协议绑定了 80 和 443 端口，为了 HTTP/3 添加了 `443/udp` 端口。
- 映射了 `Caddyfile` 作为您的配置文件。
- 从 `/srv` 映射了 `site` 路径，作为您站点的静态文件路径。
- 从 `/data` 和 `/config` 映射了路径，用来保存 [persist important information](/docs/conventions#file-locations)。

之后，根据您的 `compose.yml` 内容，创建名为 `Caddyfile` 的文件，按照业务需求进行 [配置](/docs/caddyfile/concepts)。

如果您有需要发布的静态资源文件，按照 yml 内容，您可以将他们放置在 `site/` 路径下，配置 Caddyfile 中的 [`root`](/docs/caddyfile/directives/root)，使用 `root * /srv`。如果您没有静态文件，您可以删除 `/srv` 的卷映射。

<aside class="tip">

如果您使用 Caddy 的 [反向代理](/docs/caddyfile/directives/reverse_proxy) 功能代理其他容器的业务，请注意在 Docker 的语境中，`localhost` 指的是 “本容器” 而非 “本设备”。因此，不要这样配置 `reverse_proxy localhost:8080`，而是可以这样 `reverse_proxy other-container:8080`。

</aside>

如果您需要使用自构建的 Caddy 版本或是带插件的版本，参考 [Docker build instructions](/docs/build#docker) 来创建一个自定义的 Docker 镜像。根据您的 `compose.yml` 创建 `Dockerfile`，随后将您 `compose.yml` 中的 `image:` 行替换为 `build: .`。

<h3 id="usage">
  使用
</h3>

启动容器：
<pre><code class="cmd bash">docker compose up -d</code></pre>

更改配置后重载 Caddy：
<pre><code class="cmd bash">docker compose exec -w /etc/caddy caddy caddy reload</code></pre>

查看 Caddy 的近 1000 条日志, 使用 `-f` 标志滚动刷新：
<pre><code class="cmd bash">docker compose logs caddy -n=1000 -f</code></pre>

<h3 id="local-https-with-docker">
  本地 HTTPS
</h3>

当在 Docker 下搭建本机 HTTPS 环境时，您需要一个类似于 `localhost` 或 `app.localhost` 的 [主机名](/docs/caddyfile/concepts#addresses)，这将会启用 Caddy 的本地 CA 颁发证书。这也意味着容器外部的 HTTP 客户端将不会信任 Caddy 提供的证书。为了解决这个问题，您需要将 Caddy 的根 CA 证书安装到您宿主机的 trust store：

<div x-data="{ os: $persist(defaultOS(['linux', 'mac', 'windows'], 'linux')) }" class="tabs">
<div class="tab-buttons">
	<button x-on:click="os = 'linux'" x-bind:class="{ active: os === 'linux' }">Linux</button>
	<button x-on:click="os = 'mac'" x-bind:class="{ active: os === 'mac' }">Mac</button>
	<button x-on:click="os = 'windows'" x-bind:class="{ active: os === 'windows' }">Windows</button>
</div>

<div x-show="os === 'linux'" class="tab bordered">

<pre><code class="cmd bash">docker compose cp \
    caddy:/data/caddy/pki/authorities/local/root.crt \
    /usr/local/share/ca-certificates/root.crt \
  && sudo update-ca-certificates</code></pre>

</div>

<div x-show="os === 'mac'" class="tab bordered">

<pre><code class="cmd bash">docker compose cp \
    caddy:/data/caddy/pki/authorities/local/root.crt \
    /tmp/root.crt \
  && sudo security add-trusted-cert -d -r trustRoot \
    -k /Library/Keychains/System.keychain /tmp/root.crt</code></pre>

</div>

<div x-show="os === 'windows'" class="tab bordered">

<pre><code class="cmd bash">docker compose cp \
    caddy:/data/caddy/pki/authorities/local/root.crt \
    %TEMP%/root.crt \
  && certutil -addstore -f "ROOT" %TEMP%/root.crt</code></pre>

</div>
</div>

现在很多浏览器都拥有自己的 trust store（而不再使用 system trust store），因此您可能还需要手动将证书安装到这些浏览器的 trust store，依然是使用上述命令中复制的 `root.crt` 文件。

- 对于 Firefox，在 Preferences > Privacy & Security > Certificates > View Certificates > Authorities > Import 中，选择 `root.crt` 文件。

- 对于 Chrome，在 Settings > Privacy and security > Security > Manage certificates > Authorities > Import 中，选择 `root.crt` 文件。