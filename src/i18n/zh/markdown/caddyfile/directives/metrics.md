---
title: metrics (Caddyfile directive)
---

# metrics

配置一个 [Prometheus](https://prometheus.io/) 指标（metrics）接口，从而可以进行数据抓取。**您必须首先 [在全局选项中打开 metrics 功能](/docs/caddyfile/options#metrics)。**

请注意 [admin API](/docs/api) 也提供了一个 `/metrics` 接口，此接口是不可配置的，而且当管理 API 被禁用后是不可使用的。

指标（metrics）接口会按照 [Prometheus exposition format](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format) 的要求返回数据，或者，如果协商通过，也可以按照 [OpenMetrics exposition format](https://pkg.go.dev/github.com/prometheus/client_golang@v1.9.0/prometheus/promhttp#HandlerOpts) 格式（`application/openmetrics-text`）返回数据。

您可以参考 [通过 Prometheus 指标监控 Caddy](/docs/metrics)。

## 语法

```caddy-d
metrics [<matcher>] {
	disable_openmetrics
}
```

- **disable_openmetrics** 关闭 OpenMetrics 格式的协商。一般来说没必要关闭，除非是在进行一些 bug解析之类的工作。

## 示例

使用默认的 `/metrics` 路径提供指标数据：

```caddy-d
metrics /metrics
```

使用其他路径提供指标数据：

```caddy-d
metrics /foo/bar/baz
```

使用独立的子域名提供指标服务：

```caddy
metrics.example.com {
	metrics
}
```

关闭 OpenMetrics 协商：

```caddy-d
metrics /metrics {
	disable_openmetrics
}
```
