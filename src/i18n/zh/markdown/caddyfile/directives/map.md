---
title: map (Caddyfile directive)
---

# map

基于输入的值设置自定义占位符的值。

此指令将 source 值与输入侧进行比较，如果匹配成功，则将输出值应用于每个 destination。Destinations 部分都是占位符名称。还可以为每个 destination 指定默认输出值。

如未被使用，被映射的占位符不会参与计算，因此即使映射表很大，此指令依然保持高效。

<h2 id="syntax">
	语法
</h2>

```caddy-d
map [<matcher>] <source> <destinations...> {
	[~]<input> <outputs...>
	default    <defaults...>
}
```

- **&lt;source&gt;** 用来进行匹配的输入值，通常使用一个占位符。

- **&lt;destinations...&gt;** 将会被创建的，持有输出值的占位符。

- **&lt;input&gt;** 和输入值进行匹配的值，如使用 `~` 前缀，则被视为正则表达式。

- **&lt;outputs...&gt;** 在关联的占位符中需要输出的一个或多个值。第一个位置的输出值输出到第一个 destination，第二个位置的输出值输出到第二个 destination，以此类推。
  
  作为一个特例，Caddyfile 将 outputs 中的连字符（`-`）视为 null/nil。当您想要为某些 output 设置默认值，而为其他 output 使用非默认值时，这个功能非常有用。

  output 的值可能会进行类型转化，`true` 和 `false` 会被转化为 boolean 类型，数字值会被转化为整形或是浮点数。如果您不希望进行这种转化，您可以使用 [引号](/docs/caddyfile/concepts#tokens-and-quotes) 包裹 output 的值，这样他们就会保持为字符串格式。

  output 的数量不得超过 destination 的数量，然而为了方便起见，可以少于 destination 的数量，这种情况下缺失的 output 会被隐式的填充。

  如果再 input 中使用了正则表达式，则可以使用 `${group}` 的形式引用捕获组，其中的 `group` 是捕获组的名称或是数字。捕获组 `0` 表示正则表达式的所有匹配，`1` 表示第一个捕获组，`2` 表示第二个捕获组，以此类推。

- **&lt;default&gt;** 当没有任何 input 成功匹配时指定 output 的值。

<h2 id="examples">
	示例
</h2>

The following example demonstrates most aspects of this directive:

```caddy-d
map {host}                {my_placeholder}  {magic_number} {
	example.com           "some value"      3
	foo.example.com       "another value"
	~(.*)\.example\.com$  "${1} subdomain"  5

	~.*\.net$             -                 7
	~.*\.xyz$             -                 15

	default               "unknown domain"  42
}
```

This directive switches on the value of `{host}`, i.e. the domain name of the request.

- If the request is for `example.com`, set `{my_placeholder}` to `some value`, and `{magic_number}` to `3`.
- Else, if the request is for `foo.example.com`, set `{my_placeholder}` to `another value`, and let `{magic_number}` default to `42`.
- Else, if the request is for any subdomain of `example.com`, set `{my_placeholder}` to a string containing the value of the first regexp capture group, i.e the entire subdomain, and set `{magic_number}` to 5.
- Else, if the request is for any host that ends in `.net` or `.xyz`, set only `{magic_number}` to `7` or `15`, respectively. Leave `{my_placeholder}` unset.
- Else (for all other hosts), the default values will apply: `{my_placeholder}` will be set to `unknown domain` and `{magic_number}` will be set to `42`.
