# Dtrace 指南

restify 最酷的功能之一是，无论何时添加新的路由/处理程序，它都会自动为您创建 DTrace 探针。
要使用 DTrace，您需要将 dtrace 选项传递给服务器 `restify.createServer({ dtrace: true })`。
理解它的最佳方式就是通过查看例：

```javascript
var restify = require('restify');

var server = restify.createServer({
  name: 'helloworld',
  dtrace: true
});

server.use(restify.plugins.acceptParser(server.acceptable));
server.use(restify.plugins.authorizationParser());
server.use(restify.plugins.dateParser());
server.use(restify.plugins.queryParser());
server.use(restify.plugins.urlEncodedBodyParser());

server.use(function slowHandler(req, res, next) {
  setTimeout(function() {
    return next();
  }, 250);
});

server.get({path: '/hello/:name', name: 'GetFoo'}, function respond(req, res, next) {
  res.send({
    hello: req.params.name
  });
  return next();
});

server.listen(8080, function() {
  console.log('listening: %s', server.url);
});
```

因此我们现在已经有了我们典型的 “hello world” 服务器，略有不同的是，我们引入了一个人为的 250ms 延迟。另外，请注意我们命名了我们的服务器，我们的路由以及我们所有的处理程序（函数）。尽管这是可选的，但它确实使 DTrace 更加可用。因此，如果您启动了该服务器，然后查找 DTrace 探针，则会看到如下所示的内容：

```shell
$ dtrace -l -P restify*
ID   PROVIDER            MODULE                          FUNCTION NAME
24   restify38789        mod-88f3f88                     route-start route-start
25   restify38789        mod-88f3f88                     handler-start handler-start
26   restify38789        mod-88f3f88                     handler-done handler-done
27   restify38789        mod-88f3f88                     route-done route-done
```

## route-start

|字段|类型|描述|
|-----|----|-----------|
|server name|char *|触发的 restify 服务器的名称|
|route name|char *|触发的路由名称|
|id|int|此请求的唯一 ID|
|method|char *|HTTP 请求方法|
|url|char *|（完整）HTTP URL|
|headers|char *|JSON 编码的所有请求头映射表|

## handler-start

|字段|类型|描述|
|-----|----|-----------|
|server name|char *|触发的 restify 服务器的名称|
|route name|char *|触发的路由名称|
|handler name|char *|刚刚输入的函数名称|
|id|int|此请求的唯一 ID|

## route-done

|字段|类型|描述|
|-----|----|-----------|
|server name|char *|触发的 restify 服务器的名称|
|route name|char *|触发的路由名称|
|id|int|此请求的唯一 ID|
|statusCode|int|HTTP 响应码|
|headers|char *|JSON 编码的响应头映射表|

## handler-done

|字段|类型|描述|
|-----|----|-----------|
|server name|char *|触发的 restify 服务器的名称|
|route name|char *|触发的路由名称|
|handler name|char *|刚刚输入的函数名称|
|id|int|此请求的唯一 ID|

## D 脚本示例

现在，如果您想要通过处理程序获得延迟细分，则可以执行如下操作：

```
#!/usr/sbin/dtrace -s
#pragma D option quiet

restify*:::route-start
{
  track[arg2] = timestamp;
}

restify*:::handler-start
/track[arg3]/
{
  h[arg3, copyinstr(arg2)] = timestamp;
}

restify*:::handler-done
/track[arg3] && h[arg3, copyinstr(arg2)]/
{
  @[copyinstr(arg2)] = quantize((timestamp - h[arg3, copyinstr(arg2)]) / 1000000);
  h[arg3, copyinstr(arg2)] = 0;
}

restify*:::route-done
/track[arg2]/
{
  @[copyinstr(arg1)] = quantize((timestamp - track[arg2]) / 1000000);
  track[arg2] = 0;
}
```

在同一个终端上运行服务器：

```shell
$ node helloworld.js
```

另一个 D 脚本：

```shell
$ ./helloworld.d
```

用 curl 连接几次服务器：

```shell
$ for i in {1..10} ; do curl -is http://127.0.0.1:8080/hello/mark ; done
```

然后对 D 脚本按下 Ctrl-C，您会看到堆栈底部的“slowHandler”，表明它是这个管道中的绝大数的延迟。

```shell
handler-6
value  ------------- Distribution ------------- count
-1 |                                         0
0  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 10
1  |                                         0

parseAccept
value  ------------- Distribution ------------- count
-1 |                                         0
0  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 10
1  |                                         0

parseAuthorization
value  ------------- Distribution ------------- count
-1 |                                         0
0  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 10
1  |                                         0

parseDate
value  ------------- Distribution ------------- count
-1 |                                         0
0  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 10
1  |                                         0

parseQueryString
value  ------------- Distribution ------------- count
-1 |                                         0
0  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 10
1  |                                         0

parseUrlEncodedBody
value  ------------- Distribution ------------- count
-1 |                                         0
0  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 10
1  |                                         0

respond
value  ------------- Distribution ------------- count
1  |                                         0
2  |@@@@                                     1
4  |                                         0
8  |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@     9
16 |                                         0

slowHandler
value  ------------- Distribution ------------- count
64  |                                         0
128 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@     9
256 |@@@@                                     1
512 |                                         0

getfoo
value  ------------- Distribution ------------- count
64  |                                         0
128 |@@@@                                     1
256 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@     9
512 |
```
