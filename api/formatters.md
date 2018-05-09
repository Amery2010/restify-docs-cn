### 目录

-   [用法](#用法)
-   [类型](#类型)
    -   [格式化程序](#格式化程序)
-   [内置的格式化程序](#内置的格式化程序)
    -   [formatText](#formattext)
    -   [formatJSON](#formatjson)
    -   [formatJSONP](#formatjsonp)
    -   [formatBinary](#formatbinary)

## 用法

Restify 附带了一系列有用的格式化程序，以便通过网络发送您准备好的回复，但您也可以自由添加您自己的格式化程序！

```javascript
function formatGraphQL(req, res, body) {
  var data = body;
  /* 对数据进行处理 */
  res.setHeader('Content-Length', Buffer.byteLength(data));
  return data;
}

var server = restify.createServer({
  formatters: {
    'application/graphql': formatGraphQL
  }
});

// 您的应用现在支持内容类型 'application/graphql'
```


## 类型

### 格式化程序

通过网络发送格式化的响应

类型：[Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)

**参数**

-   `req` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 请求对象（未使用）
-   `res` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象
-   `body` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应正文格式

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 格式化的响应数据。

## 内置的格式化程序

restify 预先加载了一组常用格式的格式化程序。

### formatText

通过调用主体上的 toString()（如果它存在），将主体格式化为 'text'。如果不存在，那么响应会是一个零长度的字符串。

**参数**

-   `req` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 请求对象（未使用）
-   `res` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象
-   `body` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应主体。如果它有一个 toString() 方法，它将用于进行字符串表示

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 数据

### formatJSON

JSON 格式化程序。将在主体上查找 toJson() 方法。如果不存在，那么将尝试 `JSON.stringify`。

**参数**

-   `req` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 请求对象（未使用）
-   `res` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象
-   `body` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应主体

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 数据

### formatJSONP

JSONP 格式化程序。和 JSON 一样，但包含回调调用。
使用 Unicode 转义行和段落分隔符。

**参数**

-   `req` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 请求对象（未使用）
-   `res` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象
-   `body` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应主体

返回 **[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)** 数据

### formatBinary

二进制格式化程序。

**参数**

-   `req` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 请求对象
-   `res` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应对象
-   `body` **[Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)** 响应主体

返回 **[Buffer](https://nodejs.org/api/buffer.html)** 主体
