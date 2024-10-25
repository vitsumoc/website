---
title: Caddyfile 指令
---

<style>
#directive-table table {
	margin: 0 auto;
	overflow: hidden;
}

#directive-table tr:hover {
	background: rgba(109, 226, 255, 0.11);
}

#directive-table tr td:first-child {
	position: relative;
}

#directive-table a:before {
	content: '';
	position: absolute;
	left: 0;
	top: 0;
	bottom: 0;
	display: block;
	width: 100vw;
}
</style>

<h1 id="caddyfile-directives">
	Caddyfile 指令
</h1>

指令是站点 [块](/docs/caddyfile/concepts#blocks) 中使用的功能性关键字。指令可以使用自己的块来存放 _子指令_，但除非文档特别描述，指令 **不能** 出现在其他指令中。例如，您不能在 `file_server` 块中使用 `basic_auth`，因为 `file_server` 无从得知如何进行认证。当然，您 _可能_ 可以在一些特殊的指令块例如 `handle` 或 `route` 中使用指令，因为他们被设计就是用来聚合 HTTP 指令的。

- [格式](#syntax)
- [指令顺序](#directive-order)
- [排序算法](#sorting-algorithm)

下方列表为标准 Caddy 自带的指令，可以用于 HTTP Caddyfile 中：

<div id="directive-table">

指令 | 描述
----------|------------
**[abort](/docs/zh/caddyfile/directives/abort)** | 中止 HTTP 请求
**[acme_server](/docs/zh/caddyfile/directives/acme_server)** | 嵌入式 ACME 服务器
**[basic_auth](/docs/zh/caddyfile/directives/basic_auth)** | 启动 HTTP 基本身份验证
**[bind](/docs/zh/caddyfile/directives/bind)** | 自定义服务的 socket 地址
**[encode](/docs/zh/caddyfile/directives/encode)** | 编码（通常是压缩）响应
**[error](/docs/zh/caddyfile/directives/error)** | 触发 error
**[file_server](/docs/zh/caddyfile/directives/file_server)** | 文件服务器
**[forward_auth](/docs/zh/caddyfile/directives/forward_auth)** | 将身份验证委托给外部服务
**[fs](/docs/zh/caddyfile/directives/fs)** | 选择文件系统
**[handle](/docs/zh/caddyfile/directives/handle)** | 一组独立执行的指令
**[handle_errors](/docs/zh/caddyfile/directives/handle_errors)** | 定义处理错误的路由
**[handle_path](/docs/zh/caddyfile/directives/handle_path)** | 与 handle 相似，但会删除路径前缀
**[header](/docs/zh/caddyfile/directives/header)** | 设置或移除响应头
**[import](/docs/zh/caddyfile/directives/import)** | 引入引用片段或文件
**[invoke](/docs/zh/caddyfile/directives/invoke)** | 调用具名路由
**[log](/docs/zh/caddyfile/directives/log)** | 开启 接入/请求 日志
**[log_append](/docs/zh/caddyfile/directives/log_append)** | 在接入日志后追加字段
**[log_skip](/docs/zh/caddyfile/directives/log_skip)** | 跳过匹配到请求的接入日志
**[map](/docs/zh/caddyfile/directives/map)** | 将一个输入值映射到一个或多个输出
**[method](/docs/zh/caddyfile/directives/method)** | 修改 HTTP 请求方法
**[metrics](/docs/zh/caddyfile/directives/metrics)** | 配置提供 Prometheus 指标的接口
**[php_fastcgi](/docs/caddyfile/directives/php_fastcgi)** | Serve PHP sites over FastCGI
**[push](/docs/caddyfile/directives/push)** | Push content to the client using HTTP/2 server push
**[redir](/docs/caddyfile/directives/redir)** | Issues an HTTP redirect to the client
**[request_body](/docs/caddyfile/directives/request_body)** | Manipulates request body
**[request_header](/docs/caddyfile/directives/request_header)** | Manipulates request headers
**[respond](/docs/caddyfile/directives/respond)** | Writes a hard-coded response to the client
**[reverse_proxy](/docs/caddyfile/directives/reverse_proxy)** | A powerful and extensible reverse proxy
**[rewrite](/docs/caddyfile/directives/rewrite)** | Rewrites the request internally
**[root](/docs/caddyfile/directives/root)** | Set the path to the site root
**[route](/docs/caddyfile/directives/route)** | A group of directives treated literally as single unit
**[templates](/docs/caddyfile/directives/templates)** | Execute templates on the response
**[tls](/docs/caddyfile/directives/tls)** | Customize TLS settings
**[tracing](/docs/caddyfile/directives/tracing)** | Integration with OpenTelemetry tracing
**[try_files](/docs/caddyfile/directives/try_files)** | Rewrite that depends on file existence
**[uri](/docs/caddyfile/directives/uri)** | Manipulate the URI
**[vars](/docs/caddyfile/directives/vars)** | Set arbitrary variables

</div>

## Syntax

The syntax of each directive will look something like this:

```caddy-d
directive [<matcher>] <args...> {
	subdirective [<args...>]
}
```

The `<carets>` indicate tokens to be substituted by actual values.

The`[brackets]` indicate optional parameters.

The ellipses `...` indicates a continuation, i.e. one or more parameters or lines.

Subdirectives are typically optional unless documented otherwise, even though they don't appear in `[brackets]`.


### Matchers

Most—but not all—directives accept [matcher tokens](/docs/caddyfile/matchers#syntax), which let you filter requests. Matcher tokens are usually optional. Directives support matchers if you see this in a directive's syntax:

```caddy-d
[<matcher>]
```

Because matcher tokens all work the same, the various possibilities for the matcher token will not be described on every page, to reduce duplication. Instead, refer to the [matcher documentation](/docs/caddyfile/matchers) for a detailed explanation of the syntax.


## Directive order

Many directives manipulate the HTTP handler chain. The order in which those directives are evaluated matters, so a default ordering is hard-coded into Caddy.

You can override/customize this ordering by using the [`order` global option](/docs/caddyfile/options#order) or the [`route` directive](/docs/caddyfile/directives/route).

```caddy-d
tracing

map
vars
fs
root
log_append
log_skip

header
copy_response_headers # only in reverse_proxy's handle_response block
request_body

redir

# incoming request manipulation
method
rewrite
uri
try_files

# middleware handlers; some wrap responses
basic_auth
forward_auth
request_header
encode
push
templates

# special routing & dispatching directives
invoke
handle
handle_path
route

# handlers that typically respond to requests
abort
error
copy_response # only in reverse_proxy's handle_response block
respond
metrics
reverse_proxy
php_fastcgi
file_server
acme_server
```



## Sorting algorithm

For ease of use, the Caddyfile adapter sorts directives according to the following rules:

- Differently named directives are sorted by their position in the [default order](#directive-order). The default order can be overridden with the [`order` global option](/docs/caddyfile/options). Directives from plugins _do not_ have an order, so the [`order`](/docs/caddyfile/options) global option or the [`route`](/docs/caddyfile/directives/route) directive should be used to set one.

- Same-named directives are sorted according to their [matchers](/docs/caddyfile/matchers#syntax).

  - The highest priority is a directive with a single [path matcher](/docs/caddyfile/matchers#path-matchers).

    Path matchers are sorted by specificity, from most specific to least specific.
	
	In general, this is performed by sorting by the length of the path matcher. There is one exception where if the path ends in a `*` and the paths of the two matchers are otherwise the same, the matcher with no `*` is considered more specific and sorted higher.

    For example:
    - `/foobar` is more specific than `/foo`
    - `/foo` is more specific than `/foo*`
    - `/foo/*` is more specific than `/foo*`

  - A directive with any other matcher is sorted next, in the order it appears in the Caddyfile.

    This includes path matchers with multiple values, and [named matchers](/docs/caddyfile/matchers#named-matchers).

  - A directive with no matcher (i.e. matching all requests) is sorted last.

- The [`vars`](/docs/caddyfile/directives/vars) directive has its ordering by matcher reversed, because it involves setting values which can overwrite eachother, so the most specific matcher should be evaluated last.

- The contents of the [`route`](/docs/caddyfile/directives/route) directive ignores all the above rules, and preserves the order the directives appear within.
