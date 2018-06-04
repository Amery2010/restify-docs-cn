---
title: 插件 API
permalink: /docs/plugins-api/
---

### 目录

-   [用法](#用法)
-   [server.pre() 插件](#serverpre-插件)
    -   [context](#context)
    -   [dedupeSlashes](#dedupeslashes)
    -   [pause](#pause)
    -   [sanitizePath](#sanitizepath)
    -   [reqIdHeaders](#reqidheaders)
    -   [strictQueryParams](#strictqueryparams)
    -   [userAgentConnection](#useragentconnection)
-   [server.use() 插件](#serveruse-插件)
    -   [acceptParser](#acceptparser)
    -   [authorizationParser](#authorizationparser)
    -   [dateParser](#dateparser)
    -   [queryParser](#queryparser)
    -   [jsonp](#jsonp)
    -   [bodyParser](#bodyparser)
    -   [requestLogger](#requestlogger)
    -   [gzipResponse](#gzipresponse)
    -   [serveStatic](#servestatic)
    -   [throttle](#throttle)
    -   [requestExpiry](#requestexpiry)
        -   [使用 key/bucket 映射的外部存储机制。](#使用-keybucket-映射的外部存储机制)
    -   [inflightRequestThrottle](#inflightrequestthrottle)
    -   [cpuUsageThrottle](#cpuusagethrottle)
    -   [conditionalHandler](#conditionalhandler)
    -   [conditionalRequest](#conditionalrequest)
    -   [auditLogger](#auditlogger)
    -   [metrics](#metrics)
-   [类型](#类型)
    -   [metrics~callback](#metricscallback)

## 用法

Restify 捆绑了一系列有用的插件。这些可通过 `restify.plugins` 和 `restify.pre` 访问。

```javascript
var server = restify.createServer();
server.use(restify.plugins.acceptParser(server.acceptable));
server.use(restify.plugins.authorizationParser());
server.use(restify.plugins.dateParser());
server.use(restify.plugins.queryParser());
server.use(restify.plugins.jsonp());
server.use(restify.plugins.gzipResponse());
server.use(restify.plugins.bodyParser());
server.use(restify.plugins.requestExpiry());
server.use(restify.plugins.throttle({
  burst: 100,
  rate: 50,
  ip: true,
  overrides: {
    '192.168.1.1': {
      rate: 0,        // 无限制
      burst: 0
    }
  }
}));
server.use(restify.plugins.conditionalRequest());
```


## server.pre() 插件

该模块包含各种预插件，这些插件会在路由 URL 之前使用。要在路由之前使用插件，请使用 `server.pre()` 方法。

### context

该插件创建用于设置和检索请求特定数据的 `req.set(key, val)` 和 `req.get(key)` 方法。

**例子**

```javascript
server.pre(restify.plugins.pre.context());
server.get('/', [
  function(req, res, next) {
    req.set(myMessage, 'hello world');
    return next();
  },
  function two(req, res, next) {
    res.send(req.get(myMessage)); // => 发送 'hello world'
    return next();
  }
]);
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### dedupeSlashes

该插件会删除在 URL 中找到的额外斜杠。这可以帮助处理格式不正确的 URL，否则可能会导致路由错误。

**例子**

```javascript
server.pre(restify.plugins.pre.dedupeSlashes());
server.get('/hello/:one', function(req, res, next) {
  res.send(200);
  return next();
});

// 服务器现在会把请求 /hello//jake 转换为 /hello/jake
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### pause

此预处理程序修复了在 `bodyParser` 之前使用 `asyncHandler` 时，Node.js 挂起的问题。
-   <https://github.com/restify/node-restify/issues/287>
-   <https://github.com/restify/node-restify/issues/409>
-   <https://github.com/restify/node-restify/wiki/1.4-to-2.0-Migration-Tips>

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### sanitizePath

清理请求对象上的粗糙 URL，比如将 `/foo////bar///` 变成 `/foo/bar`。

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### reqIdHeaders

该插件从传入的请求报头中提取值并将其用作请求 id 的值。随后对 `req.id()` 的调用将返回报头的值。

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 一个选项对象
    -   `opts.headers` **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>** 一组从现有请求 id 中提取的一组报头。查找优先级从左到右（优先最低索引）

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### strictQueryParams

阻止 `req.urls` 非严谨的键值查询参数。

Request-URI 以 3.2.1 节规定的格式传输。
如果 Request-URI 使用 “% HEX HEX” 编码[42]进行编码，则源服务器**必须**对 Request-URI 进行解码，以正确解释请求。
服务器**应该**用适当的状态码来响应无效的 Request-URI。

超文本传输​​协议的一部分 -- HTTP/1.1 | 5.1.2 Request-URI
RFC 2616 Fielding 等

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象
    -   `options.message` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 自定义错误消息，默认值：“Url query params does not meet strict format”

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### userAgentConnection

这基本上是为 `curl` 而存在的。在 `HEAD` 请求上，`curl` 通常只是坐等挂起，除非你明确地设置了 `Connection:close`。一般来说，您可能无论如何都要给 curl 设置 `Connection:close`。

另外，因为如果设置了 `content-length`，curl 会向 stderr 发送一条令人讨厌的消息，因此这个插件也会删除 `content-length` 头（某些用户代理会处理它并需要它，curl 则不会）。

但为了稍微通用一些，选项块需要一个用户代理正则表达式。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象
    -   `options.userAgentRegExp` **[RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)** 匹配任何合适的用户代理（可选，默认为 `/^curl.+/`）

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

## server.use() 插件

### acceptParser

解析 `Accept` 头，并确保服务器可以响应客户要求的内容。为了让服务器知道如何响应一组内容类型（使用您注册的格式化程序），在几乎所有传递 `server.acceptable` 的情况下都是必要的。如果请求是一个无法处理的类型，则该插件会返回 `NotAcceptableError` (406)。

请注意，您可以通过执行 `server.acceptable` 来获取 restify 服务器允许的一组类型。

**参数**

-   `accepts` **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>** 可接受类型的数组。

**例子**

```javascript
server.use(restify.plugins.acceptParser(server.acceptable));
```

-   抛出 **NotAcceptableError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** restify 处理程序。

### authorizationParser

restify 会以最佳方式解析 `Authorization` 头。目前只支持 HTTP Basic Auth 以及 [HTTP Signature](https://github.com/joyent/node-http-signature) 格式。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个传递给 http-signature 的可选选项对象

**例子**

_后续的处理程序将会看到 `req.authorization`，看起来像上文所示。`req.username` 也将被设置，并且默认为 'anonymous'。如果该格式无法识别，那么 `req.authorization` 中唯一可用的格式将是 `scheme` 和 `credentials` - 其余内容将由您来决定如何处理。_

```javascript
{
  scheme: "<Basic|Signature|...>",
  credentials: "<Undecoded value of header>",
  basic: {
    username: $user
    password: $password
  }
}
```

-   抛出 **InvalidArgumentError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### dateParser

解析 HTTP Date 头（如果存在的话）并检查时钟偏斜。
如果报头无效，则返回 `InvalidHeaderError` (`400`)。
过期意味着请求始于（`$now - $clockSkew`）之前的时间。
clockSkew 的默认配额是 5 分钟

**参数**

-   `clockSkew` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 允许时钟偏斜秒数。(可选，默认 300)

**例子**

```javascript
// 允许时钟偏斜 1 分钟
server.use(restify.plugins.dateParser(60));
```

-   抛出 **RequestExpiredError** 
-   抛出 **InvalidHeaderError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** restify 处理程序。

### queryParser

解析 HTTP 查询字符串（如，`/foo?id=bar&name=mark`）。
如果您使用该插件，解析的内容在 `req.query` 中始终可用，额外的参数会被合并到 `req.params` 中。
您可以通过在选项对象中传递 `mapParams: false` 来禁用它。

许多选项直接对应于由 [`qs.parse`](https://github.com/ljharb/qs) 底层定义的选项。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象
    -   `options.mapParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 拷贝已解析的查询参数到 `req.params` 中。(可选，默认为 `false`)
    -   `options.overrideParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 仅当 mapParams 为 true 时才适用。如果为 true，则会在找到已有的值时覆盖 req.params 字段。(可选，默认为 `false`)
    -   `options.allowDots` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 转换 `?foo.bar=baz` 为一个嵌套的对象：`{foo: {bar: 'baz'}}`。(可选，默认为 `false`)
    -   `options.arrayLimit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 只转换 `?a[$index]=b` 中 `$index` 少于 `arrayLimit` 的数组。(可选，默认为 `20`)
    -   `options.depth` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 解析嵌套对象的深度限制，如 `?a[b][c][d][e][f][g][h][i]=j`。(可选，默认为 `5`)
    -   `options.parameterLimit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 查询参数被解析的最大数值。额外的参数被静默丢弃。(可选，默认为 `1000`)
    -   `options.parseArrays` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否将 `?a[]=b&a[1]=c` 解析为一个数组，如 `{a: ['b', 'c']}`。(可选，默认为 `true`)
    -   `options.plainObjects` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** `req.query` 是否为“普通”对象 - 不从 Object 继承。这可以用来允许名称与 Object 方法相冲突的查询参数，如，`?hasOwnProperty=blah`。(可选，默认为 `false`)
    -   `options.strictNullHandling` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 如果为 true，`?a&b=`的结果为 `{a: null, b: ''}`。否则，为 `{a: '', b: ''}`。(可选，默认为 `false`)

**例子**

```javascript
server.use(restify.plugins.queryParser({ mapParams: false }));
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### jsonp

从请求中解析 jsonp 回调。
支持检测 `callback` 或 `jsonp` 的查询字符串，并确保存在 JSONP 参数的情况下，设置合适的内容类型。
此外，默认使用 `application/javascript` 格式化程序处理。

您_应该_在这之前运行 `queryParser` 插件，但是如果您不这样做，这个插件仍可以正确解析查询字符串。

**例子**

```javascript
var server = restify.createServer();
server.use(restify.plugins.jsonp());
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### bodyParser

在读取和解析 HTTP 请求主体时阻塞您的链。依据 `Content-Type` 执行相应的逻辑。
目前支持 `application/json`、`application/x-www-form-urlencoded` 和 `multipart/form-data`。

解析 `POST` 主体为 `req.body`。根据内容类型自动使用以下解析程序之一：

-   `urlEncodedBodyParser(options)` - 解析 URL 编码的表单体
-   `jsonBodyParser(options)` - 解析 JSON 格式的 POST 主体
-   `multipartBodyParser(options)` - 解析多段表单体

所有 bodyParsers 都支持以下选项：

-   `options.mapParams` - 默认为 false。拷贝解析后的 post 正文内容到 req.params。
-   `options.overrideParams` - 默认为 false。仅当 mapParams 为 true 时才适用。如果为 true，则会在找到已有的值时覆盖 req.params 字段。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象。
    -   `options.maxBodySize` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** 允许 HTTP 主体的最大字节数。用于限制客户端占用的服务器内存。
    -   `options.mapParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 是否 `req.params` 应该填充来自 HTTP 主体的已解析参数。
    -   `options.mapFiles` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 是否 `req.params` 应该填充通过多段请求发送的文件的内容。内部使用 [formidable](https://github.com/felixge/node-formidable) 进行解析，并且文件被表示为 `Content-Disposition` 中 `filename` 选项集的多段部分。这只会在 `mapParams` 为 true 的情况下执行。
    -   `options.overrideParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 是否 `req.params` 中的条目应该被主体中的值覆盖（如果名称相同的话）。例如，如果您有名为 `/:someval` 的路由，并有人发了一个 `x-www-form-urlencoded` 内容格式的主体内容 `someval=happy` 到路由 `/sad`，如果 `overrideParams` 为 `true`，则该值为 `happy`，否则为 `sad`。
    -   `options.multipartHandler` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 一个用来处理任何不是文件的多段部分的回调函数。如果省略，则调用缺省处理程序，这可能会或可能不会将这些部分映射到 `req.params` 中，具体取决于 `mapParams` 选项。
    -   `options.multipartFileHandler` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 一个用来处理任何是文件的多段部分的回调函数。它是不是文件取决于该部分的 `Content-Disposition` 是否具有 `filename` 参数集。这通常发生在浏览器发送表单时，并且有一个类似于 `<input type="file" />` 的参数。如果没有提供，默认行为是将内容映射到 `req.params`。
    -   `options.keepExtensions` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 如果您希望上传的文件包含原始文件的扩展名（仅限分段上传）。如果定义了 `multipartFileHandler`，则什么也不做。
    -   `options.uploadDir` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 在内容被映射到  `req.params` 之前，上传文件在传输过程中被中间存储的地方。如果定义了 `multipartFileHandler`，则什么也不做。
    -   `options.multiples` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 如果您想要支持上传字段中 html5 的多属性。
    -   `options.hash` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 如果您想要为传入文件校验计算，请将其设置为 `sha1` 或 `md5`。
    -   `options.rejectUnknown` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 如果您想在没有提供任何支持的内容类型时使用 `UnsupportedMediaTypeError` 结束请求，则设置为 `true`。
    -   `options.requestBodyOnGet` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)**  解析GET请求的主体。(可选，默认为 `false`)
    -   `options.reviver` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 仅限 `jsonParser`。如果是一个函数，则规定了在返回之前，最初由解析产生的值如何转换。想要了解更多信息，请查看 `JSON.parse(text[, reviver])`。
    -   `options.maxFieldsSize` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 仅限 `multipartParser`。限制所有字段的内存量（文件除外）可以按字节分配。默认大小是 `2 * 1024 * 1024` 字节 _(2MB)_。(可选，默认为 `2*1024*1024`)

**例子**

```javascript
server.use(restify.plugins.bodyParser({
  maxBodySize: 0,
  mapParams: true,
  mapFiles: false,
  overrideParams: false,
  multipartHandler: function(part) {
    part.on('data', function(data) {
      // 对多段数据做些处理
    });
  },
  multipartFileHandler: function(part) {
    part.on('data', function(data) {
      // 对多段文件数据做些处理
    });
  },
  keepExtensions: false,
  uploadDir: os.tmpdir(),
  multiples: true,
  hash: 'sha1',
  rejectUnknown: true,
  requestBodyOnGet: false,
  reviver: undefined,
  maxFieldsSize: 2 * 1024 * 1024
 }));
```

-   抛出 **UnsupportedMediaTypeError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### requestLogger

设置一个子 [bunyan](https://github.com/trentm/node-bunyan) 记录器，写入当前的请求 ID 以及您定义的任何其他参数。

您可以不添加任何选项，在这种情况下，只会附加请求 ID，并且不会附加序列化程序（这也是最高性能的做法）；在服务器创建时创建的记录器将被用作父记录器。这个记录器可以与 [req.log](/api/request/#log) 共用。

这个插件_不_记录每个单独的请求。请使用审计日志插件或用于该用途的自定义中间件。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象
    -   `options.headers` **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)?** 要从请求传输到日志中的顶层属性的报头列表。

**例子**

```javascript
server.use(restify.plugins.requestLogger({
  properties: {
    foo: 'bar'
  },
  serializers: {...}
}));
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### gzipResponse

如果客户端发送了一个 `accept-encoding: gzip` 头（或一个带有适当 q-val 的头），那么服务器将自动 gzip 所有响应数据。请注意，只有支持 `gzip` 的客户端可用。
这个插件会覆盖一些内部流，所以任何对 `res.send`、`res.write` 等的调用都会被压缩。副作用是无法获知报头的 `content-length`，因此将在此插件生效时，_始终_设置 `transfer-encoding: chunked`。如果客户端没有发送 `accept-encoding: gzip`，则该插件没有影响。

<https://github.com/restify/node-restify/issues/284>

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?**一个选项对象，详见 zlib.createGzip

**例子**

```javascript
server.use(restify.plugins.gzipResponse());
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### serveStatic

提供静态文件。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 一个选项对象

**例子**

_serveStatic 模块与大多数其他插件不同，因为预期您将它映射到路由上，如下所示：_

```javascript
server.get('/zh-CN/docs/current/*', restify.plugins.serveStatic({
  directory: './documentation/v1',
  default: 'index.html'
}));
```

_当您尝试点击 `http://localhost:8080/zh-CN/docs/current/` 时，上述 `route` 和 `directory` 组合将提供位于 `./documentation/v1/zh-CN/docs/current/index.html` 中的文件。如果您想要 serveStatic 模块直接从 `/documentation/v1`目录提供文件（而不是附加请求路径 `/zh-CN/docs/current/`），您可以将 `appendRequestPath` 选项设置为 `false` 则在前面的例子中提供的文件是 `./documentation/v1/index.html`。这个插件会强制提供 `directory` 下的所有文件。
所提供的 `directory` 是相对于进程工作目录而言的。您还可以为缺少直接文件匹配的任何目录提供 `default` 参数，如，index.html。您可以通过传入 `match` 参数来指定其他限制，如，使用 `RegExp` 来检查请求的文件名。
另外，您可以设置 `charSet` 参数，它会把一个字符集附加到插件检测到的内容类型上。例如，`charSet: 'utf-8'` 会导致 HTML 被提供一个 `text/html; charset=utf-8` 的 `Content-Type`。
最后，您可以传入 `maxAge` 数值，它将被设置 `Cache-Control` 头的值。默认是 `3600` (1 小时)。提供静态文件的附加选项是将 `file` 作为选项传递给 serveStatic 方法。在客户端请求 `/home/` 时，以下将从 `documentation/v1/` 目录提供 index.html。_

```javascript
server.get('/home/*', restify.plugins.serveStatic({
  directory: './documentation/v1',
  file: 'index.html'
}));
// or
server.get('/home/([a-z]+[.]html)', restify.plugins.serveStatic({
  directory: './documentation/v1',
  file: 'index.html'
}));
```

-   抛出 **MethodNotAllowedError** \|
-   抛出 **NotAuthorizedError** 
-   抛出 **ResourceNotFoundError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### throttle

创建一个可以插入到标准的 restify 请求处理管道的 API 限速器。

`restify` 提供了相当全面的 [Token bucket](http://en.wikipedia.org/wiki/Token_bucket) 实现，能够对 IP（或x-forwarded-for）和用户名（来自 `req.username`）进行节流。您可以定义“全局”的请求速率和突发速率，并且可以覆盖定义特定的键。
请注意，您始终可以将此置于每个 URL 路由中，以针对不同的资源启用不同的请求速率（例如，在一个路由中，`/my/slow/database` 更容易超过 `/my/fast/memcache`）。

如果客户端已经消耗了所有可用速率/突发，则会返回 HTTP 响应代码 `429` [Too Many Requests](http://tools.ietf.org/html/draft-nottingham-http-new-status-03#section-4)。

该节流插件提供了三个节流选项：用户名、IP 地址和 'X-Forwarded-For'。如果使用 IP/XFF，请记住它是一个 /32 匹配。用户名需要用户在 req.username 上指定（它自动设置为支持的授权类型，否则自己用之前运行的过滤器设置它）。

在这两种情况下，您都可以设置一个整数/浮点数的 `burst` 和 `rate`（以 '请求/秒' 为单位）。那些确实转化成了 `TokenBucket` 算法，所以请阅读（或参阅上面的评论...）。

无论哪种情况，顶层选项参数的突发/速率都会设置一个总限制速率，然后您可以传入具有特定用户/IP速率的 `overrides` 对象。您应该谨慎使用覆盖，因为我们会制作一个新的 TokenBucket 来跟踪每个覆盖。

在 `options` 对象上，ip 和 username 互斥。

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 所需的选项：
    -   `options.burst` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 突发
    -   `options.rate` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 速率
    -   `options.ip` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** IP
    -   `options.username` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 用户名
    -   `options.xff` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** X-Forwarded-For
    -   `options.setHeaders` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 设置响应头的速率，限制（突发）和剩余。(可选，默认为 `false`)
    -   `options.overrides` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 覆盖对象
    -   `options.tokensTable` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 一个存储引擎，这个插件将用来存储节流 keys -> bucket 的映射。如果你没有指定，默认是使用内存 O(1) LRU，具有 10k 个不同的键。任何实现只需要支持 put/get。
    -   `options.maxKeys` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 如果使用默认实现，则可以指定您想要的表格大小。(可选，默认为 `10000`)

**例子**

_一个带 `overrides` 的选项对象示例：_

```javascript
{
  burst: 10,  // 最多 10 个并发请求（如果令牌）
  rate: 0.5,  // 稳定状态：1次请求 / 2秒
  ip: true,   // 针对每个 IP 进行节流
  overrides: {
    '192.168.1.1': {
      burst: 0,
      rate: 0    // 无限制
  }
}
```

-   抛出 **TooManyRequestsError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### requestExpiry

requestExpiry 可用于限制已超过其客户端超时时间的请求。可以使用可配置的客户端超时头发送请求，如 'x-request-expiry-time'，当这个请求被客户端超时，将给出从 Epoch 开始计算的绝对毫秒数。

这个插件将通过 504 限制 'x-request-expiry-time' 小于 Date.now() 的所有传入的请求 —— 因为这些传入的请求已被客户端超时。这可以防止服务器处理不必要的请求。

requestExpiry 将使用报头来判断传入的请求是否已过期。这个插件有两个选项：

 1、绝对时间，自 Epoch 开始的绝对时间（以毫秒为单位），此请求应被视为过期。
 
 2、超时，提供请求开始之后的超时时间（以毫秒为单位）。超时加上请求开始时间等于请求被认为过期的绝对时间。

#### 使用 key/bucket 映射的外部存储机制。

默认情况下，restify 节流插件使用内存中的 LRU 将节流键（即 IP 地址）与键消耗的实际 bucket 之间的映射进行存储。如果这适合您，您可以使用 `options.maxKeys` 来调整存储在内存中的最大键数量；默认值是 10000。

在某些情况下，您希望将其卸载到共享系统中，例如 Redis，如果您拥有一组 API 服务器，并且您没有获得稳定和/或统一的请求分配。要启用该功能，您可以传入 `options.tokensTable`，它只是任何支持 `put` 和 `get` 的 `String` 对象以及一个 `Object` 值。

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 一个选项对象
    -   `opts.absoluteHeader` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 用于每个请求过期时间的报头键。
    -   `opts.startHeader` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 请求开始时间的报头键。
    -   `opts.timeoutHeader` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 在请求被认为已过期之前报头键应该消耗的毫秒数。

**例子**

_提供的唯一选项是 `header`，它是用于指定客户端超时的请求报头。_

```javascript
server.use(restify.plugins.requestExpiry({
  header: 'x-request-expiry-time'
});
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### inflightRequestThrottle

`inflightRequestThrottle` 模块允许您指定服务器能够处理的最大数量的 inflight 请求上限。这是一种简单的启发式方法，用于防止请求之间的事件循环争用导致无法接受的延迟。

自定义错误是可选的，允许您在由于太多的 inflight 请求而导致拒绝传入请求时，指定自己的响应和状态码。它默认为 `503 ServiceUnavailableError`。

这个插件应尽早在中间件堆栈中使用 `pre` 注册，以避免执行不必要的工作。

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 配置这个插件
    -   `opts.limit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 在返回错误之前服务器将要处理的最大数量的 inflight 请求
    -   `opts.err` **[Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)** 当超过 inflight 请求限制时，将使用 restify 错误作为响应
    -   `opts.server` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 该插件将节流的 restify 服务器实例。

**例子**

```javascript
var errors = require('restify-errors');
var restify = require('restify');

var server = restify.createServer();
const options = { limit: 600, server: server };
options.err = new errors.InternalServerError();
server.pre(restify.plugins.inflightRequestThrottle(options));
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 需要在 server.pre 上注册的中间件

### cpuUsageThrottle

cpuUsageThrottle 是拒绝可变数量请求的中间件（介于 0％ 和 100％ 之间），它基于 Node.js 进程 CPU 利用率的历史视图。从本质上讲，这个插件允许你通过 CPU 利用率定义 Node.js 进程的饱和状态，它将处理基于该定义 a ％ 的请求。当您想要保持 CPU 约束任务堆积，从而导致每请求延迟增加时，这会非常有用。

该算法要求您提供最大的 CPU 利用率，用于确定它应该拒绝 100％ 的流量。对于一个正常的 Node.js 服务，这是 1，因为 Node.js 是单线程的。它使用这一点，与您提供的限制进行配对，以确定它应拒绝的总流量百分比。例如，如果您指定的限制为 .5，最大值为 1，并且当前的 EWMA（下一段）值为 .75，则此插件将拒绝大约 50％ 的所有请求。

在查看进程的 CPU 使用情况时，该算法将在用户指定的时间间隔内取平均值。例如，如果给定 250ms 的间隔，则此插件将尝试记录 250ms 间隔内的平均 CPU 利用率。由于竞争资源，每个平均的持续时间可能比 250ms 更长或更短。为了弥补这一点，我们使用指数加权移动平均。EWMA 算法由 ewma 模块提供。用于配置 EWMA 的参数是半衰期。该值控制每个加载平均测量值在当前平均值中衰减到半值时的速度。例如，如果间隔为 250，半衰期为 250，则将之前的 ewma 值乘以 0.5，并加上新的 CPU 利用率平均测量值乘以 0.5 的值。之前的值和新的测量值将分别代表新值的 50％。考虑半衰期的一个好方法就是该插件对 CPU 利用率高峰的响应程度。半衰期越高，在此插件开始拒绝请求之前，较长时间的 CPU 利用率必须保持在您定义的限制之上，反过来说，在插件再次接受请求之前，它将不得不降低到极限以下。当试图确定您的用例的理想值之前这是一个可调控的值。

为了更好地理解 EWMA 算法，请参阅 ewma 模块的文档。

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 配置该插件。
    -   `opts.limit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** restify 将开始拒绝传入的 a % 的所有请求。这个值是一个百分比。例如，0.8 === 80％ 的 CPU 平均利用率。默认为 0.75。
    -   `opts.max` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** restify 将拒绝传入 100％ 的所有请求。这与 `limit` 一起使用，以确定在尝试将平均负载恢复到用户请求值时，需要拒绝的流量占总流量的百分比。由于 Node.js 是单线程的，因此默认值为 1。在一些极端情况下，Node.js 进程可能会超过 100％ 的 CPU 使用率，此时您需要更新该值。
    -   `opts.interval` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** 计算平均 CPU 利用率的频率。当我们计算一个平均的 CPU 使用率时，我们会在这个时间间隔内计算出来，这会督促我们是否应该减轻负载。这可以被认为是一个“解析率”，这个值越低，我们的平均负载的解析率越高，我们将更频繁地重新计算我们应该减少的流量的百分比。此检查相当轻量，而且默认值为 250 毫秒，您应该能够降低此值而不会对性能产生重大影响。
    -   `opts.halfLife` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** 当我们在一个时间间隔内采样 CPU 使用率时，我们创建了一系列数据点。我们取这些点并计算移动平均值。半衰期表示一个点“移动”到移动平均值一半的速度有多快。半衰期越低，新数据点对平均值的影响就越大。如果您希望对 CPU 使用率的峰值作出极其敏感的响应，请将其设置为较低的值。如果您希望自己的进程在确定是否应该减少负载时，更加强调最近的历史 CPU 使用率，请将其设置为更高的值。单位是毫秒。默认为 250。

**例子**

```javascript
var restify = require('restify');

var server = restify.createServer();
const options = {
  limit: .75,
  max: 1,
  interval: 250,
  halfLife: 500,
}

server.pre(restify.plugins.cpuUsageThrottle(options));
```

_您也可以使用 `.update()` 函数在运行时更新插件。该函数接受与构造函数相同的 `opts` 对象。_

```javascript
var plugin = restify.plugins.cpuUsageThrottle(options);
server.pre(plugin);

plugin.update({ limit: .4, halfLife: 5000 });
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 需要在 server.pre 上注册的中间件

### conditionalHandler

运行与条件匹配的第一个处理程序。

**参数**

-   `candidates` **([Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)>)** 候选对象
    -   `candidates.handler` **([Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)>)** 处理程序
    -   `candidates.version` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>)?** '1.1.0', ['1.1.0', '1.2.0']
    -   `candidates.contentType` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 接受的内容类型，'\*\\/json'

**例子**

```javascript
server.use(restify.plugins.conditionalHandler({
  contentType: 'application/json',
  version: '1.0.0',
  handler: function (req, res, next) {
    next();
  })
});

server.get('/hello/:name', restify.plugins.conditionalHandler([
  {
    version: '1.0.0',
    handler: function(req, res, next) { res.send('1.x'); }
  },
  {
    version: ['1.5.0', '2.0.0'],
    handler: function(req, res, next) { res.send('1.5.x, 2.x'); }
  },
  {
    version: '3.0.0',
    contentType: ['text/html', 'text/html']
    handler: function(req, res, next) { res.send('3.x, text'); }
  },
  {
    version: '3.0.0',
    contentType: 'application/json'
    handler: function(req, res, next) { res.send('3.x, json'); }
  },
  // 处理程序组
  {
    version: '4.0.0',
    handler: [
      function(req, res, next) { next(); },
      function(req, res, next) { next(); },
      function(req, res, next) { res.send('4.x') }
    ]
  },
]);
// 'accept-version': '^1.1.0' => 1.5.x, 2.x'
// 'accept-version': '3.x', 接受：'application/json' => '3.x, json'
```

-   抛出 **InvalidVersionError** 
-   抛出 **UnsupportedMediaTypeError** 

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### conditionalRequest

返回一组插件，用于将已设置的 `ETag` 头与客户端的 `If-Match` 和 `If-None-Match` 头以及已设置的 `Last-Modified` 头与客户端的 `If-Modified-Since` 和 `If-Unmodified-Since` 头进行比较。

您可以使用此处理程序让客户端使用 "match" 头完成良好的 HTTP 语义。具体来说，使用该插件，您可以设置 `res.etag=$yourhashhere`，然后这个插件会执行以下操作之一：

-   返回 `304` (Not Modified) [并停止处理程序链]
-   返回 `412` (Precondition Failed) [并停止处理程序链]
-   允许请求通过处理程序链。

这个插件的特定报头是：

-   `Last-Modified`
-   `If-Match`
-   `If-None-Match`
-   `If-Modified-Since`
-   `If-Unmodified-Since`

**例子**

```javascript
server.use(restify.plugins.conditionalRequest());
```

```javascript
server.use(function setETag(req, res, next) {
  res.header('ETag', 'myETag');
  res.header('Last-Modified', new Date());
});

server.use(restify.plugins.conditionalRequest());

server.get('/hello/:name', function(req, res, next) {
  res.send('hello ' + req.params.name);
});
```

-   抛出 **BadRequestError** 
-   抛出 **PreconditionFailedError** 

返回 **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)>** 处理程序

### auditLogger

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 选项对象。
    -   `opts.log` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 记录器。
    -   `opts.event` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 来自启动日志的服务器的事件，'pre'、'routed' 或 'after' 之一。
    -   `opts.context` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** 签名函数 f(req, res, route, err) 的可选上下文。每次生成审计日志时调用。该函数可以返回一个对象，用于定制 req、res、route 和 err 对象之外的任何格式。该函数的输出将在审计对象的 `context` 关键字中可用。
    -   `opts.server` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** restify 服务器，用于以编程方式发出审计日志对象
    -   `opts.printLog` **[boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 是否通过记录器打印日志。(可选，默认为 `true`)

**例子**

_审计日志记录是一个特殊的插件，因为您不使用 `.use()`，而是使用 `after` 事件：_

```javascript
server.on('after', restify.plugins.auditLogger({
  log: bunyan.createLogger({
    name: 'audit',
    stream: process.stdout
  }),
  event: 'after',
  server: SERVER,
  logMetrics : logBuffer,
  printLog : true
}));
```

_You pass in the auditor a bunyan logger, optionally server object,
Ringbuffer and a flag printLog indicate if log needs to be print out at info
level or not.  By default, without specify printLog flag, it will write out
record lookling like this:_
_您向审计器传递一个 bunyan 记录器，可选服务器对象，Ringbuffer 和一个 printLog 标识指示是否需要在信息级别打印日志。默认情况下，不指定 printLog 标识，它会写出如下所示的记录：_

```javascript
{
  "name": "audit",
  "hostname": "your.host.name",
  "audit": true,
  "remoteAddress": "127.0.0.1",
  "remotePort": 57692,
  "req_id": "ed634c3e-1af0-40e4-ad1e-68c2fb67c8e1",
  "req": {
    "method": "GET",
    "url": "/foo",
    "headers": {
      "authorization": "Basic YWRtaW46am95cGFzczEyMw==",
      "user-agent": "curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8r zlib/1.2.3",
      "host": "localhost:8080",
      "accept": "application/json"
    },
    "httpVersion": "1.1",
    "query": {
      "foo": "bar"
    },
    "trailers": {},
    "version": "*",
    "timers": {
      "bunyan": 52,
      "saveAction": 8,
      "reqResTracker": 213,
      "addContext": 8,
      "addModels": 4,
      "resNamespaces": 5,
      "parseQueryString": 11,
      "instanceHeaders": 20,
      "xForwardedProto": 7,
      "httpsRedirector": 14,
      "readBody": 21,
      "parseBody": 6,
      "xframe": 7,
      "restifyCookieParser": 15,
      "fooHandler": 23,
      "barHandler": 14,
      "carHandler": 14
    }
  },
  "res": {
    "statusCode": 200,
    "headers": {
      "access-control-allow-origin": "*",
      "access-control-allow-headers": "Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, Api-Version",
      "access-control-expose-headers": "Api-Version, Request-Id, Response-Time",
      "server": "Joyent SmartDataCenter 7.0.0",
      "x-request-id": "ed634c3e-1af0-40e4-ad1e-68c2fb67c8e1",
      "access-control-allow-methods": "GET",
      "x-api-version": "1.0.0",
      "connection": "close",
      "content-length": 158,
      "content-md5": "zkiRn2/k3saflPhxXI7aXA==",
      "content-type": "application/json",
      "date": "Tue, 07 Feb 2012 20:30:31 GMT",
      "x-response-time": 1639
    },
    "trailer": false
  },
  "route": {
    "name": "GetFoo",
    "version": ["1.0.0"]
  },
  "secure": false,
  "level": 30,
  "msg": "GetFoo handled: 200",
  "time": "2012-02-07T20:30:31.896Z",
  "v": 0
}
```

_ `timers` 字段显示每个处理程序运行的时间（以微秒为单位）。默认情况下，Restify 将为每个路由，每个处理程序记录此信息。但如果您决定包含嵌套的处理程序，则可以使用 Request 的 [startHandlerTimer](/api/request/#starthandlertimer) 和 [endHandlerTimer](/api/request/#endhandlertimer) API 自行跟踪计时。当日志事件发出时，您还可以侦听 auditlog 事件并获取与上面的日志对象相同的信息。例如：_

```javascript
SERVER.on('auditlog', function (data) {
  // 对日志做一些处理
});
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 处理程序

### metrics

该模块包含以下用于 restify  `after` 事件的插件，例如，`server.on('after', restify.plugins.metrics())`：

一个监听服务器 after 事件并发出关于该请求的信息的插件。

**参数**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 一个选项对象
    -   `opts.server` **Server** restify 服务器
-   `callback` **createMetrics~callback** 一个回调函数

**Examples**

```javascript
server.on('after', restify.plugins.metrics({ server: server },
  function (err, metrics, req, res, route) {
    // 指标是一个包含有关请求信息的对象
  })
);
```

返回 **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 返回一个适用于 restify 服务器 `after` 事件的函数

## 类型

### metrics~callback

指标插件使用的回调

类型：[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)

**参数**

-   `err` **[Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)** 
-   `metrics` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 有关请求的指标
    -   `metrics.statusCode` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 响应的状态码。在 `uncaughtException` 的情况下可以是 undefined
    -   `metrics.method` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** http 请求动词
    -   `metrics.totalLatency` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 延迟包括请求刷新和完成所有处理程序的时间
    -   `metrics.latency` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 请求刷新时的延迟
    -   `metrics.preLatency` **([Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) | null)** `pre` 处理程序的延迟
    -   `metrics.useLatency` **([Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) | null)** `use` 处理程序的延迟
    -   `metrics.routeLatency` **([Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) | null)** `route` 处理程序的延迟
    -   `metrics.path` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** `req.path()` 的值
    -   `metrics.inflightRequests` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** restify 中挂起的 inflight 请求的数量
    -   `metrics.unifinishedRequests` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 与 `inflightRequests` 相同
    -   `metrics.connectionState` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 可以是 `'close'` 或 `undefined`。如果这个值被设置，err 将是一个相应的 `RequestCloseError`。如果 connectionState 是 `'close'`，那么 `statusCode` 不可用，因为在写入响应之前连接已被切断。
-   `req` **[Request](https://developer.mozilla.org/Add-ons/SDK/High-Level_APIs/request)** 请求对象
-   `res` **[Response](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/HTML5)** 响应对象
-   `route` **Route** 为请求提供服务的路由对象
