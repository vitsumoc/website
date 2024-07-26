---
title: "入门"
---

<h1 id="getting-started">
  入门
</h1>

欢迎来到 Caddy！本教程将带您探索 Caddy 的基础用法和一些高级特性。

**目标:**
- 🔲 运行 caddy
- 🔲 试用 API
- 🔲 配置 Caddy
- 🔲 测试配置
- 🔲 制作 Caddyfile
- 🔲 使用配置适配器
- 🔲 通过配置文件启动服务
- 🔲 对比 JSON 和 Caddyfile
- 🔲 对比 API 和配置文件
- 🔲 后台运行
- 🔲 热重载配置

**您需要准备：**
- 使用 终端/命令行
- 使用文本编辑器
- 将 `caddy` 和 `curl` 配置到您的 PATH

---

**如果您已经通过包管理器 [安装 Caddy](/docs/zh/install)，那么 Caddy 很可能已经作为一个服务运行在您的主机中，请在进行本教程前检查并关闭 Caddy 服务。**

首先，让我们运行 `caddy`：

<pre><code class="cmd bash">caddy</code></pre>

好吧，在没有子命令的情况下，`caddy` 命令只会输出帮助信息。您可以在忘记命令的时候使用这种方式查看帮助。

为了将 `caddy` 运行起来，需要添加 `run` 子命令：

<pre><code class="cmd bash">caddy run</code></pre>

<aside class="complete">运行 caddy</aside>

`caddy` 已经运行，并且阻塞了终端，而且......好吧，啥也没做。默认情况下，Caddy 的配置内容是空的，我们打开另一个终端并使用 [admin API](/docs/api) 查看：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

<aside class="tip">

这 **并不是** 您的站点或服务：管理终端 `localhost:2019` 仅仅是用来控制 Caddy，而且默认情况下仅对本机开放。

</aside>

<aside class="complete">试用 API</aside>

