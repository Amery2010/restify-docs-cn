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
-   [Events](#events)
-   [Errors](#errors)
-   [Types](#types)
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
    -   `options.formatters` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 为 `res.send()` 自定义响应格式化器。
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
    -   `options.formatters` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 为 `res.send()` 自定义响应格式化器。
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

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)**，无返回值

### close

关闭此服务器，并在完成时调用回调（可选）。封装了 Node.js 的 [close()](http://nodejs.org/docs/latest/api/net.html#net_event_close)。

**参数**

-   `callback` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 回调完成后调用

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)**，无返回值

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

返回 **Route**，新创建的路由。

### head

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route**，新创建的路由。

### post

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `post` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route**，新创建的路由。

### put

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `put` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route**，新创建的路由。

### patch

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `patch` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route**，新创建的路由。

### del

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route**，新创建的路由。

### opts

根据此 HTTP 动词在给定路径上挂载一个链

**参数**

-   `opts` **[Server~methodOpts](#servermethodopts)** 如果是字符串，则处理 URL。如果是选项对象，则至少要处理 URL。

返回 **Route**，新创建的路由。

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

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)**，返回自身。

### use

允许您为所有运行的路由添加处理程序。请注意，通过 `use()` 添加的处理程序只有在路由器找到匹配路由之后才会运行。如果找不到匹配，这些处理程序将永远不会运行。接受单个函数或一组函数。

您可以传递任何组合函数或一组函数。

**参数**

-   `handler` **...([Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array))** 可变数量的处理函数
    -   and/or 可变数量的处理函数嵌套数组

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)**，返回自身。

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

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)**，返回自身。

### rm

从服务器中删除路由。您传入了从挂载回调中获取的路由 'blob'。

**参数**

-   `routeName` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 路由名称。
-   在糟糕的输入时 **[TypeError](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)** 抛出错误。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)**，如果路由被删除，则为 true；否则为 false。

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

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)**，服务器地址。

### inflightRequests

返回服务器当前正在处理尚未返回结果的请求的数量。

返回 **[number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)**，正在进行的请求数量。

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

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)**，调试信息。

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

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)**，字符串化服务器。

## Events

In additional to emitting all the events from node's
[http.Server](http://nodejs.org/docs/latest/api/http.html#http_class_http_server),
restify servers also emit a number of additional events that make building REST
and web applications much easier.

### restifyError

This event is emitted following all error events as a generic catch all. It is
recommended to use specific error events to handle specific errors, but this
event can be useful for metrics or logging. If you use this in conjunction with
other error events, the most specific event will be fired first, followed by
this one:

```js
server.get('/', function(req, res, next) {
  return next(new InternalServerError('boom'));
});

server.on('InternalServer', function(req, res, err, callback) {
  // this will get fired first, as it's the most relevant listener
  return callback();
});

server.on('restifyError', function(req, res, err, callback) {
  // this is fired second.
  return callback();
});
```

### after

After each request has been fully serviced, an `after` event is fired. This
event can be hooked into to handle audit logs and other metrics. Note that
flushing a response does not necessarily correspond with an `after` event.
restify considers a request to be fully serviced when either:

1) The handler chain for a route has been fully completed
2) An error was returned to `next()`, and the corresponding error events have
   been fired for that error type

The signature is for the after event is as follows:

```js
function(req, res, route, error) { }
```

-   `req` - the request object
-   `res` - the response object
-   `route` - the route object that serviced the request
-   `error` - the error passed to `next()`, if applicable

Note that when the server automatically responds with a
NotFound/MethodNotAllowed/VersionNotAllowed, this event will still be fired.

### pre

Before each request has been routed, a `pre` event is fired. This event can be
hooked into handle audit logs and other metrics. Since this event fires
_before_ routing has occured, it will fire regardless of whether the route is
supported or not, e.g. requests that result in a `404`.

The signature for the `pre` event is as follows:

```js
function(req, res) {}
```

-   `req` - the request object
-   `res` - the response object

Note that when the server automatically responds with a
NotFound/MethodNotAllowed/VersionNotAllowed, this event will still be fired.

### routed

A `routed` event is fired after a request has been routed by the router, but
before handlers specific to that route has run.

The signature for the `routed` event is as follows:

```js
function(req, res, route) {}
```

-   `req` - the request object
-   `res` - the response object
-   `route` - the route object that serviced the request

