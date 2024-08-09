---
title: fs (Caddyfile directive)
---

# fs

选择文件读写时使用的文件系统。

您可以连接到云端的远程文件系统，或是实现了文件系统接口的数据库，甚至是读取嵌入到 Caddy 程序内部的文件。

首先，您必须使用 [`filesystem` 全局选项](/docs/caddyfile/options#filesystem) 来声明带有名称的文件系统，随后您就可以使用本指令来选择使用哪个文件系统。

本指令通常与 [`filesystem` global option](/docs/caddyfile/options#filesystem) 共同使用，以提供静态文件，或是与 [`try_files` directive](try_files) 结合使用，根据文件存在的与否进行重写。通常还与 [`root` 指令](root) 一起使用来设置文件系统内的根路径。

<h2 id="syntax">
	语法
</h2>

```caddy-d
fs [<matcher>] <filesystem>
```

<h2 id="examples">
	示例
</h2>

使用名为 `foo` 的文件系统，假设有一个名为 `custom` 的模组并且需要认证：

```caddy
{
	filesystem foo custom {
		api_key abc123
	}
}

example.com {
	fs foo
	root /srv
	file_server
}
```

只使用 `foo` 中的镜像，其他内容来自默认文件系统：

```caddy
example.com {
	fs /images* foo
	root /srv
	file_server
}
```
