---
title: acme_server (Caddyfile directive)
---

# acme_server

嵌入式 [ACME协议](https://tools.ietf.org/html/rfc8555) 服务器，让 Caddy 实例为任何其他 ACME 兼容软件（包括其他 Caddy 实例）颁发证书。

启用时，匹配到 `/acme/*` 的请求都会由 ACME 服务器处理。

<h2 id="client-configuration">
	客户端配置
</h2>

使用默认 ACME 服务时，ACME 客户端需要将 `https://localhost/acme/local/directory` 设置为 ACME 接口。（`local` 是 Caddy 默认 CA 的 ID。）

<h2 id="syntax">
	语法
</h2>

```caddy-d
acme_server [<matcher>] {
	ca         <id>
	lifetime   <duration>
	resolvers  <resolvers...>
	challenges <challenges...>
	allow_wildcard_names
	allow {
		domains <domains...>
		ip_ranges <addresses...>
	}
	deny {
		domains <domains...>
		ip_ranges <addresses...>
	}
}
```

- **ca** 指证书签发机构的ID。默认值是 `local`，用来发布本地的自签名证书，对于大部分开发环境来说很常见。为了更广泛的使用，建议指定不同的 CA 以避免混淆。如果具有给定 ID 的 CA 尚不存在，则会创建它。请参阅 [PKI 全局选项](/docs/caddyfile/options#pki-options) 来配置备用 CA。

- **lifetime** (Default: `12h`) 是一个用来指定所颁发证书有效期的 [时间范围](/docs/conventions#durations)。这个值必须小于签名所用的 [中间证书](/docs/caddyfile/options#intermediate-lifetime) 的 lifetime。除非必要，否则不推荐修改这个值。

- **resolvers** 是在查找 TXT 记录以解决 ACME DNS 挑战时使用的 DNS 解析器的地址。支持使用 [网络地址](/docs/conventions#network-addresses)，默认为 UDP 53 端口。如果主机配置为 IP 地址，则直接拨号上游服务器。如果主机不是 IP 地址，则使用 Go 标准库的 [名称解析约定](https://golang.org/pkg/net/#hdr-Name_Resolution) 来解析地址。如果指定了多个解析器，会随机选择其中的一个。

- **challenges** 设置可用的挑战类型。如果未使用此指令或是使用此指令没有指定内容，则所有类型的挑战均可用。支持的值包括：http-01，tls-alpn-01，dns-01。

- **allow_wildcard_names** 允许使用通配符 SAN（主题备用名称）颁发证书。

- **allow**, **deny** 配置 `acme_server` 的操作策略。策略评估遵循 [此处](https://smallstep.com/docs/step-ca/policies/#policy-evaluation) Step-CA 描述的标准。

	- **domains** 设置根据策略评估标准允许或拒绝的主题域名。

	- **ip_ranges** 设置根据策略评估标准允许或拒绝的主题 IP 范围。

<h2 id="examples">
	示例
</h2>

要为域名为 `acme.example.com`，ID 为 `home` 的 ACME 服务器提供服务，使用通过 [`pki` 全局选项](/docs/caddyfile/options#pki-options) 自定义的 CA，并使用 `internal` 颁发者颁发自己的证书：

```caddy
{
	pki {
		ca home {
			name "My Home CA"
		}
	}
}

acme.example.com {
	tls {
		issuer internal {
			ca home
		}
	}
	acme_server {
		ca home
	}
}
```

如果您还有另一个 Caddy 服务器，它可以使用上面的 ACME 服务器来颁发自己的证书：

```caddy
{
	acme_ca https://acme.example.com/acme/home/directory
	acme_ca_root /path/to/home_ca_root.crt
}

example.com {
	respond "Hello, world!"
}
```
