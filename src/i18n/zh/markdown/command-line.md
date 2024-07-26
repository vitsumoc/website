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
	将指定配置的存储内容导出为压缩包

- **[caddy storage import](#caddy-storage)**
	使用导出的压缩包导入到指定配置的存储中

- **[caddy trust](#caddy-trust)**
	向本地信任仓库安装根证书

- **[caddy untrust](#caddy-untrust)**
	不再信任本地信任仓库中的证书

- **[caddy upgrade](#caddy-upgrade)**
	将 Caddy 更新到最新

- **[caddy add-package](#caddy-add-package)**
  将 Caddy 更新到最新，并添加一些插件

- **[caddy remove-package](#caddy-remove-package)**
	将 Caddy 更新到最新，并移除一些插件

- **[caddy validate](#caddy-validate)**
  验证配置文件是否可用

- **[caddy version](#caddy-version)**
  输出版本信息

- **[Signals](#signals)**
  Caddy 如何响应信号

- **[Exit codes](#exit-codes)**
  Caddy 进程退出时的返回值

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

<i>⚠️ 实验性</i>

支持导入或导出 Caddy 存储中的内容。

通过从旧的存储执行导出，再到新的存储执行导入，此指令用于在不同的 [存储模块](/docs/json/storage/) 间切换。

下方的命令可以从旧的配置文件中读取数据，提取存储内容到 stdout，之后通过管道发送到新的配置文件中定义的存储区域。

```
$ caddy storage export -c Caddyfile.old -o- |
  caddy storage import -c Caddyfile.new -i-
```

<aside class="advice">

请注意，在使用 [文件系统存储](/docs/conventions#data-directory) 时，您必须使用和 Caddy 运行时相同的用户去进行导出操作，否则您可能会引用到错误的存储地址。

例如，当您的 Caddy [作为系统服务运行](/docs/zh/running#linux-service) 时，一般情况下他运行在 `caddy` 用户下，因此您需要在 `caddy` 用户下使用导出命令，比如：`sudo -u caddy <command>`。

</aside>

#### `caddy storage export`

<pre><code class="cmd bash">caddy storage export
	-c, --config &lt;path&gt;
	[-o, --output &lt;path&gt;]</code></pre>

`--config` 选择加载的配置文件，必填，用以指定导出哪个存储中的内容。

`--output` 保存压缩包的文件名，如果值为 `-`，则输出到 stdout。

#### `caddy storage import`

<pre><code class="cmd bash">caddy storage import
	-c, --config &lt;path&gt;
	-i, --input &lt;path&gt;</code></pre>

`--config` 选择加载的配置文件，必填，用以指定导入到哪个存储中。

`--input` 需导入压缩包的文件名，如果值为 `-`，则使用 stdin 的输入。


### `caddy trust`

<pre><code class="cmd bash">caddy trust
	[--ca &lt;id&gt;]
	[--address &lt;interface&gt;]
	[-c, --config &lt;path&gt; [-a, --adapter &lt;name&gt;]]</code></pre>

将 Caddy 的 [PKI app](/docs/json/apps/pki/) 管理的 CA 颁布的根证书安装到本机信任仓库。

Caddy 会在生成根证书时尝试将其安装到本机信任仓库，但是如果权限不足，安装可能失败。如果您在没有权限的情况下安装运行 Caddy，导致证书没有自动安装的话，您可以使用此命令安装证书（通常加上 `sudo` 授权）。

默认情况下，此命令安装的根证书将使用 Caddy 默认的 CA（ID 为 “local”），您也可以使用 `--ca` 标志改为其他 CA。

此命令将尝试使用 Caddy 的 [管理 API](/docs/api) 中的 [`GET /pki/ca/<id>/certificates`](/docs/api#get-pkicaltidgtcertificates) 接口来获得根证书。如果您的管理 API 没有运行在默认地址，您可以通过 `--address` 显式指定地址或是通过 `--config` 指定配置文件，从而使用配置文件中的地址。

您也可以让网络中的其他设备调用上述接口，获得 Caddy 颁发的根证书——小心，不要将管理 API 暴露给不受信任的设备。

### `caddy untrust`

<pre><code class="cmd bash">caddy untrust
	[-p, --cert &lt;path&gt;]
	[--ca &lt;id&gt;]
	[--address &lt;interface&gt;]
	[-c, --config &lt;path&gt; [-a, --adapter &lt;name&gt;]]</code></pre>

不再信任本地信任仓库中的某个根证书。

此命令解除对某个证书的信任，而非删除证书数据。因此，反复生成证书和取消信任可能填满信任数据库。

此命令并不会删除或改变 Caddy 配置的存储中的证书文件。

此命令可以使用以下方式调用： 
- 通过 `--cert` 标志指定路径，直接取消对某个根证书的信任。
- 如果未使用标志，则通过 [管理 API](/docs/api) 中的 [`GET /pki/ca/<id>/certificates`](/docs/api#get-pkicaidcertificates) 接口获取根证书。

如果使用了管理 API，则 CA ID 默认为 “local”，您可以通过 `--ca` 标志指定其他 CA ID。如果您的管理 API 没有运行在默认地址，您可以通过 `--address` 显式指定地址或是通过 `--config` 指定配置文件，从而使用配置文件中的地址。

### `caddy upgrade`

<i>⚠️ 实验性</i>

<pre><code class="cmd bash">caddy upgrade
	[-k, --keep-backup]</code></pre>

从 [下载页](/download) 获取安装最新版本的 Caddy 替换当前运行的程序，包括相同的模块选择、包括所有已经注册到 Caddy 网站的三方插件。

升级动作并不会打断服务的运行，目前我们只是下载并替换硬盘中的二进制文件。在未来的某一天也许我们会使用更好的解决方案。

更新失败时会自动回滚，当前的程序软件会在更新前自动备份，如果更新时发生错误则会回滚。如果您希望在更新后仍然保留更新前的版本，您可以使用 `--keep-backup` 选项。

如果您的当前用户不具备对 Caddy 可执行文件的读写权限，使用此命令前您需要进行授权。

### `caddy add-package`

<i>⚠️ 实验性</i>

<pre><code class="cmd bash">caddy add-package &lt;packages...&gt;
	[-k, --keep-backup]</code></pre>

与 `caddy upgrade` 相似，获取安装最新版本的 Caddy 替换当前运行的程序，同时会将参数列表中的包 _添加_ 到 Caddy 中。您可以在我们的 [下载页](/download) 找到可以安装的包，必须使用完整的包名。

例如：

<pre><code class="cmd bash">caddy add-package github.com/caddy-dns/cloudflare</code></pre>

### `caddy remove-package`

<i>⚠️ 实验性</i>

<pre><code class="cmd bash">caddy remove-package &lt;packages...&gt;
	[-k, --keep-backup]</code></pre>

与 `caddy upgrade` 相似，获取安装最新版本的 Caddy 替换当前运行的程序，同时会在新的 Caddy 中 _移除_ 参数列表中的包（如果他们当前被使用了的话）。运行 `caddy list-modules --packages` 可以查看当前已安装的非基础包。

### `caddy validate`

<pre><code class="cmd bash">caddy validate
	[-c, --config &lt;path&gt;]
	[-a, --adapter &lt;name&gt;]
	[--envfile &lt;file&gt;]</code></pre>

验证配置文件，随后退出。此命令会反序列化配置，随后向启动服务一样加载所有的模块，但并不会真的启动服务。此命令可以检查配置在读取和加载过程中的错误，检查程度比将配置文件转换成 JSON 格式要更加深入。

`--config` 指定被验证的配置文件，如值为 `-`，则从 stdin 接收配置文件，默认使用当前路径下的 `Caddyfile`。

`--adapter` 指定使用的配置适配器（当配置文件不为 Caddy 的原生 JSON 格式时），如果配置文件名称以 `Caddyfile` 开头，那么默认会使用 `caddyfile` 适配器。

`--envfile` 从指定文件中加载环境变量，使用 `KEY=VALUE` 格式。支持 `#` 开头的注释，键可以有 `export` 前缀，值可以被双引号包裹，值使用双引号需要转义，支持多行值。

### `caddy version`
<pre><code class="cmd bash">caddy version</code></pre>

输出版本信息并退出。

## Signals

Caddy 捕获这些信号并忽略其他信号，信号可以影响进程的行为。

信号 | 行为
-------|----------
`SIGINT` | 正常退出，再次接收到此信号 Caddy 则会强制退出。
`SIGQUIT` | 立刻退出 Caddy，但仍会清理存储中的锁（因为这很重要）。
`SIGTERM` | 正常退出。
`SIGUSR1` | 忽略，如果需要重载配置，请使用 `caddy reload` 或 [API](/docs/api)。 
`SIGUSR2` | 忽略。
`SIGHUP` | 忽略。

正常关闭意味着不再接收新的链接，现有的链接也会在一个（可配置的）宽限期内正常关闭。一旦宽限期到时，链接会被立刻强制关闭。存储中的锁和其他模块中需要释放的资源会在正常关闭期间被清理。

## Exit codes

Caddy 在进程退出时的返回码：

返回码 | 含义
-----|---------
`0` | 正常退出
`1` | 启动失败。 **不要尝试自动重启，如果没有任何改变的话，自动重启的结果依然会是启动失败。**
`2` | 强制退出，Caddy 在没有进行资源清理的情况下关闭。
`3` | 失败退出，Caddy 在清理期间因为一些错误而退出。

在终端中，您可以通过 `echo $?` 查看上一个命令的返回码。
