---
title: gRPC
date: 2020-06-02 20:51:50
categories: middleware
tags: [rpc, middleware]
---
[toc]


# rpc
以java平台为例，客户端和服务端共用一个接口(将接口打成一个jar包，在客户端和服务端分别引入这个jar包)，客户端面向接口写调用，服务端面向接口写实现，中间的网络通信交给框架去实现。