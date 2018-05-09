### 目录

-   [Response](#response)
    -   [cache](#cache)
    -   [noCache](#nocache)
    -   [charSet](#charset)
    -   [header](#header)
    -   [json](#json)
    -   [link](#link)
    -   [send](#send)
    -   [sendRaw](#sendraw)
    -   [set](#set)
    -   [status](#status)
    -   [redirect](#redirect)
    -   [redirect](#redirect-1)
    -   [redirect](#redirect-2)

## Response

**拓展 http.ServerResponse**

封装了所有的 Node.js [http.ServerResponse](https://nodejs.org/zh-CN/docs/latest/api/http.html) API、事件和属性，以及以下内容。

### cache

设置 `cache-control` 头。

**参数**

-   `type` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头的值（`"public"` or `"private"`）(可选，默认为 `"public"`)
                                      
-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 一个选项对象
    -   `options.maxAge` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** max-age（秒）

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 设置为报头的值。

### noCache

关闭所有与缓存相关的报头。

返回 **[Response](#response)** 自身，响应对象。

### charSet

将提供的字符集附加到响应的 `Content-Type` 上。

**参数**

-   `type` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 字符集的值

**例子**

```javascript
res.charSet('utf-8');
```

返回 **[Response](#response)** 自身，响应对象。

### header

设置响应的报头。

**参数**

-   `key` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头的名称
-   `value` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头的值

**例子**

_如果仅指定了关键字，则返回报头的值。如果指定了键和值，则设置响应头。_

```javascript
res.header('Content-Length');
// => undefined

res.header('Content-Length', 123);
// => 123

res.header('Content-Length');
// => 123

res.header('foo', new Date());
// => Fri, 03 Feb 2012 20:09:58 GMT
```

_在合适的情况下，`header()` 也可以用来自动链接报头的值：_

```javascript
res.header('x-foo', 'a');
res.header('x-foo', 'b');
// => { 'x-foo': ['a', 'b'] }
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 检索或设置的值。

### json

语法糖：

```javascript
res.contentType = 'json';
res.send({hello: 'world'});
```

**参数**

-   `code` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?**    http 状态码
-   `body` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?**    用于 json.stringify 的值
-   `headers` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 设置在响应上的报头

**例子**

```javascript
res.header('content-type', 'json');
res.send({hello: 'world'});
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象。

### link

设置链接报头。

**参数**

-   `key` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 链接的关键字
-   `value` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 链接的值

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 将报头的值设置为资源对象。

### send

发送响应对象。传递给基于 `content-type` 头的格式化程序使用的内部 `__send`。

**参数**

-   `code` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** http 状态码
-   `body` **([Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Buffer](https://nodejs.org/api/buffer.html) \| [Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error))?** 要发送的内容
-   `headers` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 任何额外的报头设置

**例子**

_您可以使用 send() 在 Node.js 的 HTTP API 上封装所有常用的 writeHead()、write() 和 end() 调用。
您可以传 `code` 和 `body`，也可以只传一个 `body`。`body`可以时一个 `Object`、`Buffer` 或 `Error`。
当您调用 `send()` 时，restify 会根据 `content-type` 指出如何格式化响应。_

```javascript
res.send({hello: 'world'});
res.send(201, {hello: 'world'});
res.send(new BadRequestError('meh'));
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象。

### sendRaw

和 `res.send()` 一样，但跳过了格式化这一步。当载体内容已经被预先格式化时，这会非常有用。
发送响应对象。完全跳过格式化程序并将原始内容传递给内部的 `__send`。

**参数**

-   `code` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)?** http 状态码
-   `body` **([Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Buffer](https://nodejs.org/api/buffer.html) \| [Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error))?** 要发送的内容
-   `headers` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 任何额外的报头设置

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象。

### set

在响应中设置多个报头。
内部使用 `header()`，启用多个值的报头。

**参数**

-   `name` **([String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object))** 报头的名称或报头对象
-   `val` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 报头的值

**例子**

```javascript
res.header('x-foo', 'a');
res.set({
  'x-foo', 'b',
  'content-type': 'application/json'
});
// =>
// {
//   'x-foo': [ 'a', 'b' ],
//   'content-type': 'application/json'
// }
```

返回 **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 自身，响应对象。

### status

设置响应中的 http 状态码。

**参数**

-   `code` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** http 状态码

**例子**

```javascript
res.status(201);
```

返回 **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** 传入的状态码。

### redirect

Redirect 是重定向的语法糖。

**参数**

-   `options` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 用来配置重定向的链接或一个选项对象
    -   `options.secure` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 是否重定向到 http 或 https
    -   `options.hostname` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 重定向位置的主机名
    -   `options.pathname` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 重定向位置的路径名
    -   `options.port` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 重定向位置的端口号
    -   `options.query` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)?** 重定向位置的查询字符串参数
    -   `options.overrideQuery` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 如果为 `true`，`options.query` 会舍弃当前 URL 上的任何现有的查询参数。默认情况下，会合并这两个值。
    -   `options.permanent` **[Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** 如果为 `true`，设置为 301。默认为 302。
-   `next` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 强制性，完成响应并触发审计记录器。

**例子**

```javascript
res.redirect({...}, next);
```

_301/302 重定向的便捷方法。使用该方法会告诉 restify 停止执行您的处理程序链。
您也可以使用选项对象。必须要有 `next`：_

```javascript
res.redirect({
  hostname: 'www.foo.com',
  pathname: '/bar',
  port: 80,                 // 默认为 80
  secure: true,             // 设置 https
  permanent: true,
  query: {
    a: 1
  }
}, next);  // => 重定向到 301 https://www.foo.com/bar?a=1
```

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 

### redirect

使用状态码和网址进行重定向。

**参数**

-   `code` **[Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)** http 重定向状态码
-   `url` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 重定向的网址
-   `next` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 强制性，完成响应并触发审计记录器。

**例子**

```javascript
res.redirect(301, 'www.foo.com', next);
```

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 

### redirect

通过网址重定向。

**参数**

-   `url` **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 重定向的网址
-   `next` **[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)** 强制性，完成响应并触发审计记录器。

**例子**

```javascript
res.redirect('www.foo.com', next);
res.redirect('/foo', next);
```

返回 **[undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)** 