通过给 Caddy 添加配置，可以让他变得有用起来。我们有很多种配置 Caddy 的方案，这一次，我们先尝试通过 `curl` 发送一个 POST 请求到 [/load](/docs/api#post-load) 接口。

<h2 id="your-first-config">
  初见配置
</h2>

为了准备接下来的配置操作，我们先准备一个配置内容。在 caddy 的程序里，配置实际上是一个 [JSON 文档](/docs/json/)。

将下方的配置保存成一个 JSON 文件（例如：`caddy.json`）：

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

<aside class="tip">

Caddy 的 [admin API](/docs/api) 是设计给其他程序或者脚本调用的，在实际的环境中您不需要准备配置文件。但是在本次的教程中，为了演示这个功能，我们需要配置文件。

</aside>


之后使用 `curl` 上传:

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @caddy.json
</code></pre>

<aside class="complete">配置 Caddy</aside>

可以在发送一个 GET 请求，检查 Caddy 是否接收了我们的配置：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

使用您的浏览器或 `curl` 访问 [localhost:2015](http://localhost:2015)，测试配置的效果：

<pre><code class="cmd"><span class="bash">curl localhost:2015</span>
Hello, world!</code></pre>

如果您看到  _Hello, world!_，说明我们的配置生效了！在修改配置后进行检查是一个良好的习惯，特别是将他们推送到生产环境之前。

<aside class="complete">测试配置</aside>

<h2 id="your-first-caddyfile">
  初见 Caddyfile
</h2>

仅仅是 Hello World 就 _有这么多事情要做_。

另一种配置 Caddy 的方式是使用 [**Caddyfile**](/docs/caddyfile)，上文中的 JSON 内容可以被简化为：

```caddy
:2015

respond "Hello, world!"
```

请将这些内容保存为文件 `Caddyfile`（不带扩展名），放入当前路径中。

<aside class="complete">制作 Caddyfile</aside>

如果 Caddy 仍在运行，那么先中止他（<kbd>Ctrl</kbd>+<kbd>C</kbd>），随后：

<pre><code class="cmd bash">caddy adapt</code></pre>

如果您的 Caddyfile 不在当前路径，或是被命名成了其他名字：

<pre><code class="cmd bash">caddy adapt --config /path/to/Caddyfile</code></pre>

您将会看到 JSON 输出，因为 caddy 使用 [_配置适配器_](/docs/config-adapters) 将 Caddyfile 转换成了原生的 JSON 格式。

<aside class="complete">使用配置适配器</aside>

现在我们可以将这些输出保存为 JSON 文件，然后通过 POST 发送给 Caddy 了.....但是这些都不需要！因为 `caddy` 自己就能做这些事情：如果当前运行路径下有一个名为 Caddyfile 的文件，并且运行 `caddy` 的时候没有指定其他配置，那么 Caddy 就会自动加载 Caddyfile，转换为 JSON，然后按照配置运行。

现在，我们的当前路径下已经有一个 Caddyfile 了，让我们再次 `caddy run`：

<pre><code class="cmd bash">caddy run</code></pre>

如果您的 Caddyfile 在别的路径，则是：

<pre><code class="cmd bash">caddy run --config /path/to/Caddyfile</code></pre>

（如果您将文件名称修改成了并非 “Caddyfile” 开头的文件名，那么您还需要指定适配器类型 `--adapter caddyfile`）

您可以再次进行接口测试，您会发现他如期工作！

<aside class="complete">通过配置文件启动服务</aside>

正如您看到的，有很多种通过配置文件启动 Caddy 的方式：

- 当前路径下的名为 Caddyfile 的文件
- 通过 `--config` 标志指定文件（还可以使用 `--adapter` 指定文件类型）
- 使用 `--resume` 标志（如果此前加载过配置）

## JSON vs. Caddyfile

现在您已经知道了，Caddyfile 最终被转换成 JSON 使用。

Caddyfile 看起来比 JSON 简便许多，是否意味着我们应该一直使用他呢？实际上每种方式都有他的优点和缺点，最终的答案取决于您的需求和使用场景。

JSON | Caddyfile
-----|----------
容易生成 | 容易手动配置
适合程序化 | 难以自动化
表达能力强 | 表达能力中等
支持全部的 Caddy 能力 | 支持大部分 Caddy 能力
可以配置遍历 | 无法配置遍历
部分配置更改 | 仅支持完整配置更改
可以导出 | 无法导出
适合所有 API 接口 | 适合部分 API 接口
自动生成文档 | 手写文档
无处不在 | 特定位置
执行效率高 | 更多计算量
枯燥 | 有趣
**了解更多：[JSON structure](/docs/json/)** | **了解更多：[Caddyfile docs](/docs/caddyfile)**

您需要根据您的使用场景决定最佳选项。

需要注意的是，JSON 和 Caddyfile（或是 [其他受支持的配置类型](/docs/config-adapters)）都可以使用 [Caddy's API](/docs/api)。然而，所有的 API 功能接口都支持 JSON，只有 [/load endpoint](/docs/api#post-load) 支持配置适配器。

<aside class="complete">对比 JSON 和 Caddyfile</aside>

## API vs. Config files

<aside class="tip">

实际上，当您使用 `caddy` 命令加载配置文件时，程序只是自动将文件发送到 API 接口进行处理。

</aside>

您还得考虑服务器使用配置文件管理还是 API 管理。（您 _可以_ 同时使用 API 和配置文件，但是我们并不推荐这样做：单一的来源更加稳定且易维护。）

API | Config files
----|-------------
通过 HTTP 请求进行配置变更 | 通过终端指令进行配置变更
适合规模化 | 不适合规模化
难以手动维护 | 适合手动维护
非常有趣 | 有趣
**了解更多：[API tutorial](/docs/api-tutorial)** | **了解更多：[Caddyfile tutorial](/docs/caddyfile-tutorial)**

<aside class="tip">
	使用 API 手动管理配置也是可行的，但您需要准备一些 HTTP 客户端工具。
</aside>

选择 API 或是配置文件作为工作流，和选择 JSON 或是其他配置格式无关：您可以将 JSON 保存为配置文件，在终端中使用；您也可以将 Caddyfile 和 API 结合使用。

但是，绝大多数用户会选择 JSON+API 或是 Caddyfile+CLI 这样的组合方式。

就像您看到的一样，Caddy 适于各种各样的不同场景。

<aside class="complete">对比 API 和配置文件</aside>

<h2 id="start-stop-run">
  启动、停止和运行
</h2>

Caddy 是一个独立运行的服务端程序，这意味着他会阻塞您的终端，当您运行 `caddy run` 后他会持续占用您的窗口，除非您中止他（通常是<kbd>Ctrl</kbd>+<kbd>C</kbd>）。

虽然说 `caddy run` 是最常用也是最推荐的启动方式（特别是将 Caddy 配置为系统服务时！），但您也可以选择使用 `caddy start`，这会让 Caddy 在后台运行：

<pre><code class="cmd bash">caddy start</code></pre>

这样您就可以继续使用您的终端，这在一些纯命令行的情况下非常有用。

当然，这种情况下 <kbd>Ctrl</kbd>+<kbd>C</kbd> 无法用来中止 caddy，您需要使用：

<pre><code class="cmd bash">caddy stop</code></pre>

或是使用 API 中提供的 [/stop endpoint](/docs/api#post-stop).

<aside class="complete">后台运行</aside>

<h2 id="reloading-config">
  重载配置
</h2>

Caddy 支持热重载配置，配置变更过程中没有停止服务时间。

所有通过 [API endpoints](/docs/api) 进行的配置载入或是变更都是热重载的。

而当使用命令行时，虽然可以先 <kbd>Ctrl</kbd>+<kbd>C</kbd> 停止您的服务然后再重新启动他，以此来重载配置。但是不要这样做：重载配置并不需要重启服务，重启动作会导致服务中断。

<aside class="tip">
	停止或重启服务会导致业务中断。
</aside>

取而代之的是，使用 [`caddy reload`](/docs/command-line#caddy-reload) 命令进行热重载：

<pre><code class="cmd bash">caddy reload</code></pre>

这实际上会进行一次 API 调用，他会加载文件、将配置文件转换为 JSON，之后无缝替换现有的配置。

如果在加载新配置文件时产生了错误，Caddy 会回滚到上一个可用的配置。

<aside class="tip">
	从技术上来说，会先启用新的服务配置，之后再停止旧的服务配置，因此，会有一个极短的时间内两种配置同时存在！如果新的配置错误而无法使用，那么旧的服务配置则不会停止。
</aside>

<aside class="complete">热重载配置</aside>
