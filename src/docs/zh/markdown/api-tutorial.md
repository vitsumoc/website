---
title: "API 教程"
---

<h1 id="api-tutorial">
  API 教程
</h1>

本教程将引导您使用 Caddy 的 [admin API](/docs/api)，让您可以利用程序自动化控制 Caddy。

**目标：**
- 🔲 运行 Caddy
- 🔲 添加配置
- 🔲 测试配置
- 🔲 更新配置
- 🔲 遍历配置
- 🔲 使用 `@id` 标签

**您需要准备：**
- 使用 终端/命令行
- 了解 JSON
- 将 `caddy` 和 `curl` 配置到您的 PATH

---

首先使用 `run` 标志运行 Caddy：

<pre><code class="cmd bash">caddy run</code></pre>

<aside class="complete">运行 Caddy</aside>

这将阻塞您的终端，并且......没有任何其他用处。因为在默认情况下 Caddy 的配置是空的，我们可以在另一个终端上使用 [admin API](/docs/api) 确认这一点：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

我们可以向 Caddy 添加配置来让他变得有用，添加配置的一种方式是向 [/load](/docs/api#post-load) 接口发送携带配置的 POST 请求。有很多种发送 HTTP 请求的方式，在这个教程里我们使用 `curl`。

<h2 id="your-first-config">
  初见配置
</h2>

为了发送请求，我们需要提前准备一份配置内容，Caddy 的配置内容一般是 [JSON 文档](/docs/json/)（或是 [其他任何可被转换成 JSON 的格式](/docs/config-adapters)）。

<aside class="tip">
	配置文件不是必须的，配置 API 可以在没有文件的情况下被调用，这让程序自动化控制更方便。本教程中使用配置文件，是因为这样便于我们手动编辑。
</aside>

将这些内容保存到一个 JSON 文件：

```json
{
	"apps": {
		"http": {
			"servers": {
				"example": {
					"listen": [":2015"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Hello, world!"
							}]
						}
					]
				}
			}
		}
	}
}
```

之后上传到 API：

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @caddy.json
</code></pre>

<aside class="tip">
	不要忘记文件名前的 @，这在 curl 中表示发送文件。
</aside>

<aside class="complete">添加配置</aside>

我们可以再发送一个 GET 请求来查看 Caddy 是否已经应用我们的新配置：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

使用浏览器打开 [localhost:2015](http://localhost:2015) 或是使用 `curl` 验证配置是否生效：

<pre><code class="cmd"><span class="bash">curl localhost:2015</span>
Hello, world!</code></pre>

<aside class="complete">测试配置</aside>

如果您看到了 _Hello, world!_，那么恭喜您 —— 成功了！在配置后立刻进行检查是一个非常好的习惯，特别是在生产环境中。

接下来让我们把 "Hello world!" 替换成一些更加励志的内容："I can do hard things."，在您的配置文件中进行修改，修改后的 handler 对象看起来应该是这样：

```json
{
	"handler": "static_response",
	"body": "I can do hard things."
}
```

保存配置文件，再次 POST 接口来更新配置：

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @caddy.json
</code></pre>

<aside class="complete">更新配置</aside>

再次检查配置是否更新：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

通过浏览器或是 `curl` 再次检查接口效果，这次您将会看到一条励志的信息！

<h2 id="config-traversal">
  配置遍历
</h2>

为了一点小小的配置变更而重新上传整个配置文件时不合理的，我们可以使用 Caddy 的 API 功能，在完全不使用配置文件的情况下修改我们的配置。

<aside class="tip">
	在生产环境中，因为一些小小的修改而替换整个配置文件是一个危险操作，这就好比是在 linux 系统中拥有 root 权限。Caddy 的 API 允许您约束自己的变更范围，从而避免非变更部分被意外更改。
</aside>

我们可以使用 URI 的路径遍历配置结构的内部，从而仅仅修改我们想要改动的部分（消息内容）：

<pre><code class="cmd bash">curl \
	localhost:2019/config/apps/http/servers/example/routes/0/handle/0/body \
	-H "Content-Type: application/json" \
	-d '"Work smarter, not harder."'
</code></pre>

<aside class="tip">

每次您使用 API 修改配置后，Caddy 都为将最新的配置保存下来，以备您需要使用 [**--resume** 恢复配置](/docs/command-line#caddy-run)！

</aside>

您可以使用相似的路径，通过 GET 请求来确认配置，例如：

<pre><code class="cmd bash">curl localhost:2019/config/apps/http/servers/example/routes</code></pre>

结果将会是:

```json
[{"handle":[{"body":"Work smarter, not harder.","handler":"static_response"}]}]
```

<aside class="tip">

您可以使用 [`jq` command <img src="/old/resources/images/external-link.svg" class="external-link">](https://stedolan.github.io/jq/) 美化 JSON 输出：**`curl ... | jq`**

</aside>

<aside class="complete">遍历配置</aside>

**重要提示：** 您可能已经发现了，一旦您使用 API 进行了配置变更，您的配置文件就过时了，这里有几种解决方案：

- 在 [caddy run](/docs/command-line#caddy-run) 运行时添加 `--resume` 标志，加载上次的活动配置。
- 不要混用配置文件和 API，保持只有一个事实来源。
- 修改配置后使用 GET 请求 [导出 Caddy 的新配置](/docs/api#get-configpath)（推荐程度低于上述两个方案）。

<h2 id="using-id-in-json">
  在 JSON 中使用 <code>@id</code>
</h2>

配置遍历功能当然很好用，但是您有没有发现 url 的路径有点太长了？

我们可以向 handler 对象添加一个 [`@id` 标签](/docs/api#using-id-in-json)，这样就可以更方便的找到他：

<pre><code class="cmd bash">curl \
	localhost:2019/config/apps/http/servers/example/routes/0/handle/0/@id \
	-H "Content-Type: application/json" \
	-d '"msg"'
</code></pre>

这会向 handler 对象添加 `"@id": "msg"` 属性，所以现在他看起来：

```json
{
	"@id": "msg",
	"body": "Work smarter, not harder.",
	"handler": "static_response"
}
```

<aside class="tip">

**@id** 标签可以被添加到任何对象中，其值也可以是任何类型（通常都会使用字符串）。详见 [这里](/docs/api#using-id-in-json)

</aside>

这样我们就可以直接访问他：

<pre><code class="cmd bash">curl localhost:2019/id/msg</code></pre>

我们也可以使用这种短路径修改消息内容：

<pre><code class="cmd bash">curl \
	localhost:2019/id/msg/body \
	-H "Content-Type: application/json" \
	-d '"Some shortcuts are good."'
</code></pre>

并且再次检查修改：

<pre><code class="cmd bash">curl localhost:2019/id/msg/body</code></pre>

<aside class="complete">使用 <code>@id</code> 标签</aside>

