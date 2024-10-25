---
title: method (Caddyfile directive)
---

# method

修改请求中的 HTTP 方法。

## 语法

```caddy-d
method [<matcher>] <method>
```

- 会将 HTTP 请求的方法修改为 **&lt;method&gt;**。


## 示例

将所有匹配到 `/api` 的请求的方法修改为 `POST`：

```caddy-d
method /api* POST
```
