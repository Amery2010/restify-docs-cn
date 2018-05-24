### 目录

-   [createServer](#createserver)
-   [Server](#server)
    -   [listen](#listen)
    -   [close](#close)
    -   [get](#get)
    -   [head](#head)
    -   [post](#post)
    -   [put](#put)
    -   [patch](#patch)
    -   [del](#del)
    -   [opts](#opts)
    -   [pre](#pre)
    -   [use](#use)
    -   [param](#param)
    -   [rm](#rm)
    -   [address](#address)
    -   [inflightRequests](#inflightrequests)
    -   [debugInfo](#debuginfo)
    -   [toString](#tostring)
-   [事件](#事件)
-   [错误](#错误)
-   [类型](#类型)
    -   [Server~methodOpts](#servermethodopts)

---

## createServer

restify 服务器对象时您为传入的请求注册路由和处理程序的主要接口。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象
    -   `options.name` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 服务器的名称。(可选，默认为 `"restify"`)
    -   `options.dtrace` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 启用 DTrace 支持。(可选，默认为 `false`)
    -   `options.router` **Router** 路由器。(可选，默认为 `newRouter(opts)`)
    -   `options.log` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** [bunyan](https://github.com/trentm/node-bunyan) 实例。(可选，默认为 `bunyan.createLogger(options.name||"restify")`)
    -   `options.url` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 一旦 listen() 被调用，这会被填入服务器的运行地址。
    -   `options.certificate` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Buffer](https://nodejs.org/api/buffer.html))?** 如果要创建 HTTPS 服务器，请传入 PEM 编码的证书和密钥。
    -   `options.key` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Buffer](https://nodejs.org/api/buffer.html))?** 如果要创建 HTTPS 服务器，请传入 PEM 编码的证书和密钥。
    -   `options.formatters` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 为 `res.send()` 自定义响应格式化程序。
    -   `options.handleUncaughtExceptions` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 当为 `true` 时，restify 将使用一个域来捕获和响应处理程序堆栈中发生的任何未捕获的异常。[bunyan](https://github.com/trentm/node-bunyan) 实例。响应报头，默认为 `restify`。传递空字符串可以取消设置的报头。带有显著的负面性能影响。(可选，默认为 `false`)
    -   `options.spdy` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 可以被 [node-spdy](https://github.com/indutny/node-spdy) 接受的任何选项参数。
    -   `options.http2` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 可以被  [http2.createSecureServer](https://nodejs.org/api/http2.html) 接受的任何选项参数。
    -   `options.handleUpgrades` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 挂钩 Node.js HTTP 服务器的 `upgrade` 事件，通过常规请求处理链推送 `Connection: Upgrade` 请求。(可选，默认为 `false`)
    -   `options.httpsServerOptions` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 可以被 [Node.js https 服务器](http://nodejs.org/api/https.html#https_https) 接受的任何选项参数。如果提供了该参数，则以下 restify 服务器参数将被忽略：spdy、ca、certificate、key、passphrase、rejectUnauthorized、requestCert 和 ciphers；但这些参数都可以在 httpsServerOptions 上指定。
    -   `options.onceNext` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 防止调用 next() 多次(可选，默认为 `false`)
    -   `options.strictNext` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 当 next() 被多次调用时抛出错误，并启用 onceNext 选项。(可选，默认为 `false`)
    -   `options.ignoreTrailingSlash` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 忽略路径上的尾部斜杠(可选，默认为 `false`)

**例子**

```javascript
var restify = require('restify');
var server = restify.createServer();

server.listen(8080, function () {
  console.log('ready on %s', server.url);
});
```

返回 **[Server](#server)** 服务器

## Server

创建一个新的服务器。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 一个选项对象
    -   `options.name` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 服务器的名称。(可选，默认为 `"restify"`)
    -   `options.dtrace` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 启用 DTrace 支持。(可选，默认为 `false`)
    -   `options.router` **Router** 路由器
    -   `options.log` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** [bunyan](https://github.com/trentm/node-bunyan) 实例。
    -   `options.url` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 一旦 listen() 被调用，这会被填入服务器的运行地址。
    -   `options.certificate` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Buffer](https://nodejs.org/api/buffer.html))?** 如果要创建 HTTPS 服务器，请传入 PEM 编码的证书和密钥。
    -   `options.key` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Buffer](https://nodejs.org/api/buffer.html))?** 如果要创建 HTTPS 服务器，请传入 PEM 编码的证书和密钥。
    -   `options.formatters` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 为 `res.send()` 自定义响应格式化程序。
    -   `options.handleUncaughtExceptions` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 当为 `true` 时，restify 将使用一个域来捕获和响应处理程序堆栈中发生的任何未捕获的异常。[bunyan](https://github.com/trentm/node-bunyan) 实例。响应报头，默认为 `restify`。传递空字符串可以取消设置的报头。带有显著的负面性能影响。(可选，默认为 `false`)
    -   `options.spdy` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 可以被 [node-spdy](https://github.com/indutny/node-spdy) 接受的任何选项参数。
    -   `options.http2` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 可以被  [http2.createSecureServer](https://nodejs.org/api/http2.html) 接受的任何选项参数。
    -   `options.handleUpgrades` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 挂钩 Node.js HTTP 服务器的 `upgrade` 事件，通过常规请求处理链推送 `Connection: Upgrade` 请求。(可选，默认为 `false`)
    -   `options.httpsServerOptions` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 可以被 [Node.js https 服务器](http://nodejs.org/api/https.html#https_https) 接受的任何选项参数。如果提供了该参数，则以下 restify 服务器参数将被忽略：spdy、ca、certificate、key、passphrase、rejectUnauthorized、requestCert 和 ciphers；但这些参数都可以在 httpsServerOptions 上指定。
    -   `options.onceNext` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 防止调用 next() 多次(可选，默认为 `false`)
    -   `options.strictNext` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 当 next() 被多次调用时抛出错误，并启用 onceNext 选项。(可选，默认为 `false`)
    -   `options.noWriteContinue` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 当代理时阻止 `server.on('checkContinue')` 中的 `res.writeContinue()`。(可选，默认为 `false`)
    -   `options.ignoreTrailingSlash` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 忽略路径上的尾部斜杠(可选，默认为 `false`)

**例子**

```javascript
var restify = require('restify');
var server = restify.createServer();

server.listen(8080, function () {
  console.log('ready on %s', server.url);
});
```

### listen

获取服务器并监听。封装了 Node.js 的 [listen()](http://nodejs.org/docs/latest/api/net.html#net_server_listen_path_callback)。

**参数**

-   `port` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 端口
-   `host` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** 主机
-   `callback` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 可选，当监听时接收通知。

**例子**

_您可以这样来调用：_

```javascript
server.listen(80)
server.listen(80, '127.0.0.1')
server.listen('/tmp/server.sock')
```

-   抛出 **[TypeError](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)** 

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 无返回值

### close

关闭此服务器，并在完成时调用回调（可选）。封装了 Node.js 的 [close()](http://nodejs.org/docs/latest/api/net.html#net_event_close)。

**参数**

-   `callback` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 回调完成后调用

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 无返回值

### get

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

**例子**

```javascript
server.get('/', function (req, res, next) {
  res.send({ hello: 'world' });
  next();
});
```

返回 **Route** 新创建的路由。

### head

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route** 新创建的路由。

### post

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `post` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route** 新创建的路由。

### put

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `put` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route** 新创建的路由。

### patch

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `patch` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route** 新创建的路由。

### del

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route** 新创建的路由。

### opts

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route** 新创建的路由。

### pre

让您挂钩在任何路由找到 _之前_ 运行。这使您有机会截获请求并修改路由所依赖的报头等。请注意，req.params _不会_ 被设置。

**参数**

-   `handler` **...([Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array))** 允许您为所有运行的路由添加处理程序。发生路由 _之前_。如果您需要的话，这给了您改变请求报头或类似的东西的钩子。请注意，`req.params` 会是 undefined，因为它是在路由 _之后_ 填充的。接受单个函数或一组函数。可变数量的处理函数嵌套数组。

**例子**

```javascript
server.pre(function(req, res, next) {
  req.headers.accept = 'application/json';
  return next();
});
```

_例如，`pre()` 可用于删除 URL中的重复斜杠_

```javascript
server.pre(restify.pre.dedupeSlashes());
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 自身。

### use

允许您为所有运行的路由添加处理程序。请注意，通过 `use()` 添加的处理程序只有在路由器找到匹配路由之后才会运行。如果找不到匹配，这些处理程序将永远不会运行。接受单个函数或一组函数。

您可以传递任何组合函数或一组函数。

**参数**

-   `handler` **...([Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array))** 可变数量的处理函数
    -   and/or 可变数量的处理函数嵌套数组

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 自身。

### param

-   **详见：<http://expressjs.com/guide.html#route-param%20pre-conditions>**

Express.js 路由参数前提条件所提供功能的最小端口。

这基本上是捎带在 `server.use` 方法上。它附加了一个新的中间件函数，只有在 req.params 中存在指定的参数时才会触发该函数。

公开一个 API：

```javascript
server.param("user", function (req, res, next) {
  // 在这里加载用户的信息，请确保调用 next()
});
```

**参数**

-   `name` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 要响应的 URL 参数的名称
-   `fn` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 要执行的中间件函数

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 自身。

### rm

从服务器中删除路由。您传入了从挂载回调中获取的路由 'blob'。

**参数**

-   `routeName` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 路由名称。
-   在糟糕的输入时 **[TypeError](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)** 抛出错误。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 如果路由被删除，则为 true；否则为 false。

### address

返回服务器地址。封装了 Node.js 的 [close()](http://nodejs.org/docs/latest/api/net.html#net_event_close)。

**例子**

```javascript
server.address()
```

_输出：_

```javascript
{ address: '::', family: 'IPv6', port: 8080 }
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 服务器地址。

### inflightRequests

返回服务器当前正在处理尚未返回结果的请求的数量。

返回 **[number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 正在进行的请求数量。

### debugInfo

返回与服务器相关的调试信息。

**例子**

```javascript
server.getDebugInfo()
```

_输出：_

```javascript
{
  routes: [
    {
      name: 'get',
      method: 'get',
      input: '/',
      compiledRegex: /^[\/]*$/,
      compiledUrlParams: null,
      handlers: [Array]
    }
  ],
  server: {
    formatters: {
      'application/javascript': [Function: formatJSONP],
      'application/json': [Function: formatJSON],
      'text/plain': [Function: formatText],
      'application/octet-stream': [Function: formatBinary]
    },
    address: '::',
    port: 8080,
    inflightRequests: 0,
    pre: [],
    use: [ 'parseQueryString', '_jsonp' ],
    after: []
  }
}
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 调试信息。

### toString

toString()，易于服务器读取和输出。

**例子**

```javascript
server.toString()
```

_输出：_

```javascript
Accepts: application/json, text/plain, application/octet-stream,
application/javascript
Name: restify
Pre: []
Router: RestifyRouter:
	DELETE: []
	GET: [get]
	HEAD: []
	OPTIONS: []
	PATCH: []
	POST: []
	PUT: []

Routes:
	get: [parseQueryString, _jsonp, function]
Secure: false
Url: http://[::]:8080
Version:
```

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 字符串化服务器。

## 事件

除了触发所有来自 Node.js 中 [http.Server](http://nodejs.org/docs/latest/api/http.html#http_class_http_server) 的事件之外，restify 服务器还会触发大量附加事件，这些事件使构建 REST 和 Web 应用程序变得更加容易。

### restifyError

这个事件是在所有错误事件触发后作为通用捕获而存在。建议使用特定的错误事件来处理特定的错误，但此事件可用于度量或日志记录。如果您将其与其他错误事件一起使用，则会首先触发最具体的事件，然后才是此事件：

```javascript
server.get('/', function(req, res, next) {
  return next(new InternalServerError('boom'));
});

server.on('InternalServer', function(req, res, err, callback) {
  // 这将首先被触发，因为它是最有关联性的侦听器
  return callback();
});

server.on('restifyError', function(req, res, err, callback) {
  // 其次才会触发该事件
  return callback();
});
```

### after

当每个请求都已经被完全服务之后，会触发一个 `after` 事件。这个事件可以被用来处理审计日志和其他指标。请注意，刷新响应不一定与 `after` 事件相对应。在以下情况下，restify 会认为请求已经得到了全面服务：

1) 路由的处理程序链已经完全处理完成。
2) `next()` 返回了一个错误，并且针对该错误类型已经触发了相应的错误事件。

`after` 事件的函数签名如下：

```javascript
function(req, res, route, error) { }
```

-   `req` - 请求对象
-   `res` - 响应对象
-   `route` - 为请求提供服务的路由对象
-   `error` - 传递给 `next()` 的错误（如果存在的话）

请注意，当服务器自动响应 NotFound / MethodNotAllowed / VersionNotAllowed 时，该事件仍将被触发。

### pre

在每个请求被路由之前，会触发一个 `pre` 事件。此事件可以用来挂载处理审核日志和其他指标。由于此事件在路由发生 _之前_ 触发，无论路由是否受到支持，它都会触发。例如，一个导致 `404` 的请求。

`pre` 事件的函数签名如下：

```javascript
function(req, res) {}
```

-   `req` - 请求对象
-   `res` - 响应对象

请注意，当服务器自动响应 NotFound / MethodNotAllowed / VersionNotAllowed 时，该事件仍将被触发。

### routed

`routed` 事件在路由器发送请求后触发，但在该路由器的处理程序运行之前。

`routed` 事件的函数签名如下：

```javascript
function(req, res, route) {}
```

-   `req` - 请求对象
-   `res` - 响应对象
-   `route` - 为请求提供服务的路由对象

请注意，如果有不可路由的请求进入，此事件 _不会_ 触发。例如，一个导致 `404` 的请求。

### uncaughtException

如果 restify 服务器是使用 `handleUncaughtExceptions: true`，restify 将利用 [domains](https://nodejs.org/api/domain.html) 来处理处理程序链中抛出的错误。抛出的错误是显式 `throw` 语句的结果，或者由于程序员的错误（如错字或空引用）导致的结果。这些抛出的错误被域捕获，并将通过这个事件发出。例如：

```javascript
server.get('/', function(req, res, next) {
  res.send(x);  // 这会导致 ReferenceError
  return next();
});

server.on('uncaughtException', function(req, res, route, err) {
  // 上面的错误对象将会导致该事件的触发：
  // ReferenceError: x is not defined
});
```

如果您监听了该事件，您 **必须** 向客户端发送回复。此行为与标准错误事件不同。如果你不听这个事件，restify 的默认行为是调用 `res.send()` 抛出错误。

`uncaughtException` 事件的函数签名如下：

```javascript
function(req, res, route, error) { }
```

-   `req` - 请求对象
-   `res` - 响应对象
-   `route` - 为请求提供服务的路由对象
-   `error` - 传递给 `next()` 的错误（如果存在的话）

### close

服务器关闭时发出。


## 错误

Restify 作为一等公民处理错误。当一个错误对象被传递给 `next()` 函数时，服务器对象会发出一个事件，并将该错误对象序列化发送到客户端。错误对象就是任何可以通过 `instanceof Error` 检测的对象。

在将错误对象发送给客户端之前，服务器将触发使用该错误名称的事件，但不包含名称字符串中的 `Error`。例如，给定一个  `InternalServerError`，服务器将发出一个 `InternalServer` 事件。这为创建日志记录和指标，或基于错误类型的有效载荷突变创造了机会。例如：

```javascript
var errs = require('restify-errors');

server.get('/', function(req, res, next) {
  return next(new errs.InternalServerError('boom!'));
});

server.on('InternalServer', function(req, res, err, callback) {
  // 在发送响应之前，将调用此侦听器，给与做指标捕捉或记录的机会。
  myMetrics.capture(err);
  // 调用回调来完成你的工作，使得服务器发出响应。
  return callback();
});
```

在错误事件的侦听器内部，如果有必要的话，也可以修改错误的序列化方法。为此，只需实现一个自定义的 `toString()` 或 `toJSON()`。根据内容类型和用于响应格式化程序，将使用其中之一。例如，下面给出的例子：

```javascript
server.on('restifyError', function(req, res, err, callback) {
  err.toJSON = function customToJSON() {
    return {
      name: err.name,
      message: err.message
    };
  };
  err.toString = function customToString() {
    return 'i just want a string';
  };
  return callback();
});
```

具有 `accept: application/json` 的请求将触发 `toJSON()` 序列化程序，而具有 `accept: text/plain` 的请求将触发 `toString()` 序列化程序。

请注意，错误侦听器的函数签名对于所有发出的错误事件都是相同的。签名如下：

```javascript
function(req, res, err, callback) { }
```

-   `req` - 请求对象
-   `res` - 响应对象
-   `err` - 错误对象
-   `callback` - 用以调用的回调函数

将此功能与 [restify-errors](https://github.com/restify/errors) 一起使用时，restify 会为所有基本的 http 错误发出事件：

-   `400` - `BadRequestError`
-   `401` - `UnauthorizedError`
-   `402` - `PaymentRequiredError`
-   `403` - `ForbiddenError`
-   `404` - `NotFoundError`
-   `405` - `MethodNotAllowedError`
-   `406` - `NotAcceptableError`
-   `407` - `ProxyAuthenticationRequiredError`
-   `408` - `RequestTimeoutError`
-   `409` - `ConflictError`
-   `410` - `GoneError`
-   `411` - `LengthRequiredError`
-   `412` - `PreconditionFailedError`
-   `413` - `RequestEntityTooLargeError`
-   `414` - `RequesturiTooLargeError`
-   `415` - `UnsupportedMediaTypeError`
-   `416` - `RangeNotSatisfiableError` (node >= 4)
-   `416` - `RequestedRangeNotSatisfiableError` (node 0.x)
-   `417` - `ExpectationFailedError`
-   `418` - `ImATeapotError`
-   `422` - `UnprocessableEntityError`
-   `423` - `LockedError`
-   `424` - `FailedDependencyError`
-   `425` - `UnorderedCollectionError`
-   `426` - `UpgradeRequiredError`
-   `428` - `PreconditionRequiredError`
-   `429` - `TooManyRequestsError`
-   `431` - `RequestHeaderFieldsTooLargeError`
-   `500` - `InternalServerError`
-   `501` - `NotImplementedError`
-   `502` - `BadGatewayError`
-   `503` - `ServiceUnavailableError`
-   `504` - `GatewayTimeoutError`
-   `505` - `HttpVersionNotSupportedError`
-   `506` - `VariantAlsoNegotiatesError`
-   `507` - `InsufficientStorageError`
-   `509` - `BandwidthLimitExceededError`
-   `510` - `NotExtendedError`
-   `511` - `NetworkAuthenticationRequiredError`

Restify 还会发出以下事件：

### NotFound

当一个客户端向一个不存在的 URL 发送请求时，restify 会发出该事件。请注意，restify 会检查此事件上的侦听器，如果没有，则使用默认的 404 处理程序进行响应。

### MethodNotAllowed

当一个客户端向一个存在的 URL 发送请求，但不是该请求的 HTTP 动词时，restify 会发出该事件。请注意，restify 会检查此事件上的侦听器，如果没有，则使用默认的 405 处理程序进行响应。

### VersionNotAllowed

当客户端向一个有效路由发送请求，但不匹配该路由上的版本时，restify 将发出该事件。请注意，restify 会检查此事件上的侦听器，如果没有，则使用默认的 400 处理程序进行响应。

### UnsupportedMediaType

当客户端向一个有效路由发送请求，但不匹配 `content-type` 时，restify 将发出该事件。请注意，restify 会检查此事件上的侦听器，如果没有，则使用默认的 415 处理程序进行响应。


## 类型

### Server~methodOpts

服务器的方法选项。

类型：([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Regexp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) \| [Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object))

**属性**

-   `name` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 路由的名称
-   `path` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 可以是任何被 [find-my-way](https://github.com/delvedor/find-my-way) 接受的字符串

**例子**

```javascript
// 一个静态路由
server.get('/foo', function(req, res, next) {});
// 一个参数化的路由
server.get('/foo/:bar', function(req, res, next) {});
// 一个正则表达式
server.get('/example/:file(^\\d+).png', function(req, res, next) {});
// 一个选项对象
server.get({
  path: '/foo',
}, function(req, res, next) {});
```
