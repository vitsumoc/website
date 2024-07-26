---
title: Caddyfile
---

# Caddyfile

**Caddyfile** 是一个方便人类维护的配置文件格式，大部分人都会选择使用 Caddyfile 因为他便于阅读、便于编辑、便于理解，而且几乎能胜任所有的应用场景。

Caddyfile 看起来是这样：

```caddy
example.com {
	root * /var/www/wordpress
	encode gzip
	php_fastcgi unix//run/php/php-version-fpm.sock
	file_server
}
```

（这是一个真实的，具备生产条件的 Caddyfile，为 WordPress 网站提供了完全的 HTTPS 能力。）

最基础的配置思路就是，先输入您网站的地址，随后列举您网站需要的功能。[查看更多示例](/docs/caddyfile/patterns)

## Menu

- #### [快速开始](/docs/zh/quick-starts/caddyfile)
  接触并熟悉 Caddyfile
- #### [教程](/docs/zh/caddyfile-tutorial)
  学习使用 Caddyfile 执行各种常见的操作
- #### [概念](/docs/zh/caddyfile/concepts)
  必读！Caddyfile 的结构、站点地址、匹配器、占位符，等等
- #### [指令](/docs/zh/caddyfile/directives)
  位于行首的关键字，用来赋予站点能力
- #### [请求匹配器](/docs/zh/caddyfile/matchers)
  在指令中应用匹配器，匹配特定请求
- #### [全局选项](/docs/zh/caddyfile/options)
  对于全局生效，而非针对特定站点生效的设置
- #### [常见模板](/docs/zh/caddyfile/patterns)
  常见场景示例
<!-- - #### [Caddyfile specification](/docs/caddyfile/spec) TODO: Finish this -->

## Note

Caddyfile 是 Caddy 中受支持的 [配置适配器](/docs/config-adapters) 类型之一。在手动配置 Caddy 服务器时是最佳选择，但他的能力范围、适应性和可编程能力略逊于 Caddy 的 [原生 JSON 格式](/docs/json/)。如果您的 Caddy 服务通常自动化配置/自动化部署，您更应该考虑使用 JSON 格式搭配 Caddy 的 [API](/docs/api)。（您当然也可以将 Caddyfile 和 API 结合使用，但相比于 JSON 方式，这种方式会略微受限。）
