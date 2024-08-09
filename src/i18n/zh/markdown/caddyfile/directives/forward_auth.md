---
title: forward_auth (Caddyfile directive)
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

	// Fix uri subdirective, gets parsed as matcher arg because of "uri" directive
	window.$('.k:contains("uri") + .nd')
		.each(function() {
			window.$(this)
				.removeClass('nd')
				.addClass('s')
				.text(window.$(this).text());
		});
});
</script>

# forward_auth

一个自用的指令，将请求复制到鉴权网关，鉴权网关可以决定继续处理，或是需要发送登录页。

- [语法](#syntax)
- [展开形式](#expanded-form)
- [示例](#examples)
  - [Authelia](#authelia)
  - [Tailscale](#tailscale)

Caddy 的 [`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy) 能够支持使用外部服务进行“请求预校验”，但本指令是专门针对需要身份认证的场景而定制的。本指令实际上只是一种更长、更通用配置（见下）的简化形式。

本指令可以重写 `uri`，并向配置的上游发送 `GET` 请求：
- 如果上游提供了 `2xx` 状态码的响应，则表示授权接入，并且会将 `copy_headers` 中的头部复制到请求中，随后继续处理。
- 或者，如果上游提供了其他的响应码，则将上游的响应复制并返回给客户端。这种响应通常会重定向到身份鉴权网关的登陆页面。

如果这种行为并不是您预期的，您可以以下方的 [展开形式](#expanded-form) 作为基础来自定义您需要的行为。

[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy) 中所有的子指令都支持使用，并且会在处理中传递到实际上的底层 `reverse_proxy` 处理器。

<h2 id="syntax">
	语法
</h2>

```caddy-d
forward_auth [<matcher>] [<upstreams...>] {
	uri          <to>
	copy_headers <fields...> {
		<fields...>
	}
}
```

- **&lt;upstreams...&gt;** 上游认证服务列表。

- **uri** 发送给上游认证服务的 URI，一般是鉴权网关的认证接口。

- **copy_headers** 当认证请求成功时，会将成功响应中的，符合此处配置的头部复制到原始请求中。

	头可以使用 `>` 来更名，例如 `Before>After`。

	您可以使用块来列举所有的头部，或者每行一个，随您的喜好。

由于本指令是 [`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy#syntax) 的一个自定义包装，您可以使用 `reverse_proxy` 中所有的可用的子指令来定制化此指令。

<h2 id="expanded-form">
	展开形式
</h2>

`forward_auth` 指令和下方的配置相同。[Authelia](https://www.authelia.com/) 鉴权网关在此配置下可以正常工作。如果您使用的并非此网关，您也可以在这个基础上修改自己需要的认证配置，而不一定非要使用 `forward_auth` 这种快捷方式。

```caddy-d
reverse_proxy <upstreams...> {
	# 总是改成 GET，这样原始请求的 body 就不会被消耗
	method GET

	# 把 URI 改写为鉴权网关的认证接口
	rewrite <to>

	# 原始请求的 URI 和方法已经被重写
	# 此处转发原始请求的 URI 和方法
	# 这是对 reverse_proxy 已经设置的其他 X-Forwarded-* 头的补充
	header_up X-Forwarded-Method {method}
	header_up X-Forwarded-Uri {uri}

	# 对于成功的响应，复制响应头
	@good status 2xx
	handle_response @good {
		request_header {
			# 例如，copy_headers 中的每个字段
			Remote-User {rp.header.Remote-User}
			Remote-Email {rp.header.Remote-Email}
		}
	}
}
```

<h2 id="examples">
	示例
</h2>

### Authelia

通过反向代理，在您的应用服务前使用 [Authelia](https://www.authelia.com/) 进行鉴权：

```caddy
# 为鉴权网关本身提供服务
auth.example.com {
	reverse_proxy authelia:9091
}

# APP 服务
app1.example.com {
	forward_auth authelia:9091 {
		uri /api/verify?rd=https://auth.example.com
		copy_headers Remote-User Remote-Groups Remote-Name Remote-Email
	}

	reverse_proxy app1:8080
}
```

有关更多信息，请参阅 [Authelia 与 Caddy 集成的文档](https://www.authelia.com/integration/proxies/caddy/)。

### Tailscale

使用 [Tailscale](https://tailscale.com/) 鉴权（当前名为 [`nginx-auth`](https://tailscale.com/blog/tailscale-auth-nginx/)，当然 Caddy 都可以支持），并使用 `copy_headers` 中的替换语法来 *重命名* 被复制的 HTTP 头（注意每个头配置中的 `>`）：

```caddy-d
forward_auth unix//run/tailscale.nginx-auth.sock {
	uri /auth
	header_up Remote-Addr {remote_host}
	header_up Remote-Port {remote_port}
	header_up Original-URI {uri}
	copy_headers {
		Tailscale-User>X-Webauth-User
		Tailscale-Name>X-Webauth-Name
		Tailscale-Login>X-Webauth-Login
		Tailscale-Tailnet>X-Webauth-Tailnet
		Tailscale-Profile-Picture>X-Webauth-Profile-Picture
	}
}
```
