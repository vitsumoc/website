---
title: "API"
---

# API

您可以使用 Caddy 的管理服务进行配置工作，Caddy 的管理服务提供了一套 [REST <img src="/old/resources/images/external-link.svg" class="external-link">](https://en.wikipedia.org/wiki/Representational_state_transfer) 风格的 API，您也可以在您的 Caddy 配置中 [配置管理服务](/docs/json/admin/)。

**默认地址为：`localhost:2019`**

默认地址可以被环境变量 `CADDY_ADMIN` 修改，有些安装方式可能会设置环境变量来改变默认地址。Caddy 配置中的地址的优先级会高于默认地址。

<aside class="tip">
	如果您在服务器上运行不信任的代码（糟糕 😬），请注意通过进程隔离保护您的管理端口，您需要为一些易受攻击的应用打补丁，或者将管理端口配置到需要授权访问的端口号上。
</aside>

在做出任何配置变更后，最新的配置会被保存到硬盘上（除非 [此功能被关闭](/docs/json/admin/config/)），您可以通过 [`caddy run --resume`](/docs/command-line#caddy-run) 启动 Caddy 并加载最新的配置，此功能保证了在类似电源中断的故障中配置不会丢失。

想要学习和理解 Caddy 的 API，您可以从 [API 教程](/docs/zh/api-tutorial) 开始，如果您赶时间，您可以试试 [快速了解 API](/docs/quick-starts/api)。

---

- **[POST /load](#post-load)**
  Sets or replaces the active configuration

- **[POST /stop](#post-stop)**
  Stops the active configuration and exits the process

- **[GET /config/[path]](#get-configpath)**
  Exports the config at the named path

- **[POST /config/[path]](#post-configpath)**
  Sets or replaces object; appends to array
  
- **[PUT /config/[path]](#put-configpath)**
  Creates new object; inserts into array

- **[PATCH /config/[path]](#patch-configpath)**
  Replaces an existing object or array element

- **[DELETE /config/[path]](#delete-configpath)**
  Deletes the value at the named path

- **[Using `@id` in JSON](#using-id-in-json)**
  Easily traverse into the config structure

- **[Concurrent config changes](#concurrent-config-changes)**
  Avoid collisions when making unsynchronized changes to config

- **[POST /adapt](#post-adapt)**
  Adapts a configuration to JSON without running it

- **[GET /pki/ca/&lt;id&gt;](#get-pkicaltidgt)**
  Returns information about a particular [PKI app](/docs/json/apps/pki/) CA

- **[GET /pki/ca/&lt;id&gt;/certificates](#get-pkicaltidgtcertificates)**
  Returns the certificate chain of a particular [PKI app](/docs/json/apps/pki/) CA

- **[GET /reverse_proxy/upstreams](#get-reverse-proxyupstreams)**
  Returns the current status of the configured proxy upstreams


## POST /load

为 Caddy 添加配置，并覆盖之前的配置，在 Caddy 进行重载的过程中此端口会堵塞。配置变更是一个轻量、高效、无中断时间的动作。如果新的配置加载失败，那么会继续使用原有的配置，不会有中断时间。

通过配置适配器，此接口支持多种不同的配置格式。请求头中的 Content-Type 表示了请求体中配置内容应有的格式。通常来说，Content-Type 的值使用 `application/json`，请求体则使用 Caddy 的原生配置格式。如果需要使用其他格式，则修改 `/` 前后的内容用来指定相应的适配器。例如：如果提交内容为 Caddyfile， Content-Type 的值需要被指定为 `text/caddyfile`，而如果使用 JSON5，值应为 `application/json5`，等等。

如果新的配置内容和现有的配置相同，则不会发生重载。如需要强制重载，请在请求头中设置 `Cache-Control: must-revalidate`。

### Examples

加载新配置：

<pre><code class="cmd bash">curl "http://localhost:2019/load" \
	-H "Content-Type: application/json" \
	-d @caddy.json</code></pre>

注意：curl 中的 `-d` 标志会删除换行符，如果您的配置文件格式对换行敏感（例如 Caddyfile），请使用 `--data-binary` 代替：

<pre><code class="cmd bash">curl "http://localhost:2019/load" \
	-H "Content-Type: text/caddyfile" \
	--data-binary @Caddyfile</code></pre>

## POST /stop

正常停止服务并退出进程。如果您只想清空配置而非停止进程，您可以使用 [DELETE /config/](#delete-configpath)。

### Example

停止进程：

<pre><code class="cmd bash">curl -X POST "http://localhost:2019/stop"</code></pre>

## GET /config/[path]

导出 Caddy 指定路径下的配置，返回 JSON 结构。

### Examples

导出完整配置并格式化打印：

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/config/" | jq</span>
{
	"apps": {
		"http": {
			"servers": {
				"myserver": {
					"listen": [
						":443"
					],
					"routes": [
						{
							"match": [
								{
									"host": [
										"example.com"
									]
								}
							],
							"handle": [
								{
									"handler": "file_server"
								}
							]
						}
					]
				}
			}
		}
	}
}</code></pre>

仅导出监听地址：

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/config/apps/http/servers/myserver/listen"</span>
[":443"]</code></pre>

## POST /config/[path]

修改请求路径中指定位置的配置，如果请求路径指向数组，则进行 append；如果请求路径指向对象，则创建或替代对象。

特殊的，可以这样一次向数组添加多个内容：

1. 请求路径以 `/...` 结尾
2. `/...` 之前的元素是一个数组
3. 请求体的内容是一个数组

这种情况下，请求体数组中的内容会被展开，并全部添加到目标数组中。这种行为就像是 Go 语言中的：

```go
baseSlice = append(baseSlice, newElems...)
```

### Examples

添加监听地址：

<pre><code class="cmd bash">curl \
	-H "Content-Type: application/json" \
	-d '":8080"' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen"</code></pre>

添加多个监听地址：

<pre><code class="cmd bash">curl \
	-H "Content-Type: application/json" \
	-d '[":8080", ":5133"]' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen/..."</code></pre>

## PUT /config/[path]

修改请求路径中指定位置的配置，如果路径是数组中的序号，则进行插入；如果路径是对象，则一定会创建一个新值。

### Example

将一个监听地址添加到数组首位：

<pre><code class="cmd bash">curl -X PUT \
	-H "Content-Type: application/json" \
	-d '":8080"' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen/0"</code></pre>

## PATCH /config/[path]

修改请求路径中指定位置的配置，PATCH 一定会替换现有的值或是数组中的元素。

### Example

替换监听地址：

<pre><code class="cmd bash">curl -X PATCH \
	-H "Content-Type: application/json" \
	-d '[":8081", ":8082"]' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen"</code></pre>

## DELETE /config/[path]

删除请求路径中指定位置的配置。

### Examples

卸载所有配置，但保持 Caddy 运行：

<pre><code class="cmd bash">curl -X DELETE "http://localhost:2019/config/"</code></pre>

停止某个 HTTP 服务：

<pre><code class="cmd bash">curl -X DELETE "http://localhost:2019/config/apps/http/servers/myserver"</code></pre>

<h2 id="using-id-in-json">
	在 JSON 中使用 <code>@id</code>
</h2>

您可以在您的 JSON 配置的某些部分嵌入 ID，从而实现快捷访问。

只需要为 JSON 对象添加 `"@id"` 字段，并提供一个唯一值。例如，您有一个经常使用的反向代理配置：

```json
{
	"@id": "my_proxy",
	"handler": "reverse_proxy"
}
```

这样您就可以使用 `/id/` 接口来实现和 `/config/` 相同的访问效果，而无需使用完整路径。带 ID 的访问会直接定位到 ID 所在的位置。

例如，在不使用 ID 的情况下访问上述路径：

```
/config/apps/http/servers/myserver/routes/1/handle/0/upstreams
```

而在使用 ID 的情况下： 

```
/id/my_proxy/upstreams
```

这样会让配置维护更加方便。

<h2 id="concurrent-config-changes">
	并发配置变更
</h2>

<aside class="tip">

这部分内容适合所有的 `/config/` 接口，这部分内容是实验性的，随时可能会改变。

</aside>

Caddy 的配置接口为每个独立请求生成 [ACID <img src="/old/resources/images/external-link.svg" class="external-link">](https://en.wikipedia.org/wiki/ACID)，但在没有同步机制的情况下，设计到多个请求调用的数据变更动作还是有可能发生数据冲突或数据丢失的情况。

Caddy's config API provides [ACID guarantees <img src="/old/resources/images/external-link.svg" class="external-link">](https://en.wikipedia.org/wiki/ACID) for individual requests, but changes that involve more than a single request are subject to collisions or data loss if not properly synchronized.

例如，两个客户端同时进行 `GET /config/foo`，随后对其中的配置进行修改，然后再同时使用 `POST|PUT|PATCH|DELETE /config/foo/...` 应用变更，这就会导致数据冲突：其中一个客户端的变更结果会覆盖另一个客户端的结果，或者第二次变更可能会让配置处于意外状态，因为当他变更时已经不再是基于他获取的内容进行修改。两个客户端互相不知道对方进行的变更操作。

Caddy 的 API 并不支持跨越多个请求的事务，而且 HTTP 是一个无状态协议。不过，您可以使用 `Etag` 后缀和 `If-Match` 头来检测并防止冲突，作为一种乐观的并发控制机制。这在您的代码在非同步的情况下调用 `/config/...` 接口时很有用。所有的 `GET /config/...` 请求的响应都会带有一个被称为 `Etag` 的尾部，包含了请求路径和对其中内容 hash 的值（例如：`Etag: "/config/apps/http/servers 65760b8e"`）。随后在变更请求上添加 `If-Match` 头并使用上一个 `Get` 请求中 Etag 尾部。

基础的算法流程是这样的：

1. 使用 `GET` 请求配置中的 `S` 部分，保留响应中的 `Etag`。
2. 对返回配置进行需要的修改。
3. 对 `S` 部分进行 `POST|PUT|PATCH|DELETE`，在 `If-Match` 头部中添加 `Etag` 的值。
4. 如果 HTTP 返回状态码为 412（Precondition Failed），重复布州 1，或在多次尝试后放弃。

这种算法可以在不适用显示同步机制的条件下进行多个并发的配置变更。这样的设计方式可以让对配置不同模块进行修改时无需重试，只有对相同模块的修改并产生了数据冲突时才需要重试。

## POST /adapt

将输入配置适配为 JSON 格式并在响应包体中返回，并不会加载或运行这些配置。

和 [/load](#post-load) 接口一样，Content-Type 头用来指定配置文件的格式。例如，使用 `Content-Type: text/caddyfile` 适配 Caddyfile。

此接口可以适配您已经安装插件的任何 [配置适配器](/docs/config-adapters) 格式。

### Examples

将 Caddyfile 适配为 JSON：

<pre><code class="cmd bash">curl "http://localhost:2019/adapt" \
	-H "Content-Type: text/caddyfile" \
	--data-binary @Caddyfile</code></pre>

## GET /pki/ca/&lt;id&gt;

通过 ID 返回某个特定 [PKI 应用](/docs/json/apps/pki/) 的 CA 信息。如果请求的 CA ID 是默认值（`local`），且本地 CA 未配置，则会配置该 CA 并返回。如果请求的 CA ID 是其他值且未配置，则会返回错误。

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/pki/ca/local" | jq</span>
{
	"id": "local",
	"name": "Caddy Local Authority",
	"root_common_name": "Caddy Local Authority - 2022 ECC Root",
	"intermediate_common_name": "Caddy Local Authority - ECC Intermediate",
	"root_certificate": "-----BEGIN CERTIFICATE-----\nMIIB ... gRw==\n-----END CERTIFICATE-----\n",
	"intermediate_certificate": "-----BEGIN CERTIFICATE-----\nMIIB ... FzQ==\n-----END CERTIFICATE-----\n"
}</code></pre>

## GET /pki/ca/&lt;id&gt;/certificates

通过 ID 返回某个特定 [PKI 应用](/docs/json/apps/pki/) CA 的证书链。如果请求的 CA ID 是默认值（`local`），且本地 CA 未配置，则会配置该 CA 并返回结果。如果请求的 CA ID 是其他值且未配置，则会返回错误。

此接口在内部使用 [`caddy trust`](/docs/command-line#caddy-trust) 命令在系统信任仓库安装 CA 的根证书。

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/pki/ca/local/certificates"</span>
-----BEGIN CERTIFICATE-----
MIIByDCCAW2gAwIBAgIQViS12trTXBS/nyxy7Zg9JDAKBggqhkjOPQQDAjAwMS4w
...
By75JkP6C14OfU733oElfDUMa5ctbMY53rWFzQ==
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIBpDCCAUmgAwIBAgIQTS5a+3LUKNxC6qN3ZDR8bDAKBggqhkjOPQQDAjAwMS4w
...
9M9t0FwCIQCAlUr4ZlFzHE/3K6dARYKusR1ck4A3MtucSSyar6lgRw==
-----END CERTIFICATE-----</code></pre>

## GET /reverse_proxy/upstreams

将当前配置反向代理的上游（后端）状态做 JSON 返回。

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/reverse_proxy/upstreams" | jq</span>
[
	{"address": "10.0.1.1:80", "num_requests": 4, "fails": 2},
	{"address": "10.0.1.2:80", "num_requests": 5, "fails": 4},
	{"address": "10.0.1.3:80", "num_requests": 3, "fails": 3}
]</code></pre>

JSON 列表中的每一个条目都是全局上游池中配置的一个 [上游](/docs/json/apps/http/servers/routes/handle/reverse_proxy/upstreams/)。

- **address** 上游服务的地址。
- **num_requests** 是上游当前正在处理的活动请求的数量。
- **fails** 由被动运行状况检查配置的当前记住的失败请求数。

如果您的目标是确定后端的可用性，则需要根据您正在使用的处理程序配置交叉检查上游的相关属性。例如，如果您为反向代理启动了 [被动健康检查](/docs/json/apps/http/servers/routes/handle/reverse_proxy/health_checks/passive/)，您需要同时观察 `fails` 和 `num_requests` 的值来判断后端是否可用：检查 `fails` 是否小于您为代理配置的最大失败次数（例如 [`max_fails`](/docs/json/apps/http/servers/routes/handle/reverse_proxy/health_checks/passive/max_fails/)），同时检查 `num_requests` 是否小于或等于您配置的最大请求数（例如在代理全局配置的 [`unhealthy_request_count`](/docs/json/apps/http/servers/routes/handle/reverse_proxy/health_checks/passive/unhealthy_request_count/)，或是对每个上游配置的 [`max_requests`](/docs/json/apps/http/servers/routes/handle/reverse_proxy/upstreams/max_requests/)）。