Note that this event will _not_ fire if a requests comes in that are not
routable, i.e. one that would result in a `404`.

### uncaughtException

If the restify server was created with `handleUncaughtExceptions: true`,
restify will leverage [domains](https://nodejs.org/api/domain.html) to handle
thrown errors in the handler chain. Thrown errors are a result of an explicit
`throw` statement, or as a result of programmer errors like a typo or a null
ref. These thrown errors are caught by the domain, and will be emitted via this
event. For example:

```js
server.get('/', function(req, res, next) {
    res.send(x);  // this will cause a ReferenceError
    return next();
});

server.on('uncaughtException', function(req, res, route, err) {
    // this event will be fired, with the error object from above:
    // ReferenceError: x is not defined
});
```

If you listen to this event, you **must** send a response to the client. This
behavior is different from the standard error events. If you do not listen to
this event, restify's default behavior is to call `res.send()` with the error
that was thrown.

The signature is for the after event is as follows:

```js
function(req, res, route, error) { }
```

-   `req` - the request object
-   `res` - the response object
-   `route` - the route object that serviced the request
-   `error` - the error passed to `next()`, if applicable

### close

Emitted when the server closes.


## Errors

Restify handles errors as first class citizens. When an error object is passed
to the `next()` function, an event is emitted on the server object, and the
error object will be serialized and sent to the client. An error object is any
object that passes an `instanceof Error` check.

Before the error object is sent to the client, the server will fire an event
using the name of the error, without the `Error` part of the name. For example,
given an `InternalServerError`, the server will emit an `InternalServer` event.
This creates opportunities to do logging, metrics, or payload mutation based on
the type of error. For example:

```js
var errs = require('restify-errors');

server.get('/', function(req, res, next) {
    return next(new errs.InternalServerError('boom!'));
});

server.on('InternalServer', function(req, res, err, callback) {
    // before the response is sent, this listener will be invoked, allowing
    // opportunities to do metrics capturing or logging.
    myMetrics.capture(err);
    // invoke the callback to complete your work, and the server will send out
    // a response.
    return callback();
});
```

Inside the error event listener, it is also possible to change the serialization
method of the error if desired. To do so, simply implement a custom
`toString()` or `toJSON()`. Depending on the content-type and formatter being
used for the response, one of the two serializers will be used. For example,
given the folllwing example:

```js
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

A request with an `accept: application/json` will trigger the `toJSON()`
serializer, while a request with `accept: text/plain` will trigger the
`toString()` serializer.

Note that the function signature for the error listener is identical for all
emitted error events. The signature is as follows:

```js
function(req, res, err, callback) { }
```

-   `req` - the request object
-   `res` - the response object
-   `err` - the error object
-   `callback` - a callback function to invoke

When using this feature in conjunction with
[restify-errors](https://github.com/restify/errors), restify will emit events
for all of the basic http errors:

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

Restify will also emit the following events:

### NotFound

When a client request is sent for a URL that does not exist, restify
will emit this event. Note that restify checks for listeners on this
event, and if there are none, responds with a default 404 handler.

### MethodNotAllowed

When a client request is sent for a URL that exists, but not for the requested
HTTP verb, restify will emit this event. Note that restify checks for listeners
on this event, and if there are none, responds with a default 405 handler.

### VersionNotAllowed

When a client request is sent for a route that exists, but does not
match the version(s) on those routes, restify will emit this
event. Note that restify checks for listeners on this event, and if
there are none, responds with a default 400 handler.

### UnsupportedMediaType

When a client request is sent for a route that exist, but has a `content-type`
mismatch, restify will emit this event. Note that restify checks for listeners
on this event, and if there are none, responds with a default 415 handler.


## Types

### Server~methodOpts

Server method opts

Type: ([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Regexp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) \| [Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object))

**Properties**

-   `name` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** a name for the route
-   `path` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** can be any String accepted by
    [find-my-way](https://github.com/delvedor/find-my-way)

**Examples**

```javascript
// a static route
server.get('/foo', function(req, res, next) {});
// a parameterized route
server.get('/foo/:bar', function(req, res, next) {});
// a regular expression
server.get('/example/:file(^\\d+).png', function(req, res, next) {});
// an options object
server.get({
    path: '/foo',
}, function(req, res, next) {});
```
