---
title: "命令行"
---

<h1 id="command-line">
  命令行
</h1>

Caddy 支持标准的 unix-like 命令行接口，基础用法是：

```
caddy <command> [<args...>]
```

`<尖括号>` 表示需要按照您的输入替换的内容。

`[方括号]` 表示选填的参数，`(圆括号)` 表示必填的参数。

省略号 `...` 表示一个或多个参数。

`--flags` 标志可能会拥有一个单字符的快捷方式例如 `-f`。

**查看帮助：`caddy`，`caddy help` 或 `man caddy` （如果安装了帮助手册）**

---

- **[caddy adapt](#caddy-adapt)**
	将配置文件转为 JSON 并输出

- **[caddy build-info](#caddy-build-info)**
  输出构建信息

- **[caddy completion](#caddy-completion)**
  生成命令补全脚本

- **[caddy environ](#caddy-environ)**
  输出环境变量

- **[caddy file-server](#caddy-file-server)**
  开箱即用文件服务器

- **[caddy file-server export-template](#caddy-file-server-export-template)**
	用于导出文件服务器模板的辅助命令

- **[caddy fmt](#caddy-fmt)**
	格式化 Caddyfile

- **[caddy hash-password](#caddy-hash-password)**
	对密码进行哈希处理并输出 base64

- **[caddy help](#caddy-help)**
  查看帮助

- **[caddy list-modules](#caddy-list-modules)**
  列表显示已安装的 Caddy 模组

- **[caddy manpage](#caddy-manpage)**
  生成帮助信息

- **[caddy reload](#caddy-reload)**
	让运行中的 caddy 实例重载配置

- **[caddy respond](#caddy-respond)**
	提供固定 HTTP 响应内容，用于开发或测试

- **[caddy reverse-proxy](#caddy-reverse-proxy)**
	开箱即用 HTTP(S) 反向代理服务器

- **[caddy run](#caddy-run)**
  在前台运行 Caddy

- **[caddy start](#caddy-start)**
  在后台运行 Caddy

- **[caddy stop](#caddy-stop)**
  停止 Caddy

- **[caddy storage export](#caddy-storage)**
  Exports the contents of the configured storage to a tarball

- **[caddy storage import](#caddy-storage)**
  Imports a previously exported tarball to the configured storage

- **[caddy trust](#caddy-trust)**
  Installs a certificate into local trust store(s)

- **[caddy untrust](#caddy-untrust)**
  Untrusts a certificate from local trust store(s)

- **[caddy upgrade](#caddy-upgrade)**
  Upgrades Caddy to the latest release

- **[caddy add-package](#caddy-add-package)**
  Upgrades Caddy to the latest release, with additional plugins added

- **[caddy remove-package](#caddy-remove-package)**
  Upgrades Caddy to the latest release, with some plugins removed

- **[caddy validate](#caddy-validate)**
  Tests whether a config file is valid

- **[caddy version](#caddy-version)**
  Prints the version

- **[Signals](#signals)**
  How Caddy handles signals

- **[Exit codes](#exit-codes)**
  Emitted when the Caddy process exits

<h2 id="subcommands">
  子命令
</h2>

### `caddy adapt`

<pre><code class="cmd bash">caddy adapt
	[-c, --config &lt;path&gt;]
	[-a, --adapter &lt;name&gt;]
	[-p, --pretty]
	[--validate]</code></pre>

将配置文件转换为 Caddy 的原生 JSON 格式并输出到 stdout，如遇见错误则输出到 stderr 并退出。

`--config` 指定配置文件的路径，如未填写，则使用当前路径下的 `Caddyfile`。

`--adapter` 指定配置适配器的类型，默认情况下是 `caddyfile`。

`--pretty` 为输出添加缩进，便于阅读。

`--validate` 会加载配置文件并检查是否可运行（但不会真的运行）。

请注意，语法上正确的配置文件可能会在运行时失败，例如：

```caddy
localhost

tls cert_notexist.pem key_notexist.pem
```

将他转换成 JSON：

<pre><code class="cmd bash">caddy adapt --config Caddyfile</code></pre>

输出是正确的，接下来加载并检查：

<pre><code class="cmd"><span class="bash">caddy adapt --config Caddyfile --validate</span>
adapt: validation: loading app modules: module name 'tls': provision tls: loading certificates: open cert_notexist.pem: no such file or directory
</code></pre>

虽然 Caddyfile 可以正确的适配为 JSON 格式，但配置中的证书和私钥却并不存在，因此检查失败。这种错误是在检查是否可运行的阶段发生的，因此 `--validate` 参数可以进行比语法更深入的配置文件检测。

<h4 id="example">
  示例
</h4>

将 Caddyfile 转换成适于人类阅读的 JSON 格式：

<pre><code class="cmd bash">caddy adapt --config /path/to/Caddyfile --pretty</code></pre>

### `caddy build-info`

<pre><code class="cmd bash">caddy build-info</code></pre>

输出 Go 语言提供的构建信息（主模块路径，包版本，模块替换）。

### `caddy completion`

<pre><code class="cmd bash">caddy completion [bash|zsh|fish|powershell]</code></pre>

生成命令补全脚本，让您可以在使用 `caddy` 命令后添加 tab 补全或是自动补全（或其他类似的功能，取决于您的终端）。

如需了解如何在您的终端环境中安装脚本，请使用 `caddy help completion` 或 `caddy completion -h`。

### `caddy environ`

<pre><code class="cmd bash">caddy environ</code></pre>

输出 caddy 所见的环境变量，在进行初始调试或是进程管理模块调试（例如 systemd）时非常有用。

### `caddy file-server`

<pre><code class="cmd bash">caddy file-server
	[-r, --root &lt;path&gt;]
	[--listen &lt;addr&gt;]
	[-d, --domain &lt;example.com&gt;]
	[-b, --browse]
	[--reveal-symlinks]
	[-t, --templates]
	[--access-log]
	[-v, --debug]
	[--no-compress]
	[-p, --precompressed]</code></pre>

开箱即用文件服务器。

`--root` 指定文件服务器根路径，默认为当前的运行路径。

`--listen` 指定监听端口，默认为 `:80`，当使用了 `--domain` 时则默认端口为 `:443`。

`--domain` 指定主机名或域名，之后只会向该主机名或域名提供服务。并且 Caddy 会尝试提供基于 HTTPS 的服务，因此请确保您的域名正确指向您的主机。默认端口号会被变更为 443。

`--browse` 当请求的路径不包含 index 文件时，使用列表显示路径中的内容。

`--reveal-symlinks` 会在列表中显示软链接的目标，需要先启用 `--browse`。

`--templates` 开启模板渲染支持。

`--access-log` 开启访问日志。

`--debug` 开启详细日志。

`--no-compress` 禁用压缩，默认情况下会开启 Zstandard 和 Gzip 压缩。

`--precompressed` 指定压缩格式用来搜索 sidecar 文件，可以多次重复配置以指定多种文件格式。查看 [file_server 指令](/docs/caddyfile/directives/file_server#precompressed) 了解更多信息。

`file-server` 命令默认禁用管理 API，这样可以更加简单的在本地开发环境中部署多个文件服务器实例。

#### `caddy file-server export-template`

<pre><code class="cmd bash">caddy file-server export-template</code></pre>

将默认的文件列表模板输出到 stdout。

### `caddy fmt`

<pre><code class="cmd bash">caddy fmt [&lt;path&gt;]
	[-w, --overwrite]
	[-d, --diff]</code></pre>

格式化 Caddyfile 并输出到 stdout，如果添加了 `--overwrite` 参数，则会直接覆盖源文件。如果源文件需要被格式化（格式化后结果和源文件不同），返回错误码 `1`。

`<path>` 指定 Caddyfile 的路径，如果为 `-` 则从 stdin 接收输入，如果为空则默认使用当前路径下的 Caddyfile。

`--overwrite` 会将格式化结果覆盖到源文件而非输出到终端，如果输入并非一个普通文件，则该标志无效。

`--diff` 会比较输入和输出间的差异，使用 `-` 和 `+` 开头的行来描述这些差异，未被修改的行则使用两空格宽度的缩进对齐，注意这个该命令只用来查看差异，并不会真正的格式化您的文件。

### `caddy hash-password`

<pre><code class="cmd bash">caddy hash-password
	[-p, --plaintext &lt;password&gt;]
	[-a, --algorithm &lt;name&gt;]</code></pre>

一个用来对明文密码进行哈希处理的快捷方法。哈希的结果值将会输出在 stdout，您可以直接用在您的 Caddy 配置中。

### `caddy help`

<pre><code class="cmd bash">caddy help [&lt;command&gt;]</code></pre>

输出帮助信息，可以指定需要查看的子命令。

### `caddy list-modules`

<pre><code class="cmd bash">caddy list-modules
	[--packages]
	[--versions]
	[-s, --skip-standard]</code></pre>

输出已经安装的 Caddy 模组，可以选择指定包显示或是显示他们相关的 Go 模组信息。

在一些脚本化环境中，输出标准模组可能会有些冗余，您可以使用 `--skip-standard` 来跳过标准模组的输出。

注意：由于 [Go 中的一个 bug](https://github.com/golang/go/issues/29228)，当前的 Caddy 只有在不作为 main 模组编译时才能显示他的版本信息。您可使用 [xcaddy](/docs/build#xcaddy) 来编译 Caddy。

### `caddy manpage`

<pre><code class="cmd bash">caddy manpage
	(-o, --directory &lt;path&gt;)</code></pre>

通过 Caddy 命令生成 手册/文档 页，并放入指定路径，输出的内容可以被 `man` 命令读取。

`--directory` （必填）参数用来指定输出文件的路径，如果路径不存在则会创建。

生成帮助文件后，需要安装到操作系统中。安装的操作因平台而已，但对于典型的 Linux 系统，大概如此：

<pre><code class="cmd"><b>$ caddy manpage --directory man
$ gzip -r man/
$ sudo cp man/* /usr/share/man/man8/
$ sudo mandb
</b></code></pre>

随后您就可以使用 `man caddy` （或是 `man caddy-*` 用来查询子命令）在您的终端中查阅文档。

手册页和我们的网站是分别维护的，我们的网站会有更加全面的文档，而且更新频率更高。

### `caddy reload`

<pre><code class="cmd bash">caddy reload
	[-c, --config &lt;path&gt;]
	[-a, --adapter &lt;name&gt;]
	[--address &lt;interface&gt;]
	[-f, --force]</code></pre>

让正在运行的 Caddy 实例重新加载配置，该命令的实际效果和将配置文件 POST 到 [/load endpoint](/docs/api#post-load) 接口等价，但是该命令对于使用配置文件的工作流更加方便。相比于 `stop`，`start` 和 `run` 命令，该命令是更加正确的，语义更加明确的重载配置的方式。

由于该命令使用 API，管理接口必须处于可用状态。

`--config` 指定使用的配置文件，如果为 `-`，则通过 stdin 读取配置，如果未指定，则尝试读取当前路径下的 `Caddyfile` 文件并使用 `caddyfile` 配置适配器，如果文件不存在，则报错。

`--adapter` 指定配置适配器。

`--address` 当管理端口并非默认端口，且与之前配置文件中提供的不同时，需要用此参数指定管理端口地址。注意此处只能使用 TCP 地址。

`--force` 即使配置文件和当前运行环境完全相同，使用该参数也可以进行一次强制重新加载。可以用于强制让 Caddy 重新加载模块或是一些其他的周边内容，例如：重新手动加载 TLS 证书。

### `caddy respond`

<pre><code class="cmd bash">caddy respond
	[-s, --status &lt;code&gt;]
	[-H, --header "&lt;Field&gt;: &lt;value&gt;"]
	[-b, --body &lt;content&gt;]
	[-l, --listen &lt;addr&gt;]
	[-v, --debug]
	[--access-log]
	[&lt;status|body&gt;]</code></pre>

开启一个或多个固定返回的 HTTP 服务器，用于开发、测试或其他场景。对于调试 HTTP 客户端、脚本或是负载均衡器很有用。

`--status` HTTP 返回状态码。

`--header` 使用 `Field: value` 格式在返回中添加 HTTP 头，可以重复多次以添加多个头。

`--body` 指定响应体，或者可以从 stdin 输入响应体。

`--listen` 监听地址，可以是 Caddy 认可的任何 [network address](/docs/conventions#network-addresses)，或者指定端口范围来同时开启多个服务。

`--debug` 开启详细 debug 日志。

`--access-log` 开启 接入/请求 日志。

默认情况下，此命令监听一个随机的端口号，并且使用状态码为 200 的空响应处理一切请求。可以使用 `--listen` 标志指定监听地址，而且程序总是会在运行时把监听地址输出到 stdout，如果监听地址包含了一个端口范围，那么会同时启动多个服务。

最后，如果在命令尾部输入了未命名参数，当参数是三位数字时则会被视为状态码（与 `--status` 等价），否则参数会被视为响应体（与 `--body` 等价）。当然，在指明 `--status` 或 `--body` 标志的情况下不会考虑默认参数。

可以通过三种方式提供响应体：通过标志、通过尾部的未命名参数或是通过 stdin（如果标志和参数都没有被设置）管道。body 中支持有限的 [模板](https://pkg.go.dev/text/template) 能力，包括：

变量 | 描述
---------|-------------
`.N`       | 服务编号
`.Port`    | 监听端口
`.Address` | 监听地址

<h4 id="example">
  示例
</h4>

在随机端口返回空内容的 200 状态响应：
<pre><code class="cmd bash">caddy respond</code></pre>

带有响应体的响应：
<pre><code class="cmd bash">caddy respond "Hello, world!"</code></pre>

创建多个服务并使用模板：
<pre><code class="cmd"><b>$ caddy respond --listen :2000-2004 "{{printf "I'm server {{.N}} on port {{.Port}}"}}"</b>

Server address: [::]:2000
Server address: [::]:2001
Server address: [::]:2002
Server address: [::]:2003
Server address: [::]:2004

<b>$ curl 127.0.0.1:2002</b>
I'm server 2 on port 2002</code></pre>

通过管道导入一个运维页面：
<pre><code class="cmd bash">cat maintenance.html | caddy respond \
	--listen :80 \
	--status 503 \
	--header "Content-Type: text/html"</code></pre>

### `caddy reverse-proxy`

<pre><code class="cmd bash">caddy reverse-proxy
	[-f, --from &lt;addr&gt;]
	(-t, --to &lt;addr&gt;)
	[-H, --header-up "&lt;Field&gt;: &lt;value&gt;"]
	[-d, --header-down "&lt;Field&gt;: &lt;value&gt;"]
	[-c, --change-host-header]
	[-r, --disable-redirects]
	[-i, --internal-certs]
	[-v, --debug]
	[--access-log]
	[--insecure]</code></pre>

开箱即用反向代理服务器，用于快速部署、demo、开发环境等。

从 `--from` 地址接收 HTTP(S) 请求，并转发到 `--to` 地址，可以重复使用 `--to` 标志来设置多个后端服务，至少需要有一个后端服务，`--to` 地址可以包含端口范围用来快捷匹配一批端口。

如果 `--from` 地址提供了主机名或域名，那么默认会提供 HTTPS 服务，而 `--to` 地址默认使用 HTTP 服务。

如果 `--from` 标志指明了 HTTP `http://`，那么 `--from` 参数提供 HTTP 服务。

当使用 HTTPS 服务时：
  - `--disable-redirects` 可用于避免绑定到 HTTP 端口，仅使用 HTTPS 端口。

  - `--internal-certs` 可以强制指定使用内部 CA 发放证书，而非尝试获取公共信任的证书。

进行代理时：
  - `--header-up` 可以用来设置发送给后端的 HTTP 头。
  
  - `--header-down` 可以用来设置返回给客户端的 HTTP 头。
  
  - `--change-host-header` 将请求中的 Host 头更改为后端地址，而非直接使用入站的 Host 头。

    等同于下述配置 `--header-up "Host: {http.reverse_proxy.upstream.hostport}"`
  
  - `--insecure` 关闭到后端的 TLS 验证。警告：您需要清楚的了解此操作导致的安全影响。
  
  - `--debug` 开启详细日志。

此命令会关闭管理 API，便于在同一个机器上开展此命令的多个实例。

### `caddy run`

<pre><code class="cmd bash">caddy run
	[-c, --config &lt;path&gt;]
	[-a, --adapter &lt;name&gt;]
	[--pidfile &lt;file&gt;]
	[-e, --environ]
	[--envfile &lt;file&gt;]
	[-r, --resume]
	[-w, --watch]</code></pre>

运行 Caddy 并阻塞终端（“守护进程” 模式）。

`--config` 用于指定配置文件，如果值为 `-`，则从 stdin 接收配置，如果值不填，Caddy 会在空配置的状态下运行并使用 [admin API endpoints](/docs/api) 的默认设置，这样可用于给 Caddy 提供新的配置。作为一个特例，如果当前的运行路径下存在名为 "Caddyfile" 的文件且 `caddyfile` 配置适配器已经安装（默认情况下就是安装的），那么在不使用 `--config` 标志时 Caddy 会加载本地的 Caddyfile 作为配置。

`--adapter` 用于指定加载配置文件时使用适配器的名称，如果 `--config` 指定的文件以 "Caddyfile" 作为开头，则可以不添加此标志并默认使用 `caddyfile` 适配器，否则必须使用此标志指定适配器将非 Caddy 原生 JSON 格式的配置进行转化。运行中的警告会被输出，但是请注意只要没有错误，即便有警告，配置文件也会立即生效。如果您想在运用配置文件前先查看适配结果，您可以使用 [`caddy adapt`](#caddy-adapt) 子命令先进行验证。

`--pidfile` 将 PID 写入指定文件。

`--environ` 在运行前先输出环境变量，本命令的输出和 `caddy environ` 相同，区别在于本命令会在输出后运行 Caddy。

`--envfile` 用于在指定文件中加载环境变量。文件使用 `KEY=VALUE` 格式，支持 `#` 开头的行注释，key 可以使用 `export` 作为前缀，value 可以被双引号包裹（value 中的双引号需要转义），支持多行 value。

`--resume` 使用上次自动保存的配置运行 Caddy，会覆盖 `--config` 参数（如存在）。使用此参数可以在设备重启或进程重启后仍然保持配置一致，非常适用于使用 [API](/docs/api) 管理配置的环境。

`--watch` 监听配置文件的变化，并且在配置文件变化后重新加载配置。⚠️ 本功能仅被设计用于开发环境！

<aside class="advice">

请不要在生产环境中通过重启 Caddy 的方式修改配置文件，这回导致服务中断！（您可能觉得我们强调过度，但是您真的不知道我们收到了多少关于此事的抱怨。）记得用 [`caddy reload`] 来重载配置。

</aside>

### `caddy start`

<pre><code class="cmd bash">caddy start
	[-c, --config &lt;path&gt;]
	[-a, --adapter &lt;name&gt;]
	[--envfile &lt;file&gt;]
	[--pidfile &lt;file&gt;]
	[-w, --watch]</code></code></pre>

与 [`caddy run`](#caddy-run) 相同，但是将进程运行在后端，此命令只会在程序启动过程中阻塞终端，一旦程序成功运行或失败退出，都会返回。

注意：`--config` 标志 _不_ 再支持使用 `-` 值来接收 stdin 输入。

不推荐在可以将 Caddy 作为服务运行的系统上或是在 Windows 系统上使用此命令。在 Windows 系统中，此命令创建的子进程依然会和终端绑定（作为终端的子进程），因此当终端关闭时子进程也会被关闭，这可能会是使用者困扰。我们推荐 [将 Caddy 配置为服务](/docs/zh/running)。

进程启动后，您可以使用 [`caddy stop`](#caddy-stop) 命令或是 [`POST /stop`](/docs/api#post-stop) 接口来关闭后台进程。

### `caddy stop`

<pre><code class="cmd bash">caddy stop
	[--address &lt;interface&gt;]
	[-c, --config &lt;path&gt; [-a, --adapter &lt;name&gt;]]</code></pre>

<aside class="tip">

Caddy 的配置重载和停止（或重启）操作无关。**不要在生产环境使用 停止/重启 操作来重载配置。**应使用 [`caddy reload`] 命令。

</aside>

正常中止运行中的 Caddy 进程（除了停止命令的进程）并退出，此命令使用管理 API 中的 [`POST /stop`](/docs/api#post-stop) 接口执行中止。

可以使用 `--address` 标志定义管理 API 的监听地址，或根据 `--config` 标志中的配置读取管理 API 的监听地址（如果管理 API 没有使用默认的监听地址的话）。

如果您想要删除 Caddy 当前的配置但是并不想停止进程，您可以使用 [`caddy reload`] 加载一个空配置，或是使用管理 API 中的 [`DELETE /config/`](/docs/api#delete-configpath) 接口。

### `caddy storage`

<i>⚠️ 实验性功能</i>

Allows export and import of the contents of Caddy's configured data storage.

This is useful when needing to transition from one [storage module](/docs/json/storage/) to another, by exporting from your old one, updating your config, then importing into the new one.

The following command can be used to copy the storage between different modules in one shot, using old and new configs, piping the export command's output into the import command.

```
$ caddy storage export -c Caddyfile.old -o- |
  caddy storage import -c Caddyfile.new -i-
```

<aside class="advice">

Please note that when using [filesystem storage](/docs/conventions#data-directory), you must run the export command as the same user that Caddy normally runs as, otherwise the wrong storage location may be used.

For example, when running Caddy as a [systemd service](/docs/running#linux-service), it will run as the `caddy` user, so you should run the export or import commands as that user. This can typically be done with `sudo -u caddy <command>`.

</aside>


#### `caddy storage export`

<pre><code class="cmd bash">caddy storage export
	-c, --config &lt;path&gt;
	[-o, --output &lt;path&gt;]</code></pre>

`--config` is the config file load. This is required, so that the correct storage module is connected to.

`--output` is the filename to write the tarball. If `-`, the output is written to stdout.



#### `caddy storage import`

<pre><code class="cmd bash">caddy storage import
	-c, --config &lt;path&gt;
	-i, --input &lt;path&gt;</code></pre>

`--config` is the config file load. This is required, so that the correct storage module is connected to.

`--input` is the filename of the tarball to read from. If `-`, the input is read from stdin.


### `caddy trust`

<pre><code class="cmd bash">caddy trust
	[--ca &lt;id&gt;]
	[--address &lt;interface&gt;]
	[-c, --config &lt;path&gt; [-a, --adapter &lt;name&gt;]]</code></pre>

Installs a root certificate for a CA managed by Caddy's [PKI app](/docs/json/apps/pki/) into local trust stores. 

Caddy will attempt to install its root certificates into the local trust stores automatically when they are first generated, but it might fail if Caddy doesn't have the appropriate permissions to write to the trust store. This command is necessary to pre-install the certificates before using them, if the server process runs as an unprivileged user (such as via systemd). You may need to run this command with `sudo` to unix systems.

By default, this command installs the root certificate for Caddy's default CA (i.e. "local"). You may specify the ID of another CA with the `--ca` flag.

This command will attempt to connect to Caddy's [admin API](/docs/api) to fetch the root certificate, using the [`GET /pki/ca/<id>/certificates`](/docs/api#get-pkicaltidgtcertificates) endpoint. You may explicitly specify the `--address`, or use the `--config` flag to load the admin address from your config, if the running instance's admin API is not using the default listen address.

You may also use the `caddy` binary with this command to install certificates on other machines in your network, if the admin API is made accessible to other machines -- be careful if doing this, to not expose the admin API to untrusted clients.


### `caddy untrust`

<pre><code class="cmd bash">caddy untrust
	[-p, --cert &lt;path&gt;]
	[--ca &lt;id&gt;]
	[--address &lt;interface&gt;]
	[-c, --config &lt;path&gt; [-a, --adapter &lt;name&gt;]]</code></pre>

Untrusts a root certificate from the local trust store(s).

This command uninstalls trust; it does not necessarily delete the root certificate from trust stores entirely. Thus, repeatedly trusting and untrusting new certificates can fill up trust databases.

This command does not delete or modify certificate files from Caddy's configured storage.

This command can be used in one of two ways:
- By specifying a direct path to the root certificate to untrust with the `--cert` flag.
- By fetching the root certificate from the [admin API](/docs/api) using the [`GET /pki/ca/<id>/certificates`](/docs/api#get-pkicaidcertificates) endpoint. This is the default behaviour if no flags are given.

If the admin API is used, then the CA ID defaults to "local". You may specify the ID of another CA with the `--ca` flag. You may explicitly specify the `--address`, or use the `--config` flag to load the admin address from your config, if the running instance's admin API is not using the default listen address.


### `caddy upgrade`

<i>⚠️ Experimental</i>

<pre><code class="cmd bash">caddy upgrade
	[-k, --keep-backup]</code></pre>

Replaces the current Caddy binary with the latest version from [our download page](/download) with the same modules installed, including all third-party plugins that are registered on the Caddy website.

Upgrades do not interrupt running servers; currently, the command only replaces the binary on disk. This might change in the future if we can figure out a good way to do it.

The upgrade process is fault tolerant; the current binary is backed up first (copied beside the current one) and automatically restored if anything goes wrong. If you wish to keep the backup after the upgrade process is complete, you may use the `--keep-backup` option.

This command may require elevated privileges if your user does not have permission to write to the executable file.



### `caddy add-package`

<i>⚠️ Experimental</i>

<pre><code class="cmd bash">caddy add-package &lt;packages...&gt;
	[-k, --keep-backup]</code></pre>

Similarly to `caddy upgrade`, replaces the current Caddy binary with the latest version with the same modules installed, _plus_ the packages listed as arguments included in the new binary. Find the list of packages you can install from [our download page](/download). Each argument should be the full package name.

For example:

<pre><code class="cmd bash">caddy add-package github.com/caddy-dns/cloudflare</code></pre>



### `caddy remove-package`

<i>⚠️ Experimental</i>

<pre><code class="cmd bash">caddy remove-package &lt;packages...&gt;
	[-k, --keep-backup]</code></pre>

Similarly to `caddy upgrade`, replaces the current Caddy binary with the latest version with the same modules installed, but _without_ the packages listed as arguments, if they existed in the current binary. Run `caddy list-modules --packages` to see the list of package names of non-standard modules included in the current binary.



### `caddy validate`

<pre><code class="cmd bash">caddy validate
	[-c, --config &lt;path&gt;]
	[-a, --adapter &lt;name&gt;]
	[--envfile &lt;file&gt;]</code></pre>

Validates a configuration file, then exits. This command deserializes the config, then loads and provisions all of its modules as if to start the config, but the config is not actually started. This exposes errors in a configuration that arise during loading or provisioning phases and is a stronger error check than merely serializing a config as JSON.

`--config` is the config file to validate. If `-`, the config is read from stdin. Default is the `Caddyfile` in the current directory, if any.

`--adapter` is the name of the config adapter to use, if the config file is not in Caddy's native JSON format. If the config file starts with `Caddyfile`, the `caddyfile` adapter is used by default.

`--envfile` loads environment variables from the specified file, in `KEY=VALUE` format. Comments starting with `#` are supported; keys may be prefixed with `export`; values may be double-quoted (double-quotes within can be escaped); multi-line values are supported.



### `caddy version`
<pre><code class="cmd bash">caddy version</code></pre>

Prints the version and exits.



## Signals

Caddy traps certain signals and ignores others. Signals can initiate specific process behavior.

Signal | Behavior
-------|----------
`SIGINT` | Graceful exit. Send signal again to force exit immediately.
`SIGQUIT` | Quits Caddy immediately, but still cleans up locks in storage because it is important.
`SIGTERM` | Graceful exit.
`SIGUSR1` | Ignored. For config updates, use the `caddy reload` command or [the API](/docs/api).
`SIGUSR2` | Ignored.
`SIGHUP` | Ignored.

A graceful exit means that new connections are no longer accepted, and existing connections will be drained before the socket is closed. A grace period may apply (and is configurable). Once the grace period is up, connections will be forcefully terminated. Locks in storage and other resources that individual modules need to release are cleaned up during a graceful shutdown.

## Exit codes

Caddy returns a code when the process exits:

Code | Meaning
-----|---------
`0` | Normal exit.
`1` | Failed startup. **Do not automatically restart the process; it will likely error again unless changes are made.**
`2` | Forced quit. Caddy was forced to exit without cleaning up resources.
`3` | Failed quit. Caddy exited with some errors during cleanup.

In bash, you can get the exit code of the last command with `echo $?`.
