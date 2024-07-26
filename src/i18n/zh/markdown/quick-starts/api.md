---
title: 快速了解 API
---

<h1 id="api-quick-start">
	快速了解 API
</h1>

**您需要准备：**
- 使用 终端/命令行
- 将 `caddy` 和 `curl` 配置到您的 PATH

---

首先，运行 Caddy：

<pre><code class="cmd bash">caddy start</code></pre>

此时，Caddy 运行在一个空闲状态（没有配置）。通过 `curl` 进行一些简单的配置：

<pre><code class="cmd bash">curl localhost:2019/load \
    -H "Content-Type: application/json" \
    -d @- << EOF
    {
        "apps": {
            "http": {
                "servers": {
                    "hello": {
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
EOF</code></pre>

按照 [这个文档](https://en.wikipedia.org/wiki/Here_document#Unix_shells) 的要求来写 POST body 非常枯燥乏味，因此如果您需要的话，可以将内容保存为 JSON 文件 `caddy.json`，随后使用这样的方式进行配置：

<pre><code class="cmd bash">curl localhost:2019/load \
  -H "Content-Type: application/json" \
  -d @caddy.json
</code></pre>

使用您的浏览器或是 `curl` 访问 [localhost:2015](http://localhost:2015)：

<pre><code class="cmd"><span class="bash">curl localhost:2015</span>
Hello, world!</code></pre>

我们也可以在 JSON 中定义多个使用不同端口的服务：

```json
{
	"apps": {
		"http": {
			"servers": {
				"hello": {
					"listen": [":2015"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Hello, world!"
							}]
						}
					]
				},
				"bye": {
					"listen": [":2016"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Goodbye, world!"
							}]
						}
					]
				}
			}
		}
	}
}
```

更新您的 JSON 后，尝试再次访问这些接口。

您可以 [通过浏览器](http://localhost:2016) 或是 `curl` 测试新的 “goodbye” 接口：

<pre><code class="cmd"><span class="bash">curl localhost:2016</span>
Goodbye, world!</code></pre>

当您不再使用 Caddy 时，可以关闭他：

<pre><code class="cmd bash">caddy stop</code></pre>

还有很多您可以通过 API 做到的事情，例如导出配置或是对配置进行局部调整（而非全量更新）。请务必阅读 [full API tutorial](/docs/api-tutorial) 了解这些内容！

<h2 id="further-reading">
  延伸阅读
</h2>

- [Full API tutorial](/docs/api-tutorial)
- [API documentation](/docs/api)
