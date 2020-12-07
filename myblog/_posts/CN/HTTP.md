---
title: HTTP
date: 2018-04-17 00:19:14
categories: CN
tags: CN
---
HyperText Transfer Protocol，超文本传输协议，它是 web 的核心。HTTP 协议是无状态的，即对于事务处理没有记忆能力。

# HTTP 请求报文格式
一个 HTTP 请求报文由请求行(request line)、首部行（header line）、空行、请求数据四部分组成。
{% asset_img http请求.PNG HTTP 请求 %}

## 请求行
3个字段：方法字段、URL 字段、协议版本字段。

### 方法字段
包括：GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT

#### GET
绝大多数请求报文使用 GET 方法：从服务器读取文档、点击网页链接/键入网页地址浏览网页等。该方法要求服务器将 URL 定位的资源放在响应报文的数据部分，回送给客户端。此时请求数据为空，而请求参数和对应的值附加在 URL 后面，利用‘？’代表 URL 的结尾与请求参数的开始，并且参数长度受限，一般 1024 个字符。不适合传送私密数据（要放在 URL 后面），也不适合大量数据传送的场景。

#### POST
对于以上不适合 GET 的情况，可以考虑 POST 方式（即 POST 可以完成 GET 请求）。可以允许客户端给服务器提供更多的信息。将请求参数封装在 **请求数据** 中，以 “名称/值”的形式，没有大小限制，也不会显示在 URL 中。适用于页面表单中（但 GET 同样能用于表单）。二者各有优势，分情况使用。

#### GET vs POST
* GET 在浏览器回退时是无害的，而 POST 会再次提交请求。
* GET 产生的 URL 地址可以被 Bookmark，而 POST 不可以。
* GET 请求会被浏览器主动 cache，而 POST 不会，除非手动设置。
* GET 请求只能进行 url 编码，而 POST 支持多种编码方式。
* GET 请求参数会被完整保留在浏览器历史记录里，而 POST 中的参数不会被保留。
* GET 请求在 URL 中传送的参数是有长度限制的，而 POST 没有限制。
* 对参数的数据类型，GET 只接受 ASCII 字符，而 POST 没有限制。
* GET 比 POST 更不安全，因为参数直接暴露在 URL 上，所以不能用来传递敏感信息。
* GET 参数通过 URL 传递，POST 放在 Request body 中。

[参考: GET 与 POST 区别](https://mp.weixin.qq.com/s?__biz=MzI3NzIzMzg3Mw==&mid=100000054&idx=1&sn=71f6c214f3833d9ca20b9f7dcd9d33e4)

#### HEAD
类似 GET，只不过服务器接收到 HEAD 请求后只返回响应头，而不发送相应内容。因此当我们只需要查看某个页面的状态的时候，使用 HEAD 是非常高效的。

#### PUT
向服务器放置资源；

#### DELETE
请求服务器删除资源；

### URL 字段
带有请求对象的标识。

### 协议版本字段
自解释的，一般为 HTTP/1.1.

## 首部行
通知服务器有关客户端请求的信息，用“关键字: 值”对组成，每行一对。典型的请求首部有：
* User-Agent：产生请求的浏览器类型；
* Accept：客户端可识别的内容类型列表；
* Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机。
* Connection:Keep-Alive/Close：长连接/短连接标识

## 空行
最后一个首部行之后的一行，发送回车符和换行符，通知服务器不再有请求头。

## 请求数据
请求数据不在 GET 方法中使用，而在 POST 方法中使用。POST 方法适用于需要客户填写表单的场合。

# HTTP 响应报文
一个 HTTP 响应报文由三部分组成：状态行(status line)、首部行(header line)、响应数据(response body)。
{% asset_img http响应.PNG HTTP 响应 %}

## 状态行
3个字段：协议版本字段、状态码、响应状态信息。

### 协议版本字段
一般为 HTTP/1.1；

### 状态码
（2xx、4xx 等）。

### 响应状态信息
跟随在状态码后的状态信息：

#### 1xx
指示信息，请求已接收，继续处理。

#### 2xx
成功，请求已被成功接收、处理、接受。
* 200 OK：请求成功；

#### 3xx
重定向，要完成请求必须进行更进一步的操作。
* 301 Moved Permanently：请求的对象已经被永久转移，新的 URL 在响应报文的 Location 首部中。客户端将自动获取新的 URL。
* 303 See Other：重定向到其他页面，见 Location
* 304 Not Modified：告诉客户端自从上次请求取得后，该资源未修改，可使用本地缓存

#### 4xx
客户端错误，请求有语法错误或请求无法实现。
* 400 Bad Request：客户端请求语法有错误，不能被服务器所理解
* 401 Unauthorized：请求未经授权
* 403 Forbidden：服务器收到请求，但是拒绝提供服务
* 404 Not Found：请求资源不存在

#### 5xx
服务器端错误，未能实现该合法请求。
* 500 Internal Server Error：服务器发生不可预期的错误
* 503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复
* 505 HTTP Version Not Supported：服务器不支持请求报文使用的 HTTP 协议版本

## 首部行
常见的有：
Connection：close   告诉客户端发完报文后将关闭该 TCP 连接；
Date：xxx   服务器发送该报文的时间；
Server：xxx   服务器类型，如 Apache Web；
Last-Modified：xxx   缓存相关；
Content-Length：xxx   被发送对象的字节数；
Content-Type：xxx   响应正文中对象的类型；
Cache-Control：xxx   告诉客户端如何控制响应内容的缓存；
Location：xxx   重定向请求资源；
Set-Cookie：xxx   设置客户端的 cookie；

## 响应数据
即被请求的数据。

# 长连接 短连接
HTTP 协议的长连接和短连接，实质上是 TCP 协议的长连接和短连接。
HTTP/1.0 中，默认使用的是短连接，即，浏览器和服务器每进行一次 HTTP 操作，就建立一次 TCP 连接，任务结束就断开连接。如果客户端浏览器访问的某个 HTML 或其他类型的 Web 页中包含有其他的 Web 资源，如 JavaScript 文件、CSS 文件、图像文件等，每当浏览器遇到这样一个 Web 资源，就会建立一个 HTTP 会话。
到了 HTTP/1.1，默认使用长连接，即请求报文首部行中“Connection:Keep-Alive”字段，用以保持连接特性。使用长连接情况下，当打开一个网页后，客户端和服务器之间用于传输 HTTP 数据的 TCP 连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这个已经建立的连接。Keep-Alive 不会永久保持连接，而是有一个保持时间，可以在不同的服务器软件（如 Apache）中设置该变量，要求双方都支持长连接字段。

## 对比
显然，长连接可以省去较多的 TCP 建立/关闭的操作，减少浪费，节约时间。**对于频繁请求资源的场景，如直播、流媒体，较适用长连接**。短连接对于服务器来说管理较为简单，**适用于网页浏览等数据刷新频度较低的场景**，存在的连接都是有用的连接，不需要额外的控制手段。但如果客户请求频繁，将在 TCP 的建立和关闭操作上浪费时间和带宽。

# HTTPS
Hypertext Transfer Protocol over Secure Socket Layer，以安全为目标的 HTTP 协议通道，简单讲是 HTTP 的安全版。即 HTTP 下加入 SSL 层。所以，HTTPS 的安全基础是 SSL，因此加密的详细内容就需要 SSL。


https://www.jianshu.com/p/30b8b40a671c
https://www.cnblogs.com/zxj015/p/6530766.html