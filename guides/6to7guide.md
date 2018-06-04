---
title: 将6.x迁移到7.x迁移
permalink: /docs/6to7/
---

## 介绍

restify `7.x` 具有全新的路由器和中间件逻辑，可为您的应用带来显着的性能提升。从 `v7.0.0` 开始，restify 使用基于 [find-my-way](https://github.com/delvedor/find-my-way) 包的 Radix Tree 作为路由器后端。

## 突破性改变

### 服务器返回 `RequestCloseError` 代替 `RequestAbortedError`

在由于某种原因客户端终止请求的情况下，服务器返回 `RequestCloseError` 而不是 `RequestAbortedError`。

新版本的 restify 永远不会返回 `RequestAbortedError`。

### 非严格的路由消失了

选项 `strictRouting` 被删除 `createServer({ strictRouting: false })`。严格路由是新的默认路由。

### 最后路径的尾部斜线

`/path` 和 `/path/` 在 restify `v7.x` 中是不一相同的。
如果您不想区分它们，请使用 `ignoreTrailingSlash: true` 服务器选项。

### 路径必须以 `/` 开头

在 restify 7.x 中路径必须以 `/` 开头。
例如 `server.get('foo')` 无效，请将其改为 `server.get('/foo')`。

如果您使用 [enroute](https://github.com/restify/enroute) ，请确保您已将其更新至最新版本。

### 路由器路径和通配符中不同的 `RegExp` 用法

restify 的新路由器后端 [find-my-way](https://github.com/delvedor/find-my-way) 支持有限的 RegExp。

#### 定义路径的指南

要注册**参数**路径，请在参数名称之前使用*冒号*。
**通配符**使用*星号*。
*请记住，静态路由始终位于参数和通配符之前。*

```javascript
// 参数
server.get('GET', '/example/:userId', (req, res, next) => {}))
server.get('GET', '/example/:userId/:secretToken', (req, res, next) => {}))

// 通配符
server.get('GET', '/example/*', (req, res, next) => {}))
```

正则表达式路由也受支持，但请注意，RegExp 非常耗性能！

```javascript
// 带 RegExp 的参数
server.get('GET', '/example/:file(^\\d+).png', () => {}))
```

RegExp 路径块需要在括号之间。

可以在同一对斜线（"/"）内定义多个参数。如：

```javascript
server.get('/example/near/:lat-:lng/radius/:r', (req, res, next) => {}))
```

*记住在这种情况下使用破折号（"-"）作为参数分隔符。*

最后，RegExp 可能有多个参数。

```javascript
server.get('/example/at/:hour(^\\d{2})h:minute(^\\d{2})m', (req, res, next) => {
  // req.params => { hour: 12, minute: 15 }
}))
```

在这种情况下，作为参数分隔符，它可以使用任何与正则表达式不匹配的字符。

有多个参数的路由可能会对性能产生负面影响，所以尽可能地使用单一参数的方式，特别是在应用程序的热门路径上的路由。

更多信息请参阅：https://github.com/delvedor/find-my-way

### 删除已弃用的 `next.ifError`

`next.ifError(err)` 不再可用。

### 默认情况下禁用 DTrace 探测

DTrace 探针会造成一些性能影响。虽然它有着良好的可观察性，但您可能根本用不到它。

### 可以调用 `next` 多次

之前的 `restify` 会自动阻止超过一次的 `next()` 调用。
在新版本中，默认情况下此行为处于禁用状态，但您可以使用 `onceNext` 属性来激活它。

`strictNext` 选项的行为不变。
这意味着 `strictNext` 会强制执行 `onceNext` 选项。

```javascript
var server = restify.createServer({ onceNext: true })
server.use(function (req, req, next) {
  next();
  next();
});
// -> 可用

var server = restify.createServer({ strictNext: true })
server.use(function (req, req, next) {
  next();
  next();
});
// -> 抛出错误
```

### 路由器版本和内容类型

`accept-version` 和 `accept` 基于条件的路由移动到了 `conditionalHandler` 插件中，请查阅文档或示例：

```javascript
var server = restify.createServer()

server.use(restify.plugins.conditionalHandler({
  contentType: 'application/json',
  version: '1.0.0'
  handler: function (req, res, next) {
    next();
  })
});

server.get('/hello/:name', restify.plugins.conditionalHandler([
  {
    version: '1.0.0',
    handler: function(req, res, next) { res.send('1.x') }
  },
  {
    version: ['1.5.0', '2.0.0'],
    handler: function(req, res, next) { res.send('1.5.x, 2.x') }
  },
  {
    version: '3.0.0',
    contentType: ['text/html', 'text/html']
    handler: function(req, res, next) { res.send('3.x, text') }
  },
  {
    version: '3.0.0',
    contentType: 'application/json'
    handler: function(req, res, next) { res.send('3.x, json') }
  }
]);

// 'accept-version': '^1.1.0' => 1.5.x, 2.x'
// 'accept-version': '3.x', 接受：'application/json' => '3.x, json'
```

### 当请求被刷新并完成最后的处理程序时触发 After 事件

在 7.x 中，当请求被刷新并完成最后的处理程序时触发 `after` 事件。

### 指标插件的延迟

在 7.x 中，指标插件的 `latency` 是在请求完全刷新时计算的。之前它是在最后一个处理程序完成时计算的。

为了兼容之前的使用案例，新时间被添加到指标插件中：
To address the previous use-cases, new timings were added to the metrics plugin:

 - `metrics.totalLatency` 请求被刷新并完成了所有的处理程序
 - `metrics.preLatency` pre 处理程序延迟
 - `metrics.useLatency` use 处理程序延迟
 - `metrics.routeLatency` route 处理程序延迟
