---
title: log (Caddyfile directive)
---

<script>
window.$(function() {
	// Fix > in code blocks
	window.$('pre.chroma .k:contains(">")')
		.each(function() {
			const e = window.$(this);
			// Skip if ends with >
			if (e.text().trim().endsWith('>')) return;
			// Replace > with <span class="p">&gt;</span>
			e.html(e.html().replace(/&gt;/g, '<span class="p">&gt;</span>'));
		});

	// We'll add links to all the subdirectives if a matching anchor tag is found on the page.
	addLinksToSubdirectives();
});
</script>

# log

启动并配置 HTTP 请求日志（也被称为接入日志）。

<aside class="tip">

如需配置 Caddy 的运行日志，请参考 [全局选项 `log`](/docs/caddyfile/options#log)。

</aside>

`log` 指令会应用他出现的站点块中的 hostnames，除非被 `hostnames` 子指令覆盖。

配置了 `log` 指令后，默认情况下所有访问该站点的请求都会被日志记录，如需有条件的跳过某些请求，可以使用 [`log_skip`](log_skip) 指令。

如需向日志中添加自定义字段，可以使用 [`log_append`](log_append) 指令。

- [语法](#syntax)
- [Output modules](#output-modules)
  - [stderr](#stderr)
  - [stdout](#stdout)
  - [discard](#discard)
  - [file](#file)
  - [net](#net)
- [Format modules](#format-modules)
  - [console](#console)
  - [json](#json)
  - [filter](#filter)
    - [delete](#delete)
	- [rename](#rename)
	- [replace](#replace)
	- [ip_mask](#ip-mask)
	- [query](#query)
	- [cookie](#cookie)
	- [regexp](#regexp)
	- [hash](#hash)
  - [append](#append)
- [示例](#examples)

从 Caddy v2.5 开始，默认情况下，包含潜在敏感信息（`Cookie`、`Set-Cookie`、`Authorization` 和 `Proxy-Authorization`）的头将以空值记录。可以使用 [`log_credentials`](/docs/caddyfile/options#log-credentials) 全局选项禁用此行为。

<h2 id="syntax">
	语法
</h2>

```caddy-d
log [<logger_name>] {
	hostnames <hostnames...>
	output <writer_module> ...
	format <encoder_module> ...
	level  <level>
}
```

- **logger_name** 可选项，覆盖此站点中 logger 的名称。

  默认情况下，logger 的名称是自动生成的，例如 `log0`，`log1`，取决于站点在 Caddyfile 中的顺序。这个配置只有在您希望将此 logger 的内容输出到全局选项中定义的另一个 logger 中时才有用。[这个例子](#multiple-outputs) 里有描述。

- **hostnames** 可选项，覆盖日志中使用的 hostname。

  默认情况下，logger 会使用他所在的站点块中的 hostname，例如站点地址。这个配置在您想要在 [通配符站点块](/docs/caddyfile/patterns#wildcard-certificates) 中为不同的子域名区分 logger 时很有用。[这个例子](#wildcard-logs) 里有描述。

- **output** 配置日志输出的位置。参考下方的 [`output` modules](#output-modules)。

  默认：`stderr`.

- **format** 描述如何编码（或是格式化）日志，参考下方的 [`format` modules](#format-modules)。

  默认：如果检测到使用 `stderr` 输出，则使用 `console`，其他情况下使用 `json`。

- **level** 需要进行日志记录的最低等级，默认： `INFO`。

  注意，接入日志现在只有 `INFO` 和 `ERROR` 两个等级。

### Output modules

**output** 子指令用于指定日志输出到何处。

#### stderr

标准错误输出（默认情况下打印到控制台）。

```caddy-d
output stderr
```

#### stdout

标准输出（打印到控制台）。

```caddy-d
output stdout
```

#### discard

无输出。

```caddy-d
output discard
```

#### file

输出到文件，默认情况下，日志文件会自动循环存储，用于降低磁盘消耗。

日志的循环存储是使用 [lumberjack <img src="/old/resources/images/external-link.svg" class="external-link">](https://github.com/natefinch/lumberjack) 实现的。

```caddy-d
output file <filename> {
	roll_disabled
	roll_size     <size>
	roll_uncompressed
	roll_local_time
	roll_keep     <num>
	roll_keep_for <days>
}
```

- **&lt;filename&gt;** 日志文件路径。

- **roll_disabled** 禁用循环存储，这可能会导致磁盘被用尽，您应该在有其他方式管理日志文件大小的情况下使用此选项。

- **roll_size** 循环存储的文件大小。当前的实现支持兆字节分辨率；小数值向上舍入到下一个整数兆字节。例如，1.1Mi B 向上舍入为 2Mi B。

  默认： `100MiB`

- **roll_uncompressed** 关闭对日志的 gzip 压缩。

  默认：启用 gzip 压缩

- **roll_local_time** 设置滚动以在文件名中使用本地时间戳。

  默认：使用 UTC 时间

- **roll_keep** 保存多少个日志文件。

  默认：`10`

- **roll_keep_for** 使用 [duration string](/docs/conventions#durations) 描述循环文件保存的时间长度。当前的实现支持天级别的分辨率；小数值向上舍入到下一天。例如，`36h`（1.5 天）会被向上舍入为 `48h`（2 天）。

	默认：`2160h`（90 天）

#### net

写入到 socket，如果 socket 不可用，则会在尝试重连期间将日志写入到 stderr。

```caddy-d
output net <address> {
	dial_timeout <duration>
	soft_start
}
```

- **&lt;address&gt;** 写入日志内容的 [地址](/docs/conventions#network-addresses)。

- **dial_timeout** 是等待成功连接到 socket 的时间。如果 socket 出现故障，日志发送可能会被阻止长达这么长时间。

- **soft_start** 在连接到 socket 时将忽略错误，即使远程日志服务关闭，也允许您加载配置。日志将被发送到 stderr。

### Format modules

**format** 子指令用于定制日志如何编码（格式化）。此指令出现在 `log` 块中。

<aside class="tip">

**关于通用日志格式 (CLF) 的说明：** CLF 与现代结构化日志发生冲突。如需将您的日志转换为已弃用的 CLF 格式，请使用 [`transform-encoder` <img src="/old/resources/images/external-link.svg" class="external-link">](https://github.com/caddyserver/transform-encoder) 插件 。

</aside>

除了下文中每个独立编码器的专有语法之外，大部分编码器还可以使用这些通用属性：

```caddy-d
format <encoder_module> {
	message_key     <key>
	level_key       <key>
	time_key        <key>
	name_key        <key>
	caller_key      <key>
	stacktrace_key  <key>
	line_ending     <char>
	time_format     <format>
	time_local
	duration_format <format>
	level_format    <format>
}
```

- **message_key** 日志条目中消息字段的键名。默认：`msg`

- **level_key** 日志条目中日志等级字段的键名。默认：`level`

- **time_key** 日志条目中时间字段的键名。默认： `ts`

- **name_key** 日志条目中名称字段的键名。默认：`name`

- **caller_key** 日志条目中 caller 字段的键名。

- **stacktrace_key** 日志条目中栈追踪字段的键名。

- **line_ending** 用在行结尾。

- **time_format** 时间戳格式。

  默认：日志格式为 `console` 时使用 `wall_milli`，其他情况下则是 `unix_seconds_float`。
  
  可以是下列选项之一：
  - `unix_seconds_float` 自 Unix 纪元以来的浮点数秒数。
  - `unix_milli_float` 自 Unix 纪元以来的浮点数毫秒数。
  - `unix_nano` 自 Unix 纪元以来的纳秒整数。
  - `iso8601` 示例：`2006-01-02T15:04:05.000Z0700`
  - `rfc3339` 示例：`2006-01-02T15:04:05Z07:00`
  - `rfc3339_nano` 示例：`2006-01-02T15:04:05.999999999Z07:00`
  - `wall` 示例：`2006/01/02 15:04:05`
  - `wall_milli` 示例：`2006/01/02 15:04:05.000`
  - `wall_nano` 示例：`2006/01/02 15:04:05.000000000`
  - `common_log` 示例：`02/Jan/2006:15:04:05 -0700`
  - 或者，任何兼容的时间格式字符串，参考 [Go 文档](https://pkg.go.dev/time#pkg-constants) 了解细节。
  
  请注意，时间格式字符串中的内容是一种特殊的常量占位符，`2006` 表示年份，`01` 表示月份，`Jan` 表示月份的字符串表示，`02` 表示天（Go 就是这样设计的）。不要再时间格式字符串中使用当前的日期时间。

- **time_local** 使用系统本地时间而非默认的 UTC 时间来记录日志。

- **duration_format** 经过时间的格式。

  默认：`seconds`
  
  可以是下列选项之一：
  - `seconds` 经过的秒数浮点数。
  - `nano` 经过的纳秒整数。
  - `string` 使用 Go 内置的字符串格式，例如 `1m32.05s` 或 `6.31ms`。

- **level_format** 日志等级的格式。

  默认：日志格式为 `console` 时使用 `color`，其他情况下则是 `lower`。
  
  可以是下列选项之一：
  - `lower` 小写。
  - `upper` 大写。
  - `color` 大写，并带有 ANSI 颜色。

#### console

console 编码器会将日志格式化为适合人类阅读的格式，同时也会保留一些结构。

```caddy-d
format console
```

#### json

将日志条目格式化为 JSON 对象。

```caddy-d
format json
```

#### filter

支持对每个字段的过滤。

```caddy-d
format filter {
	fields {
		<field> <filter> ...
	}
	<field> <filter> ...
	wrap <encode_module> ...
}
```

可以通过用 `>` 表示嵌套层来引用嵌套字段。换句话说，对于像 `{"a":{"b":0}}` 这样的对象，内部字段可以被引用为 `a>b`。

以下字段是日志的基础字段，但因为他们是由底层日志库添加的，所以无法过滤： `ts`，`level`，`logger` 和 `msg`。

`wrap` 是可选的；如果省略，则根据当前输出模块是 [`stderr`](#stderr) 还是 [`stdout`](#stdout) 来选择默认值。如果是交互式终端，在这种情况下选择 [`console`](#console)，否则选择 [`json`](#json)。

作为一种快捷方式，可以省略 `fields` 块，直接将过滤条件写在 `filter` 块中。

可用的 filters 包括：

##### delete

在编码时跳过某字段。

```caddy-d
<field> delete
```

##### rename

修改日志字段的键名。

```caddy-d
<field> rename <key>
```

##### replace

在编码时将某字段的值替换为提供的字符串。

```caddy-d
<field> replace <replacement>
```

##### ip_mask

使用 CIDR 掩码屏蔽字段中的 IP 地址，即从左侧开始 IP 中要保留的位数。如果该字段是字符串数组（例如 HTTP 标头），则数组中的每个值都会被屏蔽。该值可以是逗号分隔的 IP 地址字符串。

由于 IPv4 和 IPv6 的地址总长度不同，因此他们的配置是区分开的。

常见情况下，本字段的过滤器大概是：
- `request>remote_ip` 用于直接连接的客户端
- `request>client_ip` 用于配置了 [`trusted_proxies`](/docs/caddyfile/options#trusted-proxies) 时解析到的 “真实客户端”
- `request>headers>X-Forwarded-For` 如果位于反向代理后方

```caddy-d
<field> ip_mask [<ipv4> [<ipv6>]] {
	ipv4 <cidr>
	ipv6 <cidr>
}
```

##### query

标记字段并执行一项或多项操作，用来操作 URL 字段的查询部分（query part）。通常来说要过滤的字段是 `request>uri`。

```caddy-d
<field> query {
	delete  <key>
	replace <key> <replacement>
	hash    <key>
}
```

可用的行为包括：

- **delete** 从查询中删除指定的键。

- **replace** 将指定键的值替换为 **replacement**。可以用来插入密文占位符，您将会看到 URL 中有该查询键，但其值是隐藏的。

- **hash** 将给定查询键的值替换为该值的 SHA-256 哈希值的前 4 个字节（小写十六进制）。在值的内容属于敏感信息时很有用，同时也可以用来区分每个请求是否使用了不同的值。

##### cookie

标记字段并执行一项或多项操作，用来操作 HTTP 头中的 `Cookie`。通常来说要过滤的字段是 `request>headers>Cookie`。

```caddy-d
<field> cookie {
	delete  <name>
	replace <name> <replacement>
	hash    <name>
}
```

可用的行为包括：

- **delete** 通过头中的名称删除指定 cookie。

- **replace** 将指定 cookie 的值替换为 **replacement**。可以用来插入密文占位符，您将会看到头中有 cookie，但其值是隐藏的。

- **hash** 将给定 cookie 的值替换为该值的 SHA-256 哈希值的前 4 个字节（小写十六进制）。在值的内容属于敏感信息时很有用，同时也可以用来区分每个请求是否使用了不同的值。

如果对同一个 cookie name 定义了多个行为，那么只有第一个行为会被应用。

##### regexp

标记字段，并编码时使用正则表达式替换。如果该字段是字符串数组（例如 HTTP 头），则数组中的每个值都会被替换。

```caddy-d
<field> regexp <pattern> <replacement>
```

使用的正则表达式语言是 Go 中的 RE2 。请参阅 [RE2 syntax reference](https://github.com/google/re2/wiki/Syntax) 和 [Go regexp syntax overview](https://pkg.go.dev/regexp/syntax)。

在替换字符串中，可以使用 `${group}` 引用捕获组，其中 `group` 是表达式中捕获组的名称或编号。捕获组 `0` 是完整的正则表达式匹配，`1` 是第一个捕获组，`2` 是第二个捕获组，依此类推。

##### hash

标记一个字段，在编码时将其值替换为现有值的 SHA-256 哈希前 4 个字节（8 个十六进制字符）。如果字段是字符串数组（例如 HTTP 头），则数组中的每个值都会进行哈希处理。

可以用来隐藏敏感信息，同时也可以提示每个请求中的信息是否是不同的值。

```caddy-d
<field> hash
```

#### append

对日志条目添加字段。

```caddy-d
format append {
	fields {
		<field> <value>
	}
	<field> <value>
	wrap <encode_module> ...
}
```

在添加额外信息，用来区分产生日志的 Caddy 实例时很有效，可以配合环境变量实现。字段的值可以是全局占位符（例如 `{env.*}`），但 _不能_ 使用请求中的占位符，因为日志的写入动作是在脱离了 HTTP 请求上下文环境后执行的。


`wrap` 是可选的；如果省略，则根据当前输出模块是 [`stderr`](#stderr) 还是 [`stdout`](#stdout) 来选择默认值。如果是交互式终端，在这种情况下选择 [`console`](#console)，否则选择 [`json`](#json)。

作为一种快捷方式，可以省略 `fields` 块，直接将过滤条件写在 `append` 块中。

<h2 id="examples">
	示例
</h2>

启动默认 logger。

换句话说，日志将输出到 `stderr`，但此行为可以通过配置 [`log` 全局选项](/docs/caddyfile/options#log) 中的 `default` 来改变：

```caddy
example.com {
	log
}
```

将日志写入文件（并默认启用循环存储）：

```caddy
example.com {
	log {
		output file /var/log/access.log
	}
}
```

自定义循环存储：

```caddy
example.com {
	log {
		output file /var/log/access.log {
			roll_size 1gb
			roll_keep 5
			roll_keep_for 720h
		}
	}
}
```

从日志中删除请求头中的 `User-Agent`：

```caddy
example.com {
	log {
		format filter {
			request>headers>User-Agent delete
		}
	}
}
```

隐藏多个敏感 cookie（请注意很多敏感的请求头在默认情况下就会被替换为空值，参考 [`log_credentials` 全局选项](/docs/caddyfile/options#log-credentials) 来启用对 `Cookie` 头的记录）：

```caddy
example.com {
	log {
		format filter {
			request>headers>Cookie cookie {
				replace session REDACTED
				delete secret
			}
		}
	}
}
```

使用掩码隐藏请求中的远端 IP 地址，对 IPv4 地址保留前 16 位（例如 255.255.0.0），对于 IPv6 地址保留前 32 位。

请注意，从 Caddy v2.7 开始，`remote_ip` 和 `client_ip` 都会被记录，其中 `client_ip` 是配置 [`trusted_proxies`](/docs/caddyfile/options#trusted-proxies) 时的 “真实 IP”：

```caddy
example.com {
	log {
		format filter {
			request>remote_ip ip_mask 16 32
			request>client_ip ip_mask 16 32
		}
	}
}
```

给所有日志条目后缀一个来自环境变量的服务器 ID，然后套一个 `filter` 来删除一个请求头：

```caddy
example.com {
	log {
		format append {
			server_id {env.SERVER_ID}
			wrap filter {
				request>headers>Cookie delete
			}
		}
	}
}
```

<span id="wildcard-logs" /> 通过覆盖日志记录器中的 `hostnames`，实现为 [通配符站点块](/docs/caddyfile/patterns#wildcard-certificates) 中的每个子域名单独写入日志文件。此处使用了 [引用片段](/docs/caddyfile/concepts#snippets) 来避免重复：

```caddy
(subdomain-log) {
	log {
		hostnames {args[0]}
		output file /var/log/{args[0]}.log
	}
}

*.example.com {
	import subdomain-log foo.example.com
	@foo host foo.example.com
	handle @foo {
		respond "foo"
	}

	import subdomain-log bar.example.com
	@bar host bar.example.com
	handle @bar {
		respond "bar"
	}
}
```

<span id="multiple-outputs" /> 将某个子域名的接入日志写入两份不同的文件（其一是 [`transform-encoder` plugin <img src="/old/resources/images/external-link.svg" class="external-link">](https://github.com/caddyserver/transform-encoder)，另外一份是 [`json`](#json)）。

本实例在站点块中将 logger 命名为 `foo`，之后在全局选项中定义两个 logger，使用 `include http.log.access.foo` 接收他产生的日志：

```caddy
{
	log access-formatted {
		include http.log.access.foo
		output file /var/log/access-foo.log
		format transform "{common_log}"
	}

	log access-json {
		include http.log.access.foo
		output file /var/log/access-foo.json
		format json
	}
}

foo.example.com {
	log foo
}
```
