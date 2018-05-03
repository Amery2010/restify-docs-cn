# 快速开始

通过 restify，您可以简单快速地创建一个服务器。以下代码就是一个标准的响应服务器：

```js
var restify = require('restify');

function respond(req, res, next) {
  res.send('hello ' + req.params.name);
  next();
}

var server = restify.createServer();
server.get('/hello/:name', respond);
server.head('/hello/:name', respond);

server.listen(8080, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

试试下面的 curl 命令来感受一下 restify 的返回结果：

```sh
$ curl -is http://localhost:8080/hello/mark -H 'accept: text/plain'
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 10
Date: Mon, 31 Dec 2012 01:32:44 GMT
Connection: keep-alive

hello mark


$ curl -is http://localhost:8080/hello/mark
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 12
Date: Mon, 31 Dec 2012 01:33:33 GMT
Connection: keep-alive

"hello mark"


$ curl -is http://localhost:8080/hello/mark -X HEAD -H 'connection: close'
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 12
Date: Mon, 31 Dec 2012 01:42:07 GMT
Connection: close
```

请注意，默认情况下，curl 使用 `Connection: keep-alive`。为了使 HEAD 方法立即返回，您需要主动传递 `Connection: close`。

由于 curl 经常与 REST API 一起使用，因此 restify 的插件集里有一个插件专门用来解决 curl 中的这种特质。该插件检测用户代理是否是 curl，如果是的话，它会将连接头设置为 `close` 并移除 `Content-Length` 头。

```js
server.pre(restify.plugins.pre.userAgentConnection());
```


## Sinatra 风格的处理程序链

像许多其他基于 Node.js 的 REST 框架一样，restify 利用 Sinatra 风格的语法来定义路由和服务这些路由的函数处理程序：

```js
server.get('/', function(req, res, next) {
  res.send('home')
  return next();
});

server.post('/foo',
  function(req, res, next) {
    req.someData = 'foo';
    return next();
  },
  function(req, res, next) {
    res.send(req.someData);
    return next();
  }
);
```

在一个 restify 服务器中，有三种不同的处理程序链：

* `pre` - 在路由之前执行的处理程序链
* `use` - 在路由之后执行的处理程序链
* `{httpVerb}` - 对特定路由执行的处理程序链

以上三种处理程序链都可以接受单个函数，多个函数或一组函数。


## 通用预处理程序：server.pre()

`pre` 处理程序链在路由之前执行。这意味着这些处理程序将针对传入的请求执行，即便它是您并未注册的路由。这可以用于记录日志和执行指标或在路由之前清理传入的请求。

```js
// 在路由之前删除 URL 中重复的斜杠
server.pre(restify.plugins.dedupeSlashes());
```


## 通用处理程序：server.use()

`use` 处理程序链式在请求被路由选择服务之后执行的。通过 `use()` 方法附加的函数处理程序将针对所有路由运行。由于 restify 以注册顺序运行处理程序，确保在定义任何路由之前，您所有的 `use()` 调用都会发生。

```js
server.use(function(req, res, next) {
	console.warn('run for all routes!');
	return next();
});
```


## 使用 next()

当处理程序链的每个函数执行之后，您需要负责调用 `next()`。调用 `next()` 后将移动到链中的下一个函数。

与其他的 REST 框架不同，调用 `res.send()` 不会自动触发 `next()`。在许多应用程序中，在 `res.send()` 之后可能需要继续处理，因此刷新响应并不等同于完成请求。

在正常情况下，`next()` 通常不会使用任何参数。如果由于某种原因您想停止处理请求，您可以调用 `next(false)` 来停止处理请求：

```js
server.use([
  function(req, res, next) {
    if (someCondition) {
      res.send('done!');
      return next(false);
    }
    return next();
  },
  function(req, res, next) {
    // 如果 someCondition 为 true，则该处理程序永远不会执行
  }
]);
```

`next()` 也接受任何 `instanceof Error` 为 true 的对象，这将导致 restify 发送该错误对象作为对客户端的响应。可以从 Error 对象的 `statusCode` 属性推断出响应的状态码。如果找不到 `statusCode`，它将默认为 500。所以下面的代码片段会通过一个 http 500 发送一个序列化的错误给客户端：

```js
server.use(function(req, res, next) {
  return next(new Error('boom!'));
});
```
  return next(new NotFoundError('not here!'));

这会发送一个 http 404，因为 `NotFoundError` 构造函数为 `statusCode` 提供了一个 404 的值：

```js
server.use(function(req, res, next) {
});
```

用 Error 对象调用 `res.send()` 会产生类似的结果，这段代码会通过一个 http 500 发送一个序列化的错误给客户端：

```js
server.use(function(req, res, next) {
  res.send(new Error('boom!'));
  return next();
});
```

这两者的区别在于使用 Error 对象调用 `next()` 允许您利用服务器的 [EventEmitter](/api/server.md#errors)。这使您可以使用通用的处理程序来处理所有出现的错误类型。更多详情，请参见[错误处理](#错误处理)章节。

最后，您可以通过调用带有 Error 对象的 `next.ifError(err)` 来引起 restify 抛出错误，从而导致进程失败。如果您遇到无法处理的错误，需要您终止该进程时，这会非常有用。


## 路由

'basic' 模式下的 restify 路由行为与 express/sinatra 非常类似，都使用 HTTP 动词与参数化资源一起来确定要运行的处理程序链。在 `req.params` 中可以找到与指定占位符关联的值。这些值在传递给您之前会被 URL 编码。

```js
function send(req, res, next) {
  res.send('hello ' + req.params.name);
  return next();
}

server.post('/hello', function create(req, res, next) {
  res.send(201, Math.random().toString(36).substr(3, 8));
  return next();
});
server.put('/hello', send);
server.get('/hello/:name', send);
server.head('/hello/:name', send);
server.del('hello/:name', function rm(req, res, next) {
  res.send(204);
  return next();
});
```

您也可以传入 [RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 对象并通过 `req.params` 访问捕获组（不会以任何方式解析）：

```js
server.get(/^\/([a-zA-Z0-9_\.~-]+)\/(.*)/, function(req, res, next) {
  console.log(req.params[0]);
  console.log(req.params[1]);
  res.send(200);
  return next();
});
```

有一些像这样的请求：

```sh
$ curl localhost:8080/foo/my/cats/name/is/gandalf
```

会导致 `req.params[0]` 为 `foo`，`req.params[1]` 为 `my/cats/name/is/gandalf`。

路由可以被指定为以下任意 http 动词 - `del`、`get`、`head`、`opts`、`post`、`put` 和 `patch`。

```js
server.get(
  '/foo/:id',
  function(req, res, next) {
    console.log('Authenticate');
    return next();
  },
  function(req, res, next) {
    res.send(200);
    return next();
  }
);
```


### 超媒体

如果参数化路由是由字符串（而不是正则表达式）定义的，那么您可以从服务器中的其他位置渲染它。这对于链接到其他资源的 HTTP 响应非常有用，而不必在整个代码库中对 URL 进行硬编码。路径和查询字符串参数都可以适当地进行 URL 编码。

```js
server.get({name: 'city', path: '/cities/:slug'}, /* ... */);

// 在另一个路由中
res.send({
  country: 'Australia',
  // 通过指定路由名称和参数来呈现 URL
  capital: server.router.render('city', {slug: 'canberra'}, {details: true})
});
```

这将返回：

```js
{
  "country": "Australia",
  "capital": "/cities/canberra?details=true"
}
```


### 版本化路由

大多数的 REST API 倾向于需要版本控制，并且使用 `Accept-Version` 报头来支持 [semver](http://semver.org/) 版本化，这种方式与您指定 NPM 版本依赖相同：

```js
var restify = require('restify');

var server = restify.createServer();

function sendV1(req, res, next) {
  res.send('hello: ' + req.params.name);
  return next();
}

function sendV2(req, res, next) {
  res.send({hello: req.params.name});
  return next();
}

server.get('/hello/:name', restify.plugins.conditionalHandler([
  { version: '1.1.3', handler: sendV1 },
  { version: '2.0.0', handler: sendV2 }
]));

server.listen(8080);
```

试着输入：

```sh
$ curl -s localhost:8080/hello/mark
{"hello":"mark"}
$ curl -s -H 'accept-version: ~1' localhost:8080/hello/mark
"hello: mark"
$ curl -s -H 'accept-version: ~2' localhost:8080/hello/mark
{"hello":"mark"}
$ curl -s -H 'accept-version: ~3' localhost:8080/hello/mark | json
{
  "code": "InvalidVersion",
  "message": "~3 is not supported by GET /hello/mark"
}
```

在第一种情况下，我们根本没有指定 `Accept-Version` 报头，所以 restify 会像请求发送了 `*` 那样进行对待。不发送 `Accept` 报头意味着客户端遵照服务器的选择，Restify 会选择最匹配的路由。在第二种情况下，我们明确要求 V1，它让我们回应了版本1处理函数的响应，但那之后我们要求 V2 并返回 JSON。最后，我们要求了一个不存在的版本，这导致出现了错误。

您可以通过在服务器创建时传递版本字段来设置路由的默认版本。最后，您可以通过使用数组来支持多版本的 API：

```js
server.get('/hello/:name' restify.plugins.conditionalHandler([
  { version: ['2.0.0', '2.1.0', '2.2.0'], handler: sendV2 }
]));
```

在这种情况下，您可能需要了解更多的信息，例如原始请求的版本字符串是什么，以及支持版本数组的路由的匹配版本是什么。有两种方法可用于获得此信息：

```js
server.get('/version/test', restify.plugins.conditionalHandler([
  {
    version: ['2.0.0', '2.1.0', '2.2.0'],
    handler: function (req, res, next) {
      res.send(200, {
        requestedVersion: req.version(),
        matchedVersion: req.matchedVersion()
      });
      return next();
    }
  }
]));
```

输入该路由将得到以下响应：

```sh
$ curl -s -H 'accept-version: <2.2.0' localhost:8080/version/test | json
{
  "requestedVersion": "<2.2.0",
  "matchedVersion": "2.1.0"
}
```


## 升级请求

对于传入的包含 `Connection: Upgrade` 报头的 HTTP 请求，将由 Node.js HTTP 服务器区别化对待。如果您想要 restify 通过常规路由链来推送升级请求，则需要在创建服务器时启用 `handleUpgrades`。

要确定请求是否符合升级条件，可以通过检查是否存在 `res.claimUpgrade()`。该方法返回具有两个属性的对象：底层连接的 `socket` 和第一次收到的数据 `Buffer` 作为 `head`（可能为零长度）。

一旦调用了 `res.claimUpgrade()`，`res` 本身被标记为无法进一步使用的 HTTP 响应；那之后尝试 `send()` 或 `end()` 等都会导致抛出一个 `Error`。同样，如果 `res` 已经向客户端发送了少量部分的响应，`res.claimUpgrade()` 也会抛出 `Error`。升级和常规的 HTTP 响应行为在任何特定的连接上是相互排斥的。

使用升级机制，您可以使用像 [watershed](https://github.com/jclulow/node-watershed) 这样的库来协商 WebSockets 连接。例如：

```js
var ws = new Watershed();
server.get('/websocket/attach', function upgradeRoute(req, res, next) {
  if (!res.claimUpgrade) {
    next(new Error('Connection Must Upgrade For WebSockets'));
    return;
  }

  var upgrade = res.claimUpgrade();
  var shed = ws.accept(req, upgrade.socket, upgrade.head);
  shed.on('text', function(msg) {
    console.log('Received message from websocket client: ' + msg);
  });
  shed.send('hello there!');

  next(false);
});
```


## 内容协商

如果你正在使用 `res.send()`，restify 会通过查找最先注册的 `formatter` 定义来自动选择响应的内容类型。注意在上面的例子中我们没有定义任何格式化器，所以我们一直利用了 restify 附带 `application/json`、`text/plain` 和 `application/octet-stream` 格式化器的事实。您可以通过在服务器创建时传入内容类型 -> 解析器散列来添加其他格式化器到 restify：

```js
var server = restify.createServer({
  formatters: {
    'application/foo': function formatFoo(req, res, body) {
      if (body instanceof Error)
        return body.stack;

      if (Buffer.isBuffer(body))
        return body.toString('base64');

      return util.inspect(body);
    }
  }
});
```

如果无法协商内容类型，则 restify 将默认使用 `application/octet-stream` 格式化器。例如，尝试发送包含未定义的格式化器的内容类型：

```js
server.get('/foo', function(req, res, next) {
  res.setHeader('content-type', 'text/css');
  res.send('hi');
  return next();
});
```

将导致响应的内容类型为 `application/octet-stream`：

```sh
$ curl -i localhost:3000/
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 2
Date: Thu, 02 Jun 2016 06:50:54 GMT
Connection: keep-alive
```

正如前文所述，restify 为 `json`、`text` 和 `binary` 附带了内置的格式化器。当您覆盖或附加格式化器时，“优先级”可能会发生改变；为了确保优先级设置为您想要的，您应该在您的格式化器定义中设置一个 `q-value`，这将确保按照您想要的方式进行排序：

```js
restify.createServer({
  formatters: {
    'application/foo; q=0.9': function formatFoo(req, res, body) {
      if (body instanceof Error)
        return body.stack;

      if (Buffer.isBuffer(body))
        return body.toString('base64');

      return util.inspect(body);
    }
  }
});
```

restify 响应对象保留了 Node.js [ServerResponse](http://nodejs.org/docs/latest/api/http.html#http.ServerResponse) 所有的“原始”方法。

```js
var body = 'hello world';
res.writeHead(200, {
  'Content-Length': Buffer.byteLength(body),
  'Content-Type': 'text/plain'
});
res.write(body);
res.end();
```

## 错误处理

您通常想要以相同的方式处理错误条件。例如，您可能想要给所有的 `InternalServerErrors` 返回 500 页面。在这种情况下，您可以为 `InternalServer` 错误事件添加一个侦听器，该错误事件在遇到此错误时会通过 restify 作为 `next(error)` 语句的一部分始终触发。这使您可以在服务器中以相同的方式处理所有同一类的错误。您也可以使用一个通用的 `restifyError` 事件来捕获所有类型的错误。

发送 404 的示例：

```js
server.get('/hello/:foo', function(req, res, next) {
  // 找不到资源错误
  var err = new restify.errors.NotFoundError('oh noes!');
  return next(err);
});

server.on('NotFound', function (req, res, err, cb) {
  // 不要调用 res.send！您现在处于错误上下文中，并且不在正常的下一链中。您可以在此时记录
  // 或执行指标，然后调用当你完成时的回调。restify 将根据你在响应中设置的报头内容类型自动
  // 渲染 NotFoundError。
  return cb();
});
```

为了自定义发送回客户端的错误：

```js
server.get('/hello/:name', function(req, res, next) {
  // 一些内部不可恢复的错误
  var err = new restify.errors.InternalServerError('oh noes!');
  return next(err);
});

server.on('InternalServer', function (req, res, err, cb) {
  // 默认情况下，restify 通常会根据内容协商将 Error 对象渲染为纯文本或 JSON。默认的文本
  // 格式化器和 JSON 格式化器非常简单，只需要在传递给 res.send 的对象上调用 toString()
  // 和 toJSON() 就可以了，在当前例子中指代错误对象。所以要自定义当错误发生时发送会客户端
  // 的内容，您需要执行如下操作：

  // 对于任何 `text/plain` 类型的响应
  err.toString = function toString() {
    return 'an internal server error occurred!';
  };
  // 对于任何 `application/json` 类型的响应
  err.toJSON = function toJSON() {
    return {
      message: 'an internal server error occurred!',
      code: 'boom!'
    }
  };

  return cb();
});

server.on('restifyError', function (req, res, err, cb) {
  // 这个侦听器会在上述两个事件之后触发！
  // 这里的 `err` 与传递给上述错误处理程序的错误相同。
  return cb();
});
```

这是 `InternalServerError` 的另一个示例，但是这次使用的时自定义的格式化器：

```js
const errs = require('restify-errors');

const server = restify.createServer({
  formatters: {
    'text/html': function(req, res, body) {
      if (body instanceof Error) {
        // 这里的 body 是 InternalServerError 的一个实例
        return '<html><body>' + body.message + '</body></html>';
      }
    }
  }
});

server.get('/', function(req, res, next) {
  res.header('content-type', 'text/html');
  return next(new errs.InternalServerError('oh noes!'));
});
```


### restify-errors

一个名为 restify-errors 的模块公开了一组用于许多常见的 http 和 REST 相关错误的错误构造函数。这些构造函数可以与 `next(err)` 模式结合使用，以便轻松利用服务器的事件发射器。完整的构造函数列表可以在 [restify-errors](https://github.com/restify/errors) 代码库中查看。这里有一些例子：

```js
var errs = require('restify-errors');

server.get('/', function(req, res, next) {
  return next(new errs.ConflictError("I just don't like you"));
});
```

```sh
$ curl -is localhost:3000
HTTP/1.1 409 Conflict
Content-Type: application/json
Content-Length: 53
Date: Fri, 03 Jun 2016 20:29:45 GMT
Connection: keep-alive

{"code":"Conflict","message":"I just don't like you"}
```

当使用 restify-errors 时，您也可以直接调用 `res.send(err)`，restify 会自动序列化您的错误：

```js
var errs = require('restify-errors');

server.get('/', function(req, res, next) {
  res.send(new errs.GoneError('gone girl'));
  return next();
});
```

```sh
$ curl -is localhost:8080/
HTTP/1.1 410 Gone
Content-Type: application/json
Content-Length: 37
Date: Fri, 03 Jun 2016 20:17:48 GMT
Connection: keep-alive

{"code":"Gone","message":"gone girl"}
```

这种自动序列化行为的发生是因为 JSON 格式化器会在 Error 对象上调用 `JSON.stringify()`，并且所有的 restify-errors 都定义了一个  `toJSON` 方法。将其与没有定义 `toJSON` 的标准 Error 对象进行比较：

```js
server.get('/sendErr', function(req, res, next) {
  res.send(new Error('where is my msg?'));
  return next();
});

server.get('/nextErr', function(req, res, next) {
  return next(new Error('where is my msg?'));
});
```

```sh
$ curl -is localhost:8080/sendErr
HTTP/1.1 410 Gone
Content-Type: application/json
Content-Length: 37
Date: Fri, 03 Jun 2016 20:17:48 GMT
Connection: keep-alive

{}

$ curl -is localhost:8080/nextErr
HTTP/1.1 410 Gone
Content-Type: application/json
Content-Length: 37
Date: Fri, 03 Jun 2016 20:17:48 GMT
Connection: keep-alive

{}
```

如果您想使用自定义错误，请确保您已经定义了 `toJSON`，或者使用 restify-error 的 `makeConstructor()` 方法来自动创建支持 `toJSON` 的错误。


#### HttpError

restify-errors 提供了继承自 HttpError 或 RestError 的构造函数。所有 HttpErrors 都有一个数字化的 http  `statusCode` 和 `body` 属性。`statusCode` 会自动设置 HTTP 的响应状态码，默认的 `body` 属性就是消息。

400 和 5xx 之间的所有状态码会自动转换为无空格的 'PascalCase' 写法的 HttpError。相关的完整列表，请查看 [Node.js 源码](https://github.com/nodejs/node/blob/master/lib/_http_server.js#L17) 。

例如，从上面的代码来看 `418: I'm a teapot` 会变成 `ImATeapotError`。


#### RestError

REST API 和 HTTP 的一个常见问题是，它们通常最终需要重载 400 和 409 来表示一堆不同的东西。在这些情况下做什么没有真正的标准，但一般而言，您希望服务器能够（安全地）解析这些东西，因此 restify 定义了一个 `RestError` 规范。`RestError` 是一个特定的 `HttpError` 类型的子类，并另外将 body 属性设置为带有 `code` 和 `message` 属性的 JavaScript 对象。例如，这是一个内置的 RestError：

```js
var errs = require('restify-errors');
var server = restify.createServer();

server.get('/hello/:name', function(req, res, next) {
  return next(new errs.InvalidArgumentError("I just don't like you"));
});

$ curl -is localhost:8080/hello/mark | json
HTTP/1.1 409 Conflict
Content-Type: application/json
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET
Access-Control-Allow-Headers: Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, Api-Version
Access-Control-Expose-Headers: Api-Version, Request-Id, Response-Time
Connection: close
Content-Length: 60
Content-MD5: MpEcO5EQFUZ2MNeUB2VaZg==
Date: Tue, 03 Jan 2012 00:50:21 GMT
Server: restify
Request-Id: bda456dd-2fe4-478d-809c-7d159d58d579
Response-Time: 3

{
  "code": "InvalidArgument",
  "message": "I just don't like you"
}
```

内置的 HttpErrors 包含：

* BadRequestError (400 Bad Request)
* UnauthorizedError (401 Unauthorized)
* PaymentRequiredError (402 Payment Required)
* ForbiddenError (403 Forbidden)
* NotFoundError (404 Not Found)
* MethodNotAllowedError (405 Method Not Allowed)
* NotAcceptableError (406 Not Acceptable)
* ProxyAuthenticationRequiredError (407 Proxy Authentication Required)
* RequestTimeoutError (408 Request Time-out)
* ConflictError (409 Conflict)
* GoneError (410 Gone)
* LengthRequiredError (411 Length Required)
* PreconditionFailedError (412 Precondition Failed)
* RequestEntityTooLargeError (413 Request Entity Too Large)
* RequesturiTooLargeError (414 Request-URI Too Large)
* UnsupportedMediaTypeError (415 Unsupported Media Type)
* RequestedRangeNotSatisfiableError (416 Requested Range Not Satisfiable)
* ExpectationFailedError (417 Expectation Failed)
* ImATeapotError (418 I'm a teapot)
* UnprocessableEntityError (422 Unprocessable Entity)
* LockedError (423 Locked)
* FailedDependencyError (424 Failed Dependency)
* UnorderedCollectionError (425 Unordered Collection)
* UpgradeRequiredError (426 Upgrade Required)
* PreconditionRequiredError (428 Precondition Required)
* TooManyRequestsError (429 Too Many Requests)
* RequestHeaderFieldsTooLargeError (431 Request Header Fields Too Large)
* InternalServerError (500 Internal Server Error)
* NotImplementedError (501 Not Implemented)
* BadGatewayError (502 Bad Gateway)
* ServiceUnavailableError (503 Service Unavailable)
* GatewayTimeoutError (504 Gateway Time-out)
* HttpVersionNotSupportedError (505 HTTP Version Not Supported)
* VariantAlsoNegotiatesError (506 Variant Also Negotiates)
* InsufficientStorageError (507 Insufficient Storage)
* BandwidthLimitExceededError (509 Bandwidth Limit Exceeded)
* NotExtendedError (510 Not Extended)
* NetworkAuthenticationRequiredError (511 Network Authentication Required)
* BadDigestError (400 Bad Request)
* BadMethodError (405 Method Not Allowed)
* InternalError (500 Internal Server Error)
* InvalidArgumentError (409 Conflict)
* InvalidContentError (400 Bad Request)
* InvalidCredentialsError (401 Unauthorized)
* InvalidHeaderError (400 Bad Request)
* InvalidVersionError (400 Bad Request)
* MissingParameterError (409 Conflict)
* NotAuthorizedError (403 Forbidden)
* RequestExpiredError (400 Bad Request)
* RequestThrottledError (429 Too Many Requests)
* ResourceNotFoundError (404 Not Found)
* WrongAcceptError (406 Not Acceptable)

* 具体的使用场景可以查阅 [HTTP状态码](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)，译者注*

内置的 RestErrors 包含：

* 400 BadDigestError
* 405 BadMethodError
* 500 InternalError
* 409 InvalidArgumentError
* 400 InvalidContentError
* 401 InvalidCredentialsError
* 400 InvalidHeaderError
* 400 InvalidVersionError
* 409 MissingParameterError
* 403 NotAuthorizedError
* 412 PreconditionFailedError
* 400 RequestExpiredError
* 429 RequestThrottledError
* 404 ResourceNotFoundError
* 406 WrongAcceptError

您也可以使用 `makeConstructor` 方法来创建自己的子类：

```js
var errs = require('restify-errors');
var restify = require('restify');

errs.makeConstructor('ZombieApocalypseError');

var myErr = new errs.ZombieApocalypseError('zomg!');
```

构造函数需要 `message`、`statusCode`、`restCode` 和 `context` 选项。请查看 restify-errors 代码库以获取更多信息。


## Socket.IO

在 restify 中使用 [socket.io](http://socket.io/)，只要将您的 restify 服务器视为“原始”的 Node.js 服务器即可：

```js
var server = restify.createServer();
var io = socketio.listen(server.server);

server.get('/', function indexHTML(req, res, next) {
  fs.readFile(__dirname + '/index.html', function (err, data) {
    if (err) {
      next(err);
      return;
    }

    res.setHeader('Content-Type', 'text/html');
    res.writeHead(200);
    res.end(data);
    next();
  });
});


io.sockets.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});

server.listen(8080, function () {
  console.log('socket.io server listening at %s', server.url);
});
```
