---
title: FTP
date: 2018-04-17 00:19:14
categories: CN
tags: CN
---
File Transfer Protocol，文件传输协议，运行在 TCP 之上的应用层协议。FTP 使用两个并行的 TCP 连接，一个是控制连接（control connection），一个数据连接（data connection）。
{% asset_img ftp的两个连接.PNG "控制连接 数据连接" %}
* 控制连接
使用 21 端口。用于在两个主机之间传输控制信息，如用户标识、口令、改变远程目录命令、存放（put）/获取（get）文件命令等。
* 数据连接
使用 20 端口。用于实际发送/接收文件。

# 过程
1. Client 发起控制连接：当用户主机（Client）与远程主机（Server）开始一个 FTP 会话时，Client 首先在 Server 的 21 号端口发送本机用户标识和口令等，试图建立一个 TCP 连接，即控制连接。
2. Server 响应：Server 在控制连接上接收切换目录以及文件传输请求之后，向 Client 发起一个 TCP 连接，即数据连接，开始数据传送/接收（单个文件），直到完成文件传输，关闭该连接。如果期间再次发起文件传输请求，则再建立新的数据连接，即数据连接可能有多条，但每条只负责一个文件的传输。

## 不同的连接状态
可见，控制连接贯穿整个用户会话期间，而每次文件传输都需要建立新的数据连接，并在完成文件传输后关闭，即数据连接的生命周期和文件相关联。FTP 必须在整个会话期间保留 Client 的状态：当前目录位置等信息。

## 命令和回答
FTP 的命令和应答是人可读的，由 7 bit ASCII 格式在控制连接上传送。

**命令**
由 4 个大写字母组成，可能有可选参数，常见的有：
* USER username：向服务器传送用户标识；
* PASS password：向服务器传送用户口令；
* LIST：用于请求服务器回送当前远程目录中的所有文件列表。该文件列表经一个（新建的）数据连接传输，而不是在控制连接上传输；
* RETR filename：用于从远程主机的当前目录获取（get）文件。该命令引起远程主机发起一个数据连接，并经该连接发送所请求的文件；
* STOR filename：用于在远程主机的当前目录上存放（put）文件。由用户发起一个数据连接并发送文件。

**回答**
命令和回答一一对应。回答是 3 位数字后跟可选信息，这与 HTTP 响应报文段状态行的状态吗和状态信息的结构相同，常见的有：
* 331 Username OK，Password required；
* 125：Data connection already open，transfer starting；
* 425：Can't open data connection（无法打开数据连接）；
* 452：Error writing file（写文件错误）。