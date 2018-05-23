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
        -   [Using an external storage mechanism for key/bucket mappings.](#using-an-external-storage-mechanism-for-keybucket-mappings)
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

Creates an API rate limiter that can be plugged into the standard
restify request handling pipeline.

`restify` ships with a fairly comprehensive implementation of
[Token bucket](http://en.wikipedia.org/wiki/Token_bucket), with the ability
to throttle on IP (or x-forwarded-for) and username (from `req.username`).
You define "global" request rate and burst rate, and you can define
overrides for specific keys.
Note that you can always place this on per-URL routes to enable
different request rates to different resources (if for example, one route,
like `/my/slow/database` is much easier to overwhlem
than `/my/fast/memcache`).

If a client has consumed all of their available rate/burst, an HTTP response
code of `429`
[Too Many Requests](http://tools.ietf.org/html/draft-nottingham-http-new-status-03#section-4)
is returned.

This throttle gives you three options on which to throttle:
username, IP address and 'X-Forwarded-For'. IP/XFF is a /32 match,
so keep that in mind if using it.  Username takes the user specified
on req.username (which gets automagically set for supported Authorization
types; otherwise set it yourself with a filter that runs before this).

In both cases, you can set a `burst` and a `rate` (in requests/seconds),
as an integer/float.  Those really translate to the `TokenBucket`
algorithm, so read up on that (or see the comments above...).

In either case, the top level options burst/rate set a blanket throttling
rate, and then you can pass in an `overrides` object with rates for
specific users/IPs.  You should use overrides sparingly, as we make a new
TokenBucket to track each.

On the `options` object ip and username are treated as an XOR.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** required options with:
    -   `options.burst` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** burst
    -   `options.rate` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** rate
    -   `options.ip` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** ip
    -   `options.username` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** username
    -   `options.xff` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** xff
    -   `options.setHeaders` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Set response headers for rate,
                                      limit (burst) and remaining. (optional, default `false`)
    -   `options.overrides` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** overrides
    -   `options.tokensTable` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** a storage engine this plugin will
                                     use to store throttling keys -> bucket mappings.
                                     If you don't specify this, the default is to
                                     use an in-memory O(1) LRU, with 10k distinct
                                     keys.  Any implementation just needs to support
                                     put/get.
    -   `options.maxKeys` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** If using the default
                                     implementation, you can specify how large you
                                     want the table to be. (optional, default `10000`)

**Examples**

_An example options object with overrides:_

```javascript
{
  burst: 10,  // Max 10 concurrent requests (if tokens)
  rate: 0.5,  // Steady state: 1 request / 2 seconds
  ip: true,   // throttle per IP
  overrides: {
    '192.168.1.1': {
      burst: 0,
      rate: 0    // unlimited
  }
}
```

-   Throws **TooManyRequestsError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### requestExpiry

Request Expiry can be used to throttle requests that have already exceeded
their client timeouts. Requests can be sent with a configurable client
timeout header, e.g. 'x-request-expiry-time', which gives in absolute ms
since epoch, when this request will be timed out by the client.

This plugin will throttle all incoming requests via a 504 where
'x-request-expiry-time' less than Date.now() -- since these incoming requests
have already been timed out by the client. This prevents the server from
processing unnecessary requests.

Request expiry will use headers to tell if the incoming request has expired.
There are two options for this plugin:
 1\. Absolute Time
    _ Time in Milliseconds since Epoch when this request should be
    considered expired
 2\. Timeout
    _ The request start time is supplied
    _ A timeout, in milliseconds, is given
    _ The timeout is added to the request start time to arrive at the
      absolute time in which the request is considered expired

#### Using an external storage mechanism for key/bucket mappings.

By default, the restify throttling plugin uses an in-memory LRU to store
mappings between throttling keys (i.e., IP address) to the actual bucket that
key is consuming.  If this suits you, you can tune the maximum number of keys
to store in memory with `options.maxKeys`; the default is 10000.

In some circumstances, you want to offload this into a shared system, such as
Redis, if you have a fleet of API servers and you're not getting steady
and/or uniform request distribution.  To enable this, you can pass in
`options.tokensTable`, which is simply any Object that supports `put` and
`get` with a `String` key, and an `Object` value.

**Parameters**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** an options object
    -   `opts.absoluteHeader` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** The header key to be used for
                                          the expiry time of each request.
    -   `opts.startHeader` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** The header key for the start time
                                          of the request.
    -   `opts.timeoutHeader` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** The header key for the time in
                                          milliseconds that should ellapse before
                                          the request is considered expired.

**Examples**

_The only option provided is `header` which is the request header used
to specify the client timeout._

```javascript
server.use(restify.plugins.requestExpiry({
    header: 'x-request-expiry-time'
});
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### inflightRequestThrottle

The `inflightRequestThrottle` module allows you to specify an upper limit to
the maximum number of inflight requests your server is able to handle. This
is a simple heuristic for protecting against event loop contention between
requests causing unacceptable latencies.

The custom error is optional, and allows you to specify your own response
and status code when rejecting incoming requests due to too many inflight
requests. It defaults to `503 ServiceUnavailableError`.

This plugin should be registered as early as possibly in the middleware stack
using `pre` to avoid performing unnecessary work.

**Parameters**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** configure this plugin
    -   `opts.limit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** maximum number of inflight requests the server
           will handle before returning an error
    -   `opts.err` **[Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)** A restify error used as a response when the
           inflight request limit is exceeded
    -   `opts.server` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** the instance of the restify server this
           plugin will throttle.

**Examples**

```javascript
var errors = require('restify-errors');
var restify = require('restify');

var server = restify.createServer();
const options = { limit: 600, server: server };
options.res = new errors.InternalServerError();
server.pre(restify.plugins.inflightRequestThrottle(options));
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** middleware to be registered on server.pre

### cpuUsageThrottle

cpuUsageThrottle is a middleware that rejects a variable number of requests
(between 0% and 100%) based on a historical view of CPU utilization of a
Node.js process. Essentially, this plugin allows you to define what
constitutes a saturated Node.js process via CPU utilization and it will
handle dropping a % of requests based on that definiton. This is useful when
you would like to keep CPU bound tasks from piling up causing an increased
per-request latency.

The algorithm asks you for a maximum CPU utilization rate, which it uses to
determine at what point it should be rejecting 100% of traffic. For a normal
Node.js service, this is 1 since Node is single threaded. It uses this,
paired with a limit that you provide to determine the total % of traffic it
should be rejecting. For example, if you specify a limit of .5 and a max of
1, and the current EWMA (next paragraph) value reads .75, this plugin will
reject approximately 50% of all requests.

When looking at the process' CPU usage, this algorithm will take a load
average over a user specified interval. example, if given an interval of
250ms, this plugin will attempt to record the average CPU utilization over
250ms intervals. Due to contention for resources, the duration of each
average may be wider or narrower than 250ms. To compensate for this, we use
an exponentially weighted moving average. The EWMA algorithm is provided by
the ewma module. The parameter for configuring the EWMA is halfLife. This
value controls how quickly each load average measurment decays to half it's
value when being represented in the current average. For example, if you
have an interval of 250, and a halfLife of 250, you will take the previous
ewma value multiplied by 0.5 and add it to the new CPU utilization average
measurement multiplied by 0.5. The previous value and the new measurement
would each represent 50% of the new value. A good way of thinking about the
halfLife is in terms of how responsive this plugin will be to spikes in CPU
utilization. The higher the halfLife, the longer CPU utilization will have
to remain above your defined limit before this plugin begins rejecting
requests and, converserly, the longer it will have to drop below your limit
before the plugin begins accepting requests again. This is a knob you will
want to with play when trying to determine the ideal value for your use
case.

For a better understanding of the EWMA algorithn, refer to the documentation
for the ewma module.

**Parameters**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** Configure this plugin.
    -   `opts.limit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** The point at which restify will begin
           rejecting a % of all requests at the front door.
           This value is a percentage.
           For example 0.8 === 80% average CPU utilization. Defaults to 0.75.
    -   `opts.max` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** The point at which restify will reject 100% of
           all requests at the front door. This is used in conjunction with limit to
           determine what % of traffic restify needs to reject when attempting to
           bring the average load back to the user requested values. Since Node.js is
           single threaded, the default for this is 1. In some rare cases, a Node.js
           process can exceed 100% CPU usage and you will want to update this value.
    -   `opts.interval` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** How frequently we calculate the average CPU
           utilization. When we calculate an average CPU utilization, we calculate it
           over this interval, and this drives whether or not we should be shedding
           load. This can be thought of as a "resolution" where the lower this value,
           the higher the resolution our load average will be and the more frequently
           we will recalculate the % of traffic we should be shedding. This check
           is rather lightweight, while the default is 250ms, you should be able to
           decrease this value without seeing a significant impact to performance.
    -   `opts.halfLife` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** When we sample the CPU usage on an
           interval, we create a series of data points.
           We take these points and calculate a
           moving average. The halfLife indicates how quickly a point "decays" to
           half it's value in the moving average. The lower the halfLife, the more
           impact newer data points have on the average. If you want to be extremely
           responsive to spikes in CPU usage, set this to a lower value. If you want
           your process to put more emphasis on recent historical CPU usage when
           determininng whether it should shed load, set this to a higher value. The
           unit is in ms. Defaults to 250.

**Examples**

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

_You can also update the plugin during runtime using the `.update()` function.
This function accepts the same `opts` object as a constructor._

```javascript
var plugin = restify.plugins.cpuUsageThrottle(options);
server.pre(plugin);

plugin.update({ limit: .4, halfLife: 5000 });
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** middleware to be registered on server.pre

### conditionalHandler

Runs first handler that matches to the condition

**Parameters**

-   `candidates` **([Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)>)** candidates
    -   `candidates.handler` **([Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)>)** handler(s)
    -   `candidates.version` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>)?** '1.1.0', ['1.1.0', '1.2.0']
    -   `candidates.contentType` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** accepted content type, '\*\\/json'

**Examples**

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
  // Array of handlers
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
// 'accept-version': '3.x', accept: 'application/json' => '3.x, json'
```

-   Throws **InvalidVersionError** 
-   Throws **UnsupportedMediaTypeError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### conditionalRequest

Returns a set of plugins that will compare an already set `ETag` header with
the client's `If-Match` and `If-None-Match` header, and an already set
Last-Modified header with the client's `If-Modified-Since` and
`If-Unmodified-Since` header.

You can use this handler to let clients do nice HTTP semantics with the
"match" headers.  Specifically, with this plugin in place, you would set
`res.etag=$yourhashhere`, and then this plugin will do one of:

-   return `304` (Not Modified) [and stop the handler chain]
-   return `412` (Precondition Failed) [and stop the handler chain]
-   Allow the request to go through the handler chain.

The specific headers this plugin looks at are:

-   `Last-Modified`
-   `If-Match`
-   `If-None-Match`
-   `If-Modified-Since`
-   `If-Unmodified-Since`

**Examples**

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

-   Throws **BadRequestError** 
-   Throws **PreconditionFailedError** 

Returns **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)>** Handlers

### auditLogger

**Parameters**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** The options object.
    -   `opts.log` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** The logger.
    -   `opts.event` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** The event from the server which initiates the
        log, one of 'pre', 'routed', or 'after'
    -   `opts.context` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** The optional context function of signature
        f(req, res, route, err).  Invoked each time an audit log is generated. This
        function can return an object that customizes the format of anything off the
        req, res, route, and err objects. The output of this function will be
        available on the `context` key in the audit object.
    -   `opts.server` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** The restify server, used to emit
        the audit log object programmatically
    -   `opts.printLog` **[boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Whether to print the log
        via the logger. (optional, default `true`)

**Examples**

_Audit logging is a special plugin, as you don't use it with `.use()`
but with the `after` event:_

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
      "user-agent": "curl/7.19.7 (universal-apple-darwin10.0)
         libcurl/7.19.7 OpenSSL/0.9.8r zlib/1.2.3",
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
      "access-control-allow-headers": "Accept, Accept-Version,
         Content-Length, Content-MD5, Content-Type, Date, Api-Version",
      "access-control-expose-headers": "Api-Version, Request-Id,
         Response-Time",
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

_The `timers` field shows the time each handler took to run in microseconds.
Restify by default will record this information for every handler for each
route. However, if you decide to include nested handlers, you can track the
timing yourself by utilizing the Request
[startHandlerTimer](#starthandlertimerhandlername) and
[endHandlerTimer](#endhandlertimerhandlername) API.
You can also listen to auditlog event and get same above log object when
log event emits. For example_

```javascript
SERVER.on('auditlog', function (data) {
    //do some process with log
});
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### metrics

The module includes the following plugins to be used with restify's `after`
event, e.g., `server.on('after', restify.plugins.metrics());`:

A plugin that listens to the server's after event and emits information
about that request.

**Parameters**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** an options obj
    -   `opts.server` **Server** restify server
-   `callback` **createMetrics~callback** a callback fn

**Examples**

```javascript
server.on('after', restify.plugins.metrics({ server: server },
    function (err, metrics, req, res, route) {
        // metrics is an object containing information about the request
}));
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** returns a function suitable to be used
  with restify server's `after` event

## 类型

### metrics~callback

Callback used by metrics plugin

Type: [Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

-   `err` **[Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)** 
-   `metrics` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** metrics about the request
    -   `metrics.statusCode` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** status code of the response. can be
          undefined in the case of an uncaughtException
    -   `metrics.method` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** http request verb
    -   `metrics.totalLatency` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** latency includes both request is flushed
                                             and all handlers finished
    -   `metrics.latency` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** latency when request is flushed
    -   `metrics.preLatency` **([Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) | null)** pre handlers latency
    -   `metrics.useLatency` **([Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) | null)** use handlers latency
    -   `metrics.routeLatency` **([Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) | null)** route handlers latency
    -   `metrics.path` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** `req.path()` value
    -   `metrics.inflightRequests` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** Number of inflight requests pending
          in restify.
    -   `metrics.unifinishedRequests` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** Same as `inflightRequests`
    -   `metrics.connectionState` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** can be either `'close'` or
         `undefined`. If this value is set, err will be a
          corresponding `RequestCloseError`.
          If connectionState is either
          `'close'`, then the `statusCode` is not applicable since the
          connection was severed before a response was written.
-   `req` **[Request](https://developer.mozilla.org/Add-ons/SDK/High-Level_APIs/request)** the request obj
-   `res` **[Response](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/HTML5)** the response obj
-   `route` **Route** the route obj that serviced the request
