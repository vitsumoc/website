---
title: import (Caddyfile directive)
---

# import

引入 [引用片段](/docs/caddyfile/concepts#snippets) 或是文件，将此指令替换为代码片段或是文件的内容。

此指令是一个特例：他会在结构解析前执行，并且可以出现在 Caddyfile 中的任何地方。

<h2 id="syntax">
	语法
</h2>

```caddy-d
import <pattern> [<args...>]
```

- **&lt;pattern&gt;** 是将要被引入的文件名称、匹配模板（glob pattern）、或是 [引用片段](/docs/caddyfile/concepts#snippets)。其中的内容将会替代此行的内容，就像是那些内容一开始就出现在这里一样。

  如果指定的文件不存在则会报错，但匹配模板为空并不会报错。

  当引入指定文件，并且文件内容为空时，会产生警告。

  如果 pattern 是文件名或是匹配模板，则此处使用使用 `import` 指令所在文件的相对路径。

  如果使用匹配模板 `*` 作为路径的最后一部分，则会忽略隐藏的文件（例如，以 `.` 作为名称开始的文件），如果需要引入隐藏文件，请使用 `.*` 作为路径的最后一部分。

- **&lt;args...&gt;** 是一个可选的实际参数列表，其中的内容会被传递给导入的词元。这些占位符也是一种特例，他们会在 Caddyfile 解析时执行而非运行时。他们有多种使用方式，和 [Go 切片](https://gobyexample.com/slices) 类似：
  - `{args[n]}` 以 0 为起始，获得第 `n` 个参数
  - `{args[:]}` 所有的参数
  - `{args[:m]}` `m` 之前的所有参数
  - `{args[n:]}` 从 `n` 开始的所有参数
  - `{args[n:m]}` 在 `n` 和 `m` 之间的所有参数

  对于插入多个词元的情况，占位符（例如上述的 `{args[:]}`）**必须** 是一个独立的 [词元](/docs/caddyfile/concepts#tokens-and-quotes)，不能作为其他词元的一部分。换句话说，他必须左右都是空格，而且不能被引号包裹。

  请注意，在 v2.7.0 之前，语法为 `{args.N}` 但这种形式已被弃用，取而代之的是上面更灵活的语法。

<h2 id="examples">
	示例
</h2>

引入 sites-enabled 文件夹中所有的文件（隐藏文件除外）：

```caddy-d
import sites-enabled/*
```

引入引用片段，使用参数设置 CORS 头：

```caddy
(cors) {
	@origin header Origin {args[0]}
	header @origin Access-Control-Allow-Origin "{args[0]}"
	header @origin Access-Control-Allow-Methods "OPTIONS,HEAD,GET,POST,PUT,PATCH,DELETE"
}

example.com {
	import cors example.com
}
```

引入引用片段，将一组上游服务代理作为参数传入：

```caddy
(https-proxy) {
	reverse_proxy {args[:]} {
		transport http {
			tls
		}
	}
}

example.com {
	import https-proxy 10.0.0.1 10.0.0.2 10.0.0.3
}
```

引入引用片段，创建以第一个参数为重写规则前缀的反向代理：

```caddy
(proxy-rewrite) {
	rewrite * {args[0]}{uri}
	reverse_proxy {args[1:]}
}

example.com {
	import proxy-rewrite /api 10.0.0.1 10.0.0.2 10.0.0.3
}
```
