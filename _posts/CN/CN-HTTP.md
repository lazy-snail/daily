---
title: CN-HTTP
date: 2018-04-17 00:19:14
tags: CN
---
#### HTTP 请求报文
一个 HTTP 请求报文由请求行(request line)、首部行（header line）、空行、请求数据四部分组成。
{% asset_img http请求.PNG HTTP 请求 %}
**请求行**：3个字段
* 方法字段（GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT）
> * GET：绝大多数请求报文使用 GET 方法：从服务器读取文档、点击网页链接/键入网页地址浏览网页等。该方法要求服务器将 URL 定位的资源放在响应报文的数据部分，回送给客户端。此时请求数据为空，而请求参数和对应的值附加在 URL 后面，利用‘？’代表 URL 的结尾与请求参数的开始，并且参数长度受限，一般 1024 个字符。不适合传送私密数据（要放在 URL 后面），也不适合大量数据传送的场景。
> * POST：对于以上不适合 GET 的情况，可以考虑 POST 方式（即 POST 可以完成 GET 请求）。可以允许客户端给服务器提供更多的信息。将请求参数封装在 **请求数据** 中，以 “名称/值”的形式，没有大小限制，也不会显示在 URL 中。适用于页面表单中（但 GET 同样能用于表单）。二者各有优势，分情况使用。
两者的进一步对比：https://mp.weixin.qq.com/s?__biz=MzI3NzIzMzg3Mw==&mid=100000054&idx=1&sn=71f6c214f3833d9ca20b9f7dcd9d33e4
> * HEAD：类似 GET，只不过服务器接收到 HEAD 请求后只返回响应头，而不发送相应内容。因此当我们只需要查看某个页面的状态的时候，使用 HEAD 是非常高效的。
> * PUT：向服务器放置资源；
> * DELETE：请求服务器删除资源；
> * _其他略_

* URL 字段：带有请求对象的标识。
* 协议版本字段：自解释的，一般为 HTTP/1.1.

**首部行**：
通知服务器有关客户端请求的信息，用“关键字: 值”对组成，每行一对。典型的请求首部有：
* User-Agent：产生请求的浏览器类型；
* Accept：客户端可识别的内容类型列表；
* Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机。

**空行**
最后一个首部行之后的一行，发送回车符和换行符，通知服务器不再有请求头。

**请求数据**
请求数据不在 GET 方法中使用，而在 POST 方法中使用。POST 方法适用于需要客户填写表单的场合。

#### HTTP 响应报文
一个 HTTP 响应报文由三部分组成：状态行(status line)、首部行(header line)、响应数据(response body)。
{% asset_img http响应.PNG HTTP 响应 %}
**状态行**：3个字段
* 协议版本字段，一般为 HTTP/1.1；
* 状态码（2xx、4xx 等），
* 响应状态信息，跟随在状态码后的状态信息；
> * 1xx：指示信息，请求已接收，继续处理；
> * 2xx：成功，请求已被成功接收、处理、接受；
　　200 OK：请求成功
> * 3xx：重定向，要完成请求必须进行更进一步的操作；
　　301 Moved Permanently：请求的对象已经被永久转移，新的 URL 在响应报文的 Location 首部中。客户端将自动获取新的 URL。
　　303 See Other：重定向到其他页面，见 Location
　　304 Not Modified：告诉客户端自从上次请求取得后，该资源未修改，可使用本地缓存
> * 4xx：客户端错误，请求有语法错误或请求无法实现；
　　400 Bad Request：客户端请求语法有错误，不能被服务器所理解
　　401 Unauthorized：请求未经授权
　　403 Forbidden：服务器收到请求，但是拒绝提供服务
　　404 Not Found：请求资源不存在
> * 5xx：服务器端错误，未能实现该合法请求。
　　500 Internal Server Error：服务器发生不可预期的错误
　　503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复
　　505 HTTP Version Not Supported：服务器不支持请求报文使用的 HTTP 协议版本

**首部行**：常见的有：
Connection：close   告诉客户端发完报文后将关闭该 TCP 连接；
Date：xxx   服务器发送该报文的时间；
Server：xxx   服务器类型，如 Apache Web；
Last-Modified：xxx   缓存相关；
Content-Length：xxx   被发送对象的字节数；
Content-Type：xxx   响应正文中对象的类型；
Cache-Control：xxx   告诉客户端如何控制响应内容的缓存；
Location：xxx   重定向请求资源；
Set-Cookie：xxx   设置客户端的 cookie；

**响应数据**：
即被请求的数据。
