---
title: MQ
date: 2018-06-14 14:59:41
categories: backend
tags: [backend]
---
[toc]
# MQ
Message Queue，消息队列：是分布式系统中重要组件，主要用于解决 **应用耦合、异步消息、流量削峰** 等问题。生产环境中如抢购、秒杀等需要控制并发量的场景中都需要用到 MQ。
常用 MQ 有：RabbitMQ（Rabbit）、RocketMQ（Ali -> Apache）、ActiveMQ（Apache）、Kafka（Linkedin -> Apache）、MetaMQ 等，部分数据库如 Redis、MySQL 等也可以实现消息队列功能。


# 应用场景

## 应用耦合
多应用之间通过 MQ 对同一消息进行处理，避免调用接口失败导致整个过程失败。如，用户使用 QQ 相册上传图片，人脸识别系统会对图片进行人脸识别，一般做法是：服务器收到图片后，图片上传系统立即调用人脸识别系统，调用完成后返回结果：
https://cloud.tencent.com/developer/article/1006035

## 异步处理
多应用对 MQ 中同一消息进行处理，应用间并发处理消息，相比于串行处理，能够减少处理时间。如，

## 限流削峰
如，购物网站经常举行秒杀活动，会导致瞬时访问量过大，流量激增，可能导致服务器宕机。加入 MQ 后，服务器以可接受的数量从 MQ 中拉取数据进行处理，相当于 MQ 作为中间缓冲。

## 消息驱动的系统


# 两种模式
## p2p
点对点模式包括 3 个据角色：
* 消息队列；
* 发送者（生产者）；
* 接收者（消费者）
{% asset_img p2p模式.png p2p 模式 %}
消息发送者生产消息发送到 queue 中，然后消息接收者从 queue 中取出并消费消息。特点：
* 每个消息只有一个接收者，即一旦被消费，该消息就从 MQ 中取出；
* 发送者和接收者之间没有依赖性，发送者发送消息之后，不管有没有接收者在运行，都不会影响到发送者下次发送消息；
* 接收者在成功接收到消息之后需要向队列应答，以便 MQ 删除当前接收的消息。

## 发布/订阅
发布/订阅模式包括 3 个据角色：
* 角色主题（Topic）
* 发布者（Publisher）
* 订阅者（Subscriber）
{% asset_img 发布-订阅模式.png 发布/订阅模式 %}
发布者将消息发送到 Topic，系统将这些消息传递给多个订阅者。特点：
* 每个消息可以有多个订阅者；
* 发布者和订阅者之间有时间上的依赖性：针对某个 Topic 的订阅者，必须创建一个订阅者之后，才能消费发布者的消息；
* 为了消费消息，订阅者需要提前订阅该 Topic，并保持在线运行。

# 常用 MQ 简介
## RabbitMQ
基于 AMQP（高级消息队列协议）完成的、可复用的企业消息系统。

https://www.zhihu.com/question/43557507
https://www.jianshu.com/p/689ce4205021