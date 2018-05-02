# 快速开始

通过 restify，你可以简单快速地创建一个服务器。以下代码就是一个标准的响应服务器：

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

请注意，默认情况下，curl 使用 `Connection: keep-alive`。为了使 HEAD 方法立即返回，你需要主动传递 `Connection: close`。

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

`pre` 处理程序链在路由之前执行。这意味着这些处理程序将针对传入的请求执行，即便它是你并未注册的路由。这可以用于日志和指标或在路由之前清理传入的请求。

```js
// 在路由之前删除 URL 中重复的斜杠
server.pre(restify.plugins.dedupeSlashes());
```


## 通用处理程序：server.use()

`use` 处理程序链式在请求被路由选择服务之后执行的。通过 `use()` 方法附加的函数处理程序将针对所有路由运行。由于 restify 以注册顺序运行处理程序，确保在定义任何路由之前，你所有的 `use()` 调用都会发生。

```js
server.use(function(req, res, next) {
	console.warn('run for all routes!');
	return next();
});
```


## 使用 next()

当处理程序链的每个函数执行之后，你需要负责调用 `next()`。调用 `next()` 后将移动到链中的下一个函数。

与其他的 REST 框架不同，调用 `res.send()` 不会自动触发 `next()`。在许多应用程序中，在 `res.send()` 之后可能需要继续处理，因此刷新响应并不等同于完成请求。

在正常情况下，`next()` 通常不会使用任何参数。如果由于某种原因你想停止处理请求，你可以调用 `next(false)` 来停止处理请求：

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

这两者的区别在于使用 Error 对象调用 `next()` 允许你利用服务器的 [EventEmitter](/api/server.md#errors)。这使你可以使用通用的处理程序来处理所有出现的错误类型。更多详情，请参见[错误处理](#错误处理)章节。

最后，你可以通过调用带有 Error 对象的 `next.ifError(err)` 来引起 restify 抛出错误，从而导致进程失败。如果你遇到无法处理的错误，需要你终止该进程时，这会非常有用。

## 路由

'basic' 模式下的 restify 路由行为与 express/sinatra 非常类似，都使用 HTTP 动词与参数化资源一起来确定要运行的处理程序链。在 `req.params` 中可以找到与指定占位符关联的值。这些值在传递给你之前会被 URL 编码。

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

你也可以传入 [RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 对象并通过 `req.params` 访问捕获组（不会以任何方式解析）：

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

如果参数化路由是由字符串（而不是正则表达式）定义的，那么你可以从服务器中的其他位置渲染它。这对于链接到其他资源的 HTTP 响应非常有用，而不必在整个代码库中对 URL 进行硬编码。路径和查询字符串参数都可以适当地进行 URL 编码。

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

大多数的 REST API 倾向于需要版本控制，并且使用 `Accept-Version` 报头来支持 [semver](http://semver.org/) 版本化，这种方式与你指定 NPM 版本依赖相同：

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

你可以通过在服务器创建时传递版本字段来设置路由的默认版本。最后，你可以通过使用数组来支持多版本的 API：

```js
server.get('/hello/:name' restify.plugins.conditionalHandler([
  { version: ['2.0.0', '2.1.0', '2.2.0'], handler: sendV2 }
]));
```

在这种情况下，你可能需要了解更多的信息，例如原始请求的版本字符串是什么，以及支持版本数组的路由的匹配版本是什么。有两种方法可用于获得此信息：

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

## Upgrade Requests

Incoming HTTP requests that contain a `Connection: Upgrade` header are treated
somewhat differently by the node HTTP server.  If you want restify to push
Upgrade requests through the regular routing chain, you need to enable
`handleUpgrades` when creating the server.

To determine if a request is eligible for Upgrade, check for the existence of
`res.claimUpgrade()`.  This method will return an object with two properties:
the `socket` of the underlying connection, and the first received data `Buffer`
as `head` (may be zero-length).

Once `res.claimUpgrade()` is called, `res` itself is marked unusable for
further HTTP responses; any later attempt to `send()` or `end()`, etc, will
throw an `Error`.  Likewise if `res` has already been used to send at least
part of a response to the client, `res.claimUpgrade()` will throw an `Error`.
Upgrades and regular HTTP Response behaviour are mutually exclusive on any
particular connection.

Using the Upgrade mechanism, you can use a library like
[watershed](https://github.com/jclulow/node-watershed) to negotiate WebSockets
connections.  For example:

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

## Content Negotiation

If you're using `res.send()` restify will automatically select the content-type
to respond with, by finding the first registered `formatter` defined.  Note in
the examples above we've not defined any formatters, so we've been leveraging
the fact that restify ships with `application/json`, `text/plain` and
`application/octet-stream` formatters. You can add additional formatters to
restify by passing in a hash of content-type -> parser at server creation time:

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

If a content-type can't be negotiated, then restify will default to using the
`application/octet-stream` formatter. For example, attempting to send a
content-type that does not have a defined formatter:

```js
server.get('/foo', function(req, res, next) {
  res.setHeader('content-type', 'text/css');
  res.send('hi');
  return next();
});
```

Will result in a response with a content-type of `application/octet-stream`:

```sh
$ curl -i localhost:3000/
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 2
Date: Thu, 02 Jun 2016 06:50:54 GMT
Connection: keep-alive
```

As previously noted, restify ships with built-in formatters for json, text,
and binary. When you override or append to this, the "priority" might change;
to ensure that the priority is set to what you want, you should set a `q-value`
on your formatter definitions, which will ensure sorting happens the way you
want:

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

The restify response object retains has all the "raw" methods of a node
[ServerResponse](http://nodejs.org/docs/latest/api/http.html#http.ServerResponse)
 on it as well.

```js
var body = 'hello world';
res.writeHead(200, {
  'Content-Length': Buffer.byteLength(body),
  'Content-Type': 'text/plain'
});
res.write(body);
res.end();
```

## Error handling

It is common to want to handle an error conditions the same way. As an example,
you may want to serve a 500 page on all `InternalServerErrors`. In this case,
you can add a listener for the `InternalServer` error event that is always
fired when this Error is encountered by restify as part of a `next(error)`
statement. This gives you a way to handle all errors of the same class
identically across the server. You can also use a generic `restifyError` event
which will catch errors of all types.

An example of sending a 404:

```js
server.get('/hello/:foo', function(req, res, next) {
  // resource not found error
  var err = new restify.errors.NotFoundError('oh noes!');
  return next(err);
});

server.on('NotFound', function (req, res, err, cb) {
  // do not call res.send! you are now in an error context and are outside
  // of the normal next chain. you can log or do metrics here, and invoke
  // the callback when you're done. restify will automtically render the
  // NotFoundError depending on the content-type header you have set in your
  // response.
  return cb();
});
```

For customizing the error being sent back to the client:

```js
server.get('/hello/:name', function(req, res, next) {
  // some internal unrecoverable error
  var err = new restify.errors.InternalServerError('oh noes!');
  return next(err);
});

server.on('InternalServer', function (req, res, err, cb) {
  // by default, restify will usually render the Error object as plaintext or
  // JSON depending on content negotiation. the default text formatter and JSON
  // formatter are pretty simple, they just call toString() and toJSON() on the
  // object being passed to res.send, which in this case, is the error object.
  // so to customize what it sent back to the client when this error occurs,
  // you would implement as follows:

  // for any response that is text/plain
  err.toString = function toString() {
    return 'an internal server error occurred!';
  };
  // for any response that is application/json
  err.toJSON = function toJSON() {
    return {
      message: 'an internal server error occurred!',
      code: 'boom!'
    }
  };

  return cb();
});

server.on('restifyError', function (req, res, err, cb) {
  // this listener will fire after both events above!
  // `err` here is the same as the error that was passed to the above
  // error handlers.
  return cb();
});
```

Here is another example of `InternalServerError`, but this time with a custom
formatter:

```js
const errs = require('restify-errors');

const server = restify.createServer({
  formatters: {
    'text/html': function(req, res, body) {
      if (body instanceof Error) {
        // body here is an instance of InternalServerError
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

A module called restify-errors exposes a suite of error constructors for many
common http and REST related errors. These constructors can be used in
conjunction with the `next(err)` pattern to easily leverage the server's event
emitter. The full list of constructors can be viewed over at the
[restify-errors](https://github.com/restify/errors) repository. Here are some
examples:


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

When using restify-errors, you can also directly call `res.send(err)`, and
restify will automatically serialize your error for you:

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

This automatic serialization happens because the JSON formatter will call
`JSON.stringify()` on the Error object, and all restify-errors have a `toJSON`
method defined. Compare this to a standard Error object which does not have
`toJSON` defined:

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

If you want to use custom errors, make sure you have `toJSON` defined, or use
restify-error's `makeConstructor()` method to automatically create Errors that
are supported with with `toJSON`.


#### HttpError

restify-errors provides constructors that inherit from either HttpError or
RestError. All HttpErrors have a numeric http `statusCode` and `body`
properties. The statusCode will automatically set the HTTP response status
code, and the body attribute by default will be the message.

All status codes between 400 and 5xx are automatically converted into
an HttpError with the name being 'PascalCase' and spaces removed.  For
the complete list, take a look at the
[node source](https://github.com/nodejs/node/blob/master/lib/_http_server.js#L17).

From that code above `418: I'm a teapot` would be `ImATeapotError`, as
an example.


#### RestError

A common problem with REST APIs and HTTP is that they often end
up needing to overload 400 and 409 to mean a bunch of different
things.  There's no real standard on what to do in these cases, but in
general you want machines to be able to (safely) parse these things
out, and so restify defines a convention of a `RestError`.  A
`RestError` is a subclass of one of the particular `HttpError` types,
and additionally sets the body attribute to be a JS object with the
attributes `code` and `message`.  For example, here's a built-in RestError:


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

The built-in HttpErrors are:

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

And the built in RestErrors are:

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

You can also create your own subclasses using the `makeConstructor` method:

```js
var errs = require('restify-errors');
var restify = require('restify');

errs.makeConstructor('ZombieApocalypseError');

var myErr = new errs.ZombieApocalypseError('zomg!');
```

The constructor takes `message`, `statusCode`, `restCode`, and `context`
options. Please check out the restify-errors repo for more information.


## Socket.IO

To use [socket.io](http://socket.io/) with restify, just treat your restify
server as if it were a "raw" node server:

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
