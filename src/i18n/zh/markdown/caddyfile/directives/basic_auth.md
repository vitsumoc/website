---
title: basic_auth (Caddyfile directive)
---

# basic_auth

启用 HTTP 基本身份验证，可通过用户名和密码保护目录和文件。

**请注意，普通 HTTP + 基本身份验证的组合并不安全。** 请谨慎决定使用 HTTP 基本身份验证保护哪些内容。

当用户请求受保护的资源时，如果用户尚未提供用户名和密码，浏览器将提示用户输入用户名和密码。如果 Authorization 头中存在正确的凭据，服务器将授予对资源的访问权限。如果没有 Authorization 头或凭据不正确，服务器将响应 HTTP 401 Unauthorized。

Caddy 配置不接受明文密码，您 **必须** 在配置前先对密码进行哈希。您可以使用 [`caddy hash-password`](/docs/command-line#caddy-hash-password) 命令完成这一操作。

认证成功后，`{http.auth.user.id}` 将变得可用，其中包含着用户的 username。

在 v2.8.0 版本之前，这条指令的名称是 `basicauth`，但是为了指令命名的一致性，现在改为了 `basic_auth`。

<h2 id="syntax">
	语法
</h2>

```caddy-d
basic_auth [<matcher>] [<hash_algorithm> [<realm>]] {
	<username> <hashed_password>
	...
}
```

- **&lt;hash_algorithm&gt;** 此配置中密码使用的哈希（或 KDF） 算法的名称，默认为：`bcrypt`

- **&lt;realm&gt;** 自定义域名。

- **&lt;username&gt;** 用户名或用户 ID。

- **&lt;hashed_password&gt;** 密码的哈希值。

<h2 id="examples">
	示例
</h2>

对访问 `example.com` 的所有请求进行身份验证：

```caddy
example.com {
	basic_auth {
		# Username "Bob", password "hiccup"
		Bob $2a$14$Zkx19XLiW6VYouLHR5NmfOFU0z2GTNmpkT/5qqR7hx4IjWJPDhjvG
	}
	respond "Welcome, {http.auth.user.id}" 200
}
```

保护 `/secret/` 中的文件，只有 Bob 可以访问（其他路径公开）：

```caddy
example.com {
	root * /srv

	basic_auth /secret/* {
		# Username "Bob", password "hiccup"
		Bob $2a$14$Zkx19XLiW6VYouLHR5NmfOFU0z2GTNmpkT/5qqR7hx4IjWJPDhjvG
	}

	file_server
}
```

