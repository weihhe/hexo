title: HTTP 长连接
author: weihehe
tags:
  - 协议
categories:
  - 网络
date: 2024-08-18 17:59:00
---
应用层协议的一种
<!--more-->

## 概念

- HTTP本身是一个Web服务器，可以直接访问到。
 
- https://weihehe.top/ —— 域名。

	- 为什么我们没有输入端口号，因为https默认自带了端口。 
	- `https`——443端口
	- `http`——80端口

`url`——统一资源定位符，可以定位全网的一个资源。

![URL](/images/HTTP-URL.png)

### urlencode和urldecode

- 像 / ? : 等这样的字符, 已经被url当做特殊意义理解了. 因此这些字符不能随意出现。

- 为了避免冲突或歧义，这些字符需要被编码成一种特殊的格式。这种编码通常称为 **URL 编码** 或 **百分号编码**（Percent-encoding）。编码是通过将特殊字符转换为 `%` 后跟两个十六进制数来表示该字符的 ASCII 值。例如，空格字符在 URL 编码中表示为 `%20`。

- urldecode就是urlencode的逆过程。


## HTTP 请求：

### 1. 请求行 

请求行包括以下三个部分，并用**若干空格**分隔：

- **请求方法**：表示要对资源执行的操作，例如 `GET`, `POST`, `PUT`, `DELETE` 等。
- **请求目标（URL）**：请求的资源路径，通常为绝对路径。例如，`/index.html`。
- **HTTP 版本**：指定使用的 HTTP 协议版本，如 `HTTP/1.1` 或 `HTTP/2.0`。

**示例**:

```
GET /index.html HTTP/1.1
```

### 2. 请求报头 

请求头是若干个**键值对**，每个键值对之间用冒号 `:` 分隔，用于传递客户端和服务器之间的元数据信息（Key : Value）。常见的请求头包括：

- **Host**：表示请求资源的主机名和端口号。
- **User-Agent**：标识发出请求的客户端类型。
- **Accept**：表示客户端可接受的响应内容类型。
- **Content-Type**：在请求包含数据时，表示请求体的数据格式。
- **Authorization**：用于传递身份验证信息。
- **Content-length**请求正文长度，方便我们进行正确的读取。

### 3. 空行

空行用于分隔**请求报头和请求正文**。请求头结束后必须有一个空行，即一个回车符（`\r\n`），表示请求头部分的结束。

### 4. 请求正文(请求体) 

请求体用于传递实际的数据，这部分可选，通常在 `POST`, `PUT` 等请求方法中使用。请求体包含了客户端要发送到服务器的具体数据，例如表单数据、文件内容等。

### **示例 HTTP 请求**：

```
POST /login HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

username=example&password=12345
```

## HTTP 响应：

### 1. 状态行 

状态行包含以下三个部分，按顺序排列，并用空格分隔：

- **HTTP 版本**：指明服务器使用的 HTTP 协议版本，如 `HTTP/1.1` 或 `HTTP/2.0`。
- **状态码**：一个三位数的数字，表示请求结果的状态，如 `200` 表示成功，`404` 表示资源未找到，`500` 表示服务器内部错误。
- **状态描述**：状态码的文本描述可以用于报错，如 `OK`, `Not Found`, `Internal Server Error` 等。

**示例**:

```
HTTP/1.1 200 OK
```

### 2. 响应报头 

响应头是若干个**键值对**，每个键值对之间用冒号 `:` 分隔，用于传递服务器和客户端之间的元数据信息（类似request）。常见的响应头包括：

- **Content-Type**：表示响应内容的 MIME 类型，例如 `text/html`, `application/json`。
- **Content-Length**：表示响应体的长度（以字节为单位）。
- **Server**：指明服务器的软件信息，例如 `Apache/2.4.1`。
- **Date**：表示响应生成的日期和时间。
- **Set-Cookie**：设置客户端 Cookie 信息。

**示例**:

```
Content-Type: text/html; charset=UTF-8
Content-Length: 138
Server: Apache/2.4.1
Date: Sat, 17 Aug 2024 12:34:56 GMT
```

### 3. 空行

响应头结束后有一个空行（即一对回车换行符 `\r\n`），表示响应头部分的结束。

### 4. 响应正文 

响应体包含了服务器返回给客户端的实际数据，

```html
<html>
<head>
<title>Example</title>
</head>
<body>
<h1>Hello, World!</h1>
</body>
</html>
```

**示例 HTTP 响应**：

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 138
Server: Apache/2.4.1
Date: Sat, 17 Aug 2024 12:34:56 GMT

<html>
<head>
<title>Example</title>
</head>
<body>
<h1>Hello, World!</h1>
</body>
</html>
```
## 连接方式(可以通过Connection选择来选择)

### 短连接(Connection: close
)

- **定义**：短连接是指在每次 HTTP 请求和响应完成后，客户端与服务器之间的 TCP 连接立即关闭。

- **特点**：每次请求都要重新建立一个新的连接，完成后再关闭。这种方式在 HTTP/1.0 中默认使用。

### 长连接(Connection: keep-alive
)

- **定义**：长连接是指在一次 HTTP 请求响应完成后，TCP 连接不会立即关闭，而是保持打开状态，以便后续的请求复用同一个连接。

- **特点**：客户端和服务器可以在同一个连接上进行多次请求和响应，减少了连接的建立和关闭次数。在 HTTP/1.1 中，长连接是默认的连接方式。

## 会话保持功能

- 指的是在一个客户端和服务器之间维持一个持续的会话状态，以确保在多个请求中，服务器能够识别并保持与同一客户端的连接或交互状态。

- 由于 HTTP 协议本质上是无状态的，每次请求都是独立的，服务器不会自动记住之前的请求状态，因此需要通过某种机制来实现会话保持。
	- Cookies：
		- 服务器通过 HTTP 响应中的 `Set-Cookie` 头字段向客户端发送一个唯一标识符，客户端会在后续的请求中通过 `Cookie` 头字段将这个标识符发回服务器。
		- 服务器可以根据这个标识符来关联与客户端的会话状态，例如用户登录信息、购物车内容等。
		- 其中保存`cookie`的方式又能被区分为"内存级"和“磁盘级”。
