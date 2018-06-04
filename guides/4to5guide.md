---
title: 将4.x迁移到5.x迁移
permalink: /docs/4to5/
---

## 介绍

restify 5.0 终于来了！非常感谢所有我们的贡献者。5.x 修复了大量的错误，添加了一些新功能，并介绍了一些重大更改。本指南有助于理解自上次 4.x 发布以来发生的所有主要更改。更详细的更改日志可以在 CHANGES.md 中找到。

#### queryParser() and bodyParser()

默认情况下，queryParser 和 bodyParser 不再将 req.query 和 req.body 映射到 req.params。为了获得旧的行为，请在使用这些插件时启用 mapParams 行为。

### restify-errors

以前在 `restify.errors` 命名空间中可用的错误现在位于其自己的[存储库](https://github.com/restify/errors)中，并且在 npm 上[独立发布](https://www.npmjs.com/package/restify-errors)。
`restify-errors` 可以独立于任何其他项目中的 restify 来使用，以用于定制错误类和链错误。

### restify-clients

All restify clients have been broken out into their own
[repository](https://github.com/restify/clients), and are published
[independently on npm](https://www.npmjs.com/package/restify-clients).
所有 restify 客户端已被分离到它们自己的[存储库](https://github.com/restify/clients)中，并在 npm 上[独立发布](https://www.npmjs.com/package/restify-clients)。

### server.on('restifyError', ...)

现在 restify 会发出一个通用的错误事件。所有传递给 `next()` 的错误都会触发此错误事件。如果您为某类错误附加了特定的侦听器，那么最具体的侦听器将首先被触发，而通用触发器会最后被触发。

```javascript
// 在某些路由中，创建了一个 500 错误
server.get('/', function(req, res, next) {
  return next(new InternalServerError('oh noes!'));
  // 这将**首先**触发 InternalServerError，允许您在触发 restifyError 事件之前以某种方式处理它。
  // 通用处理程序的语义意味着它应该始终被触发，但并不意味着我们不应该让您在关心他们的错误处理程序中首先处理它。
  // 这只有在我们连续发出事件时才有可能。
});

// 处理 500 错误
server.on('InternalServer', function(req, res, err, cb) {
  // 此事件首先被触发。
  // 您可以在此处使用 `err.handled = true` 来注释错误，因为我们**必须**总在这之后触发通用的处理程序。
  err.handled = true;
  return cb();
});

// 通用错误处理程序
server.on('restifyError', function(req, res, err, cb) {
  // 这个事件最后被触发了。做一些通用的指标/日志记录工作
  if (!err.handled) {
    // 做一些事情
  }
  return cb();
});
```

### server.on('redirect', ...)

当使用 `res.redirect()` 时，restify 现在会发出重定向事件。该事件是由重定向到新位置时触发的。

```javascript
SERVER.on('redirect', function (newLocation) {
  // newLocation 是我们重定向到的新网址。
});
```

### server.on('NotFound', ...)
### server.on('MethodNotAllowed', ...)
### server.on('VersionNotAllowed', ...)
### server.on('UnsupportedMediaType', ...)

现在已经对这四类 restify 错误事件进行了规范化，可以像其他错误事件一样进行操作。以前，侦听这些事件需要您发送响应。现在已经可以像其他错误事件一样正常化工作：

```javascript
server.on('NotFound', function(req, res, err, cb) {
  // 在这里做一些日志记录或指标收集。
  // 如果您想发送一个自定义的响应，您可以通过在错误对象上设置响应主体来完成。
  err.body = "whoops! can't find your stuff!"; // 错误对象的主体成为响应
  return cb();
});
```

### CORS

CORS 已从 restify 核心代码中删除。对于 CORS 支持，请使用 [TabDigital](https://github.com/TabDigital/restify-cors-middleware) 的插件。

### 严格路由

现在可以通过 `strictRouting` 选项来支持严格路由。这允许区分具有尾部斜线的路由。默认值为 `false`，它模仿 4.x 中的行为，即去除尾部斜线。

```javascript
var server = restify.createServer({
  strictRouting: true
});
// 通过设置 strictRouting 选项，这是两条完全不同的路由
server.get('/foo/', function(req, res, next) { });
server.get('/foo', function(req, res, next) { });
```

### res.sendRaw()

restify 有一个格式化程序的概念，其中每个格式化程序都会在发送之前格式化响应的内容。已经添加了一个新的方法 `res.sendRaw()`，该方法允许在已设置预格式化内容（预先压缩，预 JSON 字符串化等）的场景中绕过格式化程序。`sendRaw` 与 `send` 具有相同的签名。

### 删除不在文档中的 API

以前版本的 restify 在主要对象上有一些未公开的导出。这些已从 5.x 中删除，其中包括：

* `restify.CORS` - 已从核心代码中去除 CORS
* `restify.httpDate` - 不在文档中
* `restify.realizeUrl` - 不在文档中

### next(err) & res.send(err)

为了帮助减少无意中暴露给客户端的错误，restify 不再为 Error 对象执行特殊的 JSON 序列化。例如：

```javascript
server.get('/sendErr', function(req, res, next) {
  res.send(new Error('where is my msg?'));
  return next();
});

server.get('/nextErr', function(req, res, next) {
  return next(new Error('where is my msg?'));
});
```

```shell
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

响应是一个空对象，因为 `JSON.stringify(err)` 返回了一个空对象。为了获得正确的序列化错误，首选方法是使用 restify-errors，它定义了 `toJSON` 方法。另外，如果您有自定义的错误类，您可以定义一个 `toJSON` 方法，它会在您的错误被字符串化时被调用。如果您有许多自定义错误类型，请考虑使用 restify-errors 来帮助您轻松创建和管理它们。最后，您可以使用 restify-errors 选择自动 `toJSON` 序列化：

```javascript
var errs = require('restify-errors');

server.get('/', function(req, res, next) {
  res.send(new errs.GoneError('gone girl'));
  return next();
});
```

```shell
$ curl -is localhost:8080/
HTTP/1.1 410 Gone
Content-Type: application/json
Content-Length: 37
Date: Fri, 03 Jun 2016 20:17:48 GMT
Connection: keep-alive

{"code":"Gone","message":"gone girl"}
```

## 弃用

以下内容目前仍受支持，但只有可用性上的支持，可能会在未来版本中删除。使用这些功能会导致 restify 在日志中打印弃用警告。

### domains

在 4.x 中，默认使用域。任何由域捕获的错误都可以通过 `server.on('uncaughtException', ...)` 事件来处理。但是，这种行为在默认情况下很难被人察觉，许多错误往往未被最终用户处理或忽视。

由于域已被弃用，我们选择默认关闭域。如果您要使用域，则可以在创建服务器时通过 `handleUncaughtExceptions` 选项将其重新打开：

```javascript
var server = restify.createServer({
  handleUncaughtExceptions: true
});
```

### next.ifError()

`next.ifError()` 功能利用了底层的域。此功能也被弃用，并且只有在 `handleUncaughtExceptions` 标志被设置为 true 时才可用。
