### 目录

-   [Request](#Request)
    -   [accepts](#accepts)
    -   [acceptsEncoding](#acceptsencoding)
    -   [contentLength](#contentlength)
    -   [getContentType](#getcontenttype)
    -   [date](#date)
    -   [href](#href)
    -   [id](#id)
    -   [getPath](#getpath)
    -   [getQuery](#getquery)
    -   [time](#time)
    -   [version](#version)
    -   [header](#header)
    -   [trailer](#trailer)
    -   [is](#is)
    -   [isChunked](#ischunked)
    -   [isKeepAlive](#iskeepalive)
    -   [isSecure](#issecure)
    -   [isUpgradeRequest](#isupgraderequest)
    -   [isUpload](#isupload)
    -   [toString](#tostring)
    -   [userAgent](#useragent)
    -   [startHandlerTimer](#starthandlertimer)
    -   [endHandlerTimer](#endhandlertimer)
    -   [connectionState](#connectionstate)
    -   [getRoute](#getroute)
-   [Log](#Log)

## Request

**扩展 http.IncomingMessage**

封装了所有的 Node.js [http.IncomingMessage](https://nodejs.org/api/http.html) API、事件和属性，以及以下内容。

### accepts

检查 Accept 头是否存在，并包含给定的类型。当 Accept 头不存在时，返回 `true`。否则检查是否完全匹配给定的类型，以及子类型。

**参数**

-   `types` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>)** 一个包含接受类型报头的数组

**例子**

_您可以传递子类型，如 html，然后使用 mime 查找表将其内部转换为 text/html：_

```javascript
// 接受：text/html
req.accepts('html');
// => true

// 接受：text/*; application/json
req.accepts('html');
req.accepts('text/html');
req.accepts('text/plain');
req.accepts('application/json');
// => true

req.accepts('image/png');
req.accepts('png');
// => false
```

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)**，类型是否被接受。

### acceptsEncoding

检查请求是否接受指定的编码类型。

**参数**

-   `types` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>)** 一个包含编码类型报头的数组

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)**，编码类型是否被接受。

### contentLength

返回 content-length 头的值。

返回 **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)**

### getContentType

返回 content-type 头的值。如果没有设置 content-type，将返回默认值 `application/octet-stream`。

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)**，内容类型。

### date

返回一个 Date 对象，表示请求的建立时间。和 `time()` 一样，但返回一个 Date 对象。

返回 **[Date](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date)**，何时开始处理请求。

### href

返回完整的请求 URL。

**例子**

```javascript
// 传入的请求是 http://localhost:3000/foo/bar?a=1
server.get('/:x/bar', function(req, res, next) {
	console.warn(req.href());
	// => /foo/bar/?a=1
});
```

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)**

### id

返回请求 id。如果传入一个 `reqId` 值，这将成为请求的新 id。请求 id 是不可变的，并且只能设置一次。尝试多次设置请求 id 会导致 restify 抛出错误。

**参数**

-   `reqId` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 请求 id

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)**，id。

### getPath

返回已清理的请求 URL。

**例子**

```javascript
// 传入的请求是 http://localhost:3000/foo/bar?a=1
server.get('/:x/bar', function(req, res, next) {
	console.warn(req.path());
	// => /foo/bar
});
```

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)**

### getQuery

返回原始查询字符串。如果找不到查询字符串，则返回空字符串。

**例子**

```javascript
// 传入的请求是 /foo?a=1
req.getQuery();
// => 'a=1'
```

_如果使用 queryParser 插件，则可以从 req.query 获取到解析后的查询字符串：_

```javascript
// 传入的请求是 /foo?a=1
server.use(restify.plugins.queryParser());
req.query;
// => { a: 1 }
```

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 查询字符串。

### time

自请求开始处理时起的时间 ms。和 `date()` 一样，但返回一个数字。

返回 **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)**，请求开始处理时的时间：自 1970年 1月 1日，UTC 时间 00:00:00 起的毫秒数。

### version

返回 accept-version 头。

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)**

### header

获取不区分大小写的请求头关键字，可以选择提供默认值（兼容 express）。返回任何请求的报头。同时“纠正”任何拼写正确的“引用”报头为实际使用的拼写。

**参数**

-   `key` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头关键字
-   `defaultValue` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 如果在请求中未找到报头，则使用默认值

**例子**

```javascript
req.header('Host');
req.header('HOST');
req.header('Accept', '*\/*');
```

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头的值。

### trailer

返回请求中的任何 trailer 头。返回任何请求的报头。同时“纠正”任何拼写正确的“引用”报头为实际使用的拼写。

**参数**

-   `name` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头的名称
-   `value` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 如果在请求中未找到报头，则使用默认值

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** trailer 值。

### is

检查传入请求是否包含 `Content-Type` 头字段，以及是否包含给定的 MIME 类型。

**参数**

-   `type` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 一个 content-type 头的值

**例子**

```javascript
// 使用 Content-Type: text/html; charset=utf-8
req.is('html');
req.is('text/html');
// => true

// 当 Content-Type 是 application/json 时
req.is('json');
req.is('application/json');
// => true

req.is('html');
// => false
```

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否是 content-type 头。

### isChunked

检查传入的请求是否被分块。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否被分块。

### isKeepAlive

检查传入请求是否保持活动状态。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否保持活动状态。

### isSecure

检查传入请求是否加密。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否加密。

### isUpgradeRequest

检查传入请求是否已升级。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否已升级。

### isUpload

检查传入请求是否为上传动词。

返回 **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否为上传。

### toString

toString 序列化

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 序列化的请求。

### userAgent

返回 user-agent 头。

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 用户代理

### startHandlerTimer

为请求处理程序启动计时器。默认情况下，restify 自动为处理程序链中注册的所有处理程序调用此操作。然而，这可以被手动调用为处理链中的嵌套函数记录时间信息。

**参数**

-   `handlerName` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 处理程序的名称

**例子**

_调用此函数后，您必须显式调用 endHandlerTimer()，否则时间信息将不准确。_

```javascript
server.get('/', function fooHandler(req, res, next) {
	vasync.pipeline({
		funcs: [
			function nestedHandler1(req, res, next) {
				req.startHandlerTimer('nestedHandler1');
				// do something
				req.endHandlerTimer('nestedHandler1');
				return next();
			},
			function nestedHandler1(req, res, next) {
				req.startHandlerTimer('nestedHandler2');
				// do something
				req.endHandlerTimer('nestedHandler2');
				return next();
			}...
		]...
	}, next);
});
```

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 无返回值

### endHandlerTimer

结束请求处理程序的计时器。如果在处理程序上调用 `startRequestHandler`，则必须调用此函数。否则，记录的时间将不正确。

**参数**

-   `handlerName` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 处理程序的名称

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 无返回值

### connectionState

返回请求的连接状态。当前可能的值是：

-   `close` - 当请求被客户关闭时

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 连接状态 (`"close"`)

### getRoute

返回与当前请求匹配的路由对象。

**例子**

_路由信息对象结构：_

```javascript
{
  path: '/ping/:name',
  method: 'GET',
  versions: [],
  name: 'getpingname'
}
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 路由

## Log

如果您使用了 [RequestLogger](/api/plugins/#requestlogger) 插件，则可以在 `req.log` 上使用子记录器：

```javascript
function myHandler(req, res, next) {
  var log = req.log;

  log.debug({params: req.params}, 'Hello there %s', 'foo');
}
```

子记录器会将请求的 UUID 注入到每个日志语句的 `req._id` 属性中。由于记录器在请求的生命周期中持续存在，您可以使用它跨越任何数量的独立处理程序来关联某个请求的语句。
