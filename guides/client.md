# 客户端指南

事实上在 restify 中有三个独立的客户端：

* **JsonClient:** 发送并期望是 application/json
* **StringClient:** 发送网址编码的请求并期望是 text/plain
* **HttpClient:** 封装了 Node.js 的 http/https 库

这个想法是，如果您想支持“典型” control-plane REST API，您可能需要 `JsonClient`，或者如果您使用其他序列化模式（比如 XML），您会编写自己的客户端来扩展 `StringClient`。如果您需要支持流，您需要在 `HttpClient` 之上做一些工作，因为 `StringClient` 对缓冲请求/响应并不友好。

所有客户端都支持以指数回退重试以获取 TCP 连接；他们不会像之前版本的 restify 客户端那样对 5xx 错误码执行重试。您可以将 `retry` 设置为 `false`，以完全禁用此逻辑。此外，所有客户端都支持 `connectTimeout` 字段，*每次重试*都使用该字段。默认情况下不设置 `connectTimeout`，所以您最终得到了 Node.js 套接字的默认值。

这是一个连接 [Joyent CloudAPI](https://apidocs.joyent.com/cloudapi/) 的例子：

```javascript
var restify = require('restify-clients');

// 创建一个 JSON 客户端
var client = restify.createJsonClient({
  url: 'https://us-east-1.api.joyent.com'
});


client.basicAuth('$login', '$password');
client.get('/my/machines', function(err, req, res, obj) {
  assert.ifError(err);

  console.log(JSON.stringify(obj, null, 2));
});
```

按照简洁模式，客户端可以使用字符串 URL 而不是选项对象进行初始化：

```javascript
var restify = require('restify-clients');

var client = restify.createJsonClient('https://us-east-1.api.joyent.com');
```

请注意，所有的进阶文档都提到了像 `get/put/del` 这样的采用字符串路径的“简洁”形式。
您还可以将一个对象传递给具有额外参数（特别是报头）的任何方法：

```javascript
var options = {
  path: '/foo/bar',
  headers: {
    'x-foo': 'bar'
  },
  retry: {
    'retries': 0
  },
  agent: false
};

client.get(options, function(err, req, res) { .. });
```

如果在将请求发送到服务器之前您需要在请求中插入其他报头，您可以在创建客户端时提供同步回调函数作为 `signRequest` 选项。这对 [node-http-signature](https://github.com/joyent/node-http-signature) 特别有用，因为它需要附加所选传出报头的加密签名。如果提供，将使用单个参数调用此回调函数：传出的 `http.ClientRequest` 对象。

## JsonClient

JSON 客户端是与 restify-clients 绑定在一起的最高级客户端；它会导出一组直接映射到 HTTP 动词的方法。所有的回调看起来像函数 `function(err, req, res, [obj])`，其中 `obj` 是可选的，取决于是否返回内容。HTTP 状态码不被解释，所以如果服务器返回 4xx 或者带有 JSON 负载的内容，`obj` 将会是该 JSON 负载。但是，如果服务器返回状态码大于等于 400（这将是其中一个 HTTP 错误），则会设置 `err`。如果 `err` 看起来像 `RestError`：

```javascript
{
  "code": "FooError",
  "message": "some foo happened"
}
```

那么 `err` 会为您 "upconverted" 为 `RestError`。否则会是一个 `HttpError`。

### createJsonClient(options)

```javascript
var client = restify.createJsonClient({
  url: 'https://api.us-east-1.joyent.com',
  version: '*'
});
```

选项：

|名称|类型|描述|
|----|----|-----------|
|accept|String|发送可接受的报头|
|connectTimeout|Number|套接字的超时时间|
|requestTimeout|Number|请求完成的超时时间|
|dtrace|Object|node-dtrace-provider 句柄|
|gzip|Object|将在发送时使用 `content-encoding: gzip` 压缩数据|
|headers|Object|在所有请求中设置 HTTP 报头|
|log|Object|[bunyan](https://github.com/trentm/node-bunyan) 实例|
|retry|Object|为 node-retry 选项提供；"false" 禁用重试；默认重试 4 次|
|signRequest|Function|在发送请求之前插入报头的同步回调|
|url|String|要连接的完整 URL|
|userAgent|String|使用 user-agent 字符串；restify 会插入一个，但您可以覆盖它|
|version|String|semver 字符串来设置可接受版本|

### get(path, callback)

执行 HTTP get；如果没有有效负载被返回，`obj` 默认为 `{}`（因此您不会得到一堆空指针错误）。

```javascript
client.get('/foo/bar', function(err, req, res, obj) {
  assert.ifError(err);
  console.log('%j', obj);
});
```

### head(path, callback)

和 `get` 一样，但没有 `obj`：

```javascript
client.head('/foo/bar', function(err, req, res) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
});
```

### post(path, object, callback)

需要将一个完整的对象序列化并发送到服务器。

```javascript
client.post('/foo', { hello: 'world' }, function(err, req, res, obj) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
  console.log('%j', obj);
});
```

### put(path, object, callback)

和 `post` 一样：

```javascript
client.put('/foo', { hello: 'world' }, function(err, req, res, obj) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
  console.log('%j', obj);
});
```

### del(path, callback)

`del` 不需要内容，正如您所知的，它不应该有内容：

```javascript
client.del('/foo/bar', function(err, req, res) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
});
```

## StringClient

`StringClient` 是构建 `JsonClient` 的基础，并为您提供了编写其他缓冲/解析客户端（如称 XML 客户端）的基础。如果您需要与某些“原始” HTTP 服务器交互，那么 `StringClient` 就是您想要的，因为默认情况下，它会为您的内容上传提供 `application/x-www-form-url-encoded`，并以 `text/plain` 格式下载。要扩展一个 `StringClient`，请查看 `JsonClient` 的源代码。实际上，您扩展它，需要在构造函数中设置适当的选项，并实现 `write`（针对 put/post）和 `parse` 方法（针对所有 HTTP 主体），这样就可以了。

### createStringClient(options)

```javascript
var client = restify.createStringClient({
  url: 'https://example.com'
})
```

### get(path, callback)

执行 HTTP get；如果没有有效载荷被返回，那么 `data` 默认为 `''`（因此您不会得到一堆空指针错误）。

```javascript
client.get('/foo/bar', function(err, req, res, data) {
  assert.ifError(err);
  console.log('%s', data);
});
```

### head(path, callback)

和 `get` 一样，但没有 `data`：

```javascript
client.head('/foo/bar', function(err, req, res) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
});
```

### post(path, object, callback)

需要将一个完整的对象序列化并发送到服务器。

```javascript
client.post('/foo', { hello: 'world' }, function(err, req, res, data) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
  console.log('%s', data);
});
```

### put(path, object, callback)

和 `post` 一样：

```javascript
client.put('/foo', { hello: 'world' }, function(err, req, res, data) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
  console.log('%s', data);
});
```

### del(path, callback)

`del` 不需要内容，正如您所知的，它不应该有内容：

```javascript
client.del('/foo/bar', function(err, req, res) {
  assert.ifError(err);
  console.log('%d -> %j', res.statusCode, res.headers);
});
```

## HttpClient

`HttpClient` 是 restify 中发布的最低级别的客户端，基本上只是 Node.js 的 http/https 模块顶部的一些语法糖（与其他客户端一样的 HTTP 方法）。如果你想用 restify 进行流处理，这将非常有用。请注意，以下事件很不幸地被命名为  `result` 而不是 `response`（因为[事件 'response'](http://nodejs.org/docs/latest/api/all.html#event_response_) 已被占用）。

```javascript
client = restify.createClient({
  url: 'http://127.0.0.1'
});

client.get('/str/mcavage', function(err, req) {
  assert.ifError(err); // 连接错误

  req.on('result', function(err, res) {
    assert.ifError(err); // HTTP 状态码 >= 400

    res.body = '';
    res.setEncoding('utf8');
    res.on('data', function(chunk) {
      res.body += chunk;
    });

    res.on('end', function() {
      console.log(res.body);
    });
  });
});
```

或写一个：

```javascript
client.post(opts, function(err, req) {
  assert.ifError(connectErr);

  req.on('result', function(err, res) {
    assert.ifError(err);
    res.body = '';
    res.setEncoding('utf8');
    res.on('data', function(chunk) {
      res.body += chunk;
    });

    res.on('end', function() {
      console.log(res.body);
    });
  });

  req.write('hello world');
  req.end();
});
```

请注意 `get/head/del` 都会为您调用 `req.end()`，所以您不能在这些方法上写数据。否则，会在所有与 `JsonClient/StringClient` 同名的方法中存在。

希望扩展 `HttpClient` 的人应该看看源码，注意可能需要覆写 `read` 和 `write`。

### 代理

有几个选项用于启用 http 客户端代理。以下选项可用于设置代理网址：

```javascript
// 在客户端配置中设置代理选项
restify.createClient({
  proxy: 'http://127.0.0.1'
});
```

来自环境变量：

```shell
$ export HTTPS_PROXY = 'https://127.0.0.1'
$ export HTTP_PROXY = 'http://127.0.0.1'
```

有一个选项可以禁止在单个网址或所有网址上使用代理服务器。这可以通过设置环境变量来启用。

不代理任何网址的请求

```shell
$ export NO_PROXY='*'
```

不代理本地网址的请求

```shell
$ export NO_PROXY='127.0.0.1'
```

不代理本地网址其中端口为 8000 的请求

```shell
$ export NO_PROXY='localhost:8000'
```

不代理多个 IP 地址的请求

```shell
$ export NO_PROXY='127.0.0.1, 8.8.8.8'
```

**注意**：所请求的 url 必须与代理配置中的完整主机名或 NO_PROXY 环境变量相匹配。不执行 DNS 查找来确定主机名的 IP 地址。

### basicAuth(username, password)

此快捷方法（适用于所有客户端），只需为所有 HTTP 请求设置 `Authorization` 标头就可以了：

```javascript
client.basicAuth('mark', 'mysupersecretpassword');
```

### 升级

如果您成功与 HTTP 服务器协商升级，会发出一个带有参数 `err`、`res`、`socket` 和 `head` 的 `upgradeResult` 事件。您可以使用此功能与服务器建立 WebSockets 连接。例如，使用 [watershed](https://github.com/jclulow/node-watershed) 库：

```javascript
var ws = new Watershed();
var wskey = ws.generateKey();
var options = {
  path: '/websockets/attach',
  headers: {
    connection: 'upgrade',
    upgrade: 'websocket',
    'sec-websocket-key': wskey,
  }
};
client.get(options, function(err, res, socket, head) {
  res.once('upgradeResult', function(err2, res2, socket2, head2) {
    var shed = ws.connect(res2, socket2, head2, wskey);
    shed.on('text', function(msg) {
      console.log('message from server: ' + msg);
      shed.end();
    });
    shed.send('greetings program');
  });
});
```
