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

Parses the `Accept` header, and ensures that the server can respond to what
the client asked for. In almost all cases passing in `server.acceptable` is
all that's required, as that's an array of content types the server knows
how to respond to (with the formatters you've registered). If the request is
for a non-handled type, this plugin will return a `NotAcceptableError` (406).

Note you can get the set of types allowed from a restify server by doing
`server.acceptable`.

**Parameters**

-   `accepts` **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)>** array of accept types.

**Examples**

```javascript
server.use(restify.plugins.acceptParser(server.acceptable));
```

-   Throws **NotAcceptableError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** restify handler.

### authorizationParser

Parses out the `Authorization` header as best restify can.
Currently only HTTP Basic Auth and
[HTTP Signature](https://github.com/joyent/node-http-signature)
schemes are supported.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** an optional options object that is
                                   passed to http-signature

**Examples**

_Subsequent handlers will see `req.authorization`, which looks like above.`req.username` will also be set, and defaults to 'anonymous'.  If the scheme
is unrecognized, the only thing available in `req.authorization` will be
`scheme` and `credentials` - it will be up to you to parse out the rest._

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

-   Throws **InvalidArgumentError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### dateParser

Parses out the HTTP Date header (if present) and checks for clock skew.
If the header is invalid, a `InvalidHeaderError` (`400`) is returned.
If the clock skew exceeds the specified value,
a `RequestExpiredError` (`400`) is returned.
Where expired means the request originated at a time
before (`$now - $clockSkew`).
The default clockSkew allowance is 5m (thanks
Kerberos!)

**Parameters**

-   `clockSkew` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** allowed clock skew in seconds. (optional, default `300`)

**Examples**

```javascript
// Allows clock skew of 1m
server.use(restify.plugins.dateParser(60));
```

-   Throws **RequestExpiredError** 
-   Throws **InvalidHeaderError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** restify handler.

### queryParser

Parses the HTTP query string (i.e., `/foo?id=bar&name=mark`).
If you use this, the parsed content will always be available in `req.query`,
additionally params are merged into `req.params`.
You can disable by passing in `mapParams: false` in the options object.

Many options correspond directly to option defined for the underlying
[`qs.parse`](https://github.com/ljharb/qs).

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** an options object
    -   `options.mapParams` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** disable passing (optional, default `true`)
    -   `options.mapParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Copies parsed query parameters
        into`req.params`. (optional, default `false`)
    -   `options.overrideParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Only applies when if
        mapParams true.
        When true, will stomp on req.params field when existing value is found. (optional, default `false`)
    -   `options.allowDots` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Transform `?foo.bar=baz` to a
        nested object: `{foo: {bar: 'baz'}}`. (optional, default `false`)
    -   `options.arrayLimit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** Only transform `?a[$index]=b`
        to an array if `$index` is less than `arrayLimit`. (optional, default `20`)
    -   `options.depth` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** The depth limit for parsing
        nested objects, e.g. `?a[b][c][d][e][f][g][h][i]=j`. (optional, default `5`)
    -   `options.parameterLimit` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** Maximum number of query
        params parsed. Additional params are silently dropped. (optional, default `1000`)
    -   `options.parseArrays` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Whether to parse
        `?a[]=b&a[1]=c` to an array, e.g. `{a: ['b', 'c']}`. (optional, default `true`)
    -   `options.plainObjects` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Whether `req.query` is a
        "plain" object -- does not inherit from `Object`.
        This can be used to allow query params whose names collide with Object
        methods, e.g. `?hasOwnProperty=blah`. (optional, default `false`)
    -   `options.strictNullHandling` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** If true, `?a&b=`
        results in `{a: null, b: ''}`. Otherwise, `{a: '', b: ''}`. (optional, default `false`)

**Examples**

```javascript
server.use(restify.plugins.queryParser({ mapParams: false }));
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### jsonp

Parses the jsonp callback out of the request.
Supports checking the query string for `callback` or `jsonp` and ensuring
that the content-type is appropriately set if JSONP params are in place.
There is also a default `application/javascript` formatter to handle this.

You _should_ set the `queryParser` plugin to run before this, but if you
don't this plugin will still parse the query string properly.

**Examples**

```javascript
var server = restify.createServer();
server.use(restify.plugins.jsonp());
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### bodyParser

Blocks your chain on reading and parsing the HTTP request body.  Switches on
`Content-Type` and does the appropriate logic.  `application/json`,
`application/x-www-form-urlencoded` and `multipart/form-data` are currently
supported.

Parses `POST` bodies to `req.body`. automatically uses one of the following
parsers based on content type:

-   `urlEncodedBodyParser(options)` - parses url encoded form bodies
-   `jsonBodyParser(options)` - parses JSON POST bodies
-   `multipartBodyParser(options)` - parses multipart form bodies

All bodyParsers support the following options:

-   `options.mapParams` - default false. copies parsed post body values onto
    req.params
-   `options.overrideParams` - default false. only applies when if
    mapParams true. when true, will stomp on req.params value when
    existing value is found.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** an option object
    -   `options.maxBodySize` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** The maximum size in bytes allowed in
        the HTTP body. Useful for limiting clients from hogging server memory.
    -   `options.mapParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** if `req.params` should be filled with
        parsed parameters from HTTP body.
    -   `options.mapFiles` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** if `req.params` should be filled with
        the contents of files sent through a multipart request.
        [formidable](https://github.com/felixge/node-formidable) is used internally
        for parsing, and a file is denoted as a multipart part with the `filename`
        option set in its `Content-Disposition`. This will only be performed if
        `mapParams` is true.
    -   `options.overrideParams` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** if an entry in `req.params`
        should be overwritten by the value in the body if the names are the same.
        For instance, if you have the route `/:someval`,
        and someone posts an `x-www-form-urlencoded`
        Content-Type with the body `someval=happy` to `/sad`, the value will be
        `happy` if `overrideParams` is `true`, `sad` otherwise.
    -   `options.multipartHandler` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** a callback to handle any
        multipart part which is not a file.
        If this is omitted, the default handler is invoked which may
        or may not map the parts into `req.params`, depending on
        the `mapParams`-option.
    -   `options.multipartFileHandler` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** a callback to handle any
        multipart file.
        It will be a file if the part has a `Content-Disposition` with the
        `filename` parameter set. This typically happens when a browser sends a
        form and there is a parameter similar to `<input type="file" />`.
        If this is not provided, the default behaviour is to map the contents
        into `req.params`.
    -   `options.keepExtensions` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** if you want the uploaded
        files to include the extensions of the original files
        (multipart uploads only).
        Does nothing if `multipartFileHandler` is defined.
    -   `options.uploadDir` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** Where uploaded files are
        intermediately stored during transfer before the contents is mapped
        into `req.params`.
        Does nothing if `multipartFileHandler` is defined.
    -   `options.multiples` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** if you want to support html5 multiple
        attribute in upload fields.
    -   `options.hash` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** If you want checksums calculated for
        incoming files, set this to either `sha1` or `md5`.
    -   `options.rejectUnknown` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** Set to `true` if you want to end
        the request with a `UnsupportedMediaTypeError` when none of
        the supported content types was given.
    -   `options.requestBodyOnGet` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)**  Parse body of a GET
        request. (optional, default `false`)
    -   `options.reviver` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)?** `jsonParser` only. If a function,
        this prescribes how the value originally produced by parsing is transformed,
        before being returned. For more information check out
        `JSON.parse(text[, reviver])`.
    -   `options.maxFieldsSize` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** `multipartParser`
        only.
        Limits the amount of memory all fields together (except files)
        can allocate in bytes.
        The default size is `2 * 1024 * 1024` bytes _(2MB)_. (optional, default `2*1024*1024`)

**Examples**

```javascript
server.use(restify.plugins.bodyParser({
    maxBodySize: 0,
    mapParams: true,
    mapFiles: false,
    overrideParams: false,
    multipartHandler: function(part) {
        part.on('data', function(data) {
          // do something with the multipart data
        });
    },
   multipartFileHandler: function(part) {
        part.on('data', function(data) {
          // do something with the multipart file data
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

-   Throws **UnsupportedMediaTypeError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### requestLogger

Sets up a child [bunyan](https://github.com/trentm/node-bunyan) logger with
the current request id filled in, along with any other parameters you define.

You can pass in no options to this, in which case only the request id will be
appended, and no serializers appended (this is also the most performant); the
logger created at server creation time will be used as the parent logger.
This logger can be used normally, with [req.log](#request-api).

This plugin does _not_ log each individual request. Use the Audit Logging
plugin or a custom middleware for that use.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** an options object
    -   `options.headers` **[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)?** A list of headers to transfer from
                                         the request to top level props on the log.

**Examples**

```javascript
server.use(restify.plugins.requestLogger({
    properties: {
        foo: 'bar'
    },
    serializers: {...}
}));
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### gzipResponse

If the client sends an `accept-encoding: gzip` header (or one with an
appropriate q-val), then the server will automatically gzip all
response data.
Note that only `gzip` is supported, as this is most widely supported by
clients in the wild.
This plugin will overwrite some of the internal streams, so any
calls to `res.send`, `res.write`, etc., will be compressed.  A side effect is
that the `content-length` header cannot be known, and so
`transfer-encoding: chunked` will _always_ be set when this is in effect.
This plugin has no impact if the client does not send
`accept-encoding: gzip`.

<https://github.com/restify/node-restify/issues/284>

**Parameters**

-   `opts` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** an options object, see: zlib.createGzip

**Examples**

```javascript
server.use(restify.plugins.gzipResponse());
```

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

### serveStatic

Serves static files.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** an options object

**Examples**

_The serveStatic module is different than most of the other plugins, in that
it is expected that you are going to map it to a route, as below:_

```javascript
server.get('/zh-CN/docs/current/*', restify.plugins.serveStatic({
  directory: './documentation/v1',
  default: 'index.html'
}));
```

_The above `route` and `directory` combination will serve a file located in
`./documentation/v1/zh-CN/docs/current/index.html` when you attempt to hit
`http://localhost:8080/zh-CN/docs/current/`. If you want the serveStatic module to
serve files directly from the `/documentation/v1` directory
(and not append the request path `/zh-CN/docs/current/`),
you can set the `appendRequestPath` option to `false`, and the served file
would be `./documentation/v1/index.html`, in the previous example.The plugin will enforce that all files under `directory` are served.
The `directory` served is relative to the process working directory.
You can also provide a `default` parameter such as index.html for any
directory that lacks a direct file match.
You can specify additional restrictions by passing in a `match` parameter,
which is just a `RegExp` to check against the requested file name.
Additionally, you may set the `charSet` parameter, which will append a
character set to the content-type detected by the plugin.
For example, `charSet: 'utf-8'` will result in HTML being served with a
`Content-Type` of `text/html; charset=utf-8`.
Lastly, you can pass in a `maxAge` numeric, which will set the
`Cache-Control` header. Default is `3600` (1 hour).An additional option for serving a static file is to pass `file` in to the
serveStatic method as an option. The following will serve index.html from
the documentation/v1/ directory anytime a client requests `/home/`._

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

-   Throws **MethodNotAllowedError** \|
-   Throws **NotAuthorizedError** 
-   Throws **ResourceNotFoundError** 

Returns **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** Handler

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
