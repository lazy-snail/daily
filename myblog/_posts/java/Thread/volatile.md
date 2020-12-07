---
title: java-volatile
date: 2018-06-07 16:23:16
categories: java
tags: [java, 并发]
---
[toc]
线程安全的实现方式之一。

# volatile
见 _java 内存模型_

## synchronized vs volatile
* volatile 是线程同步的轻量级实现，所以 volatile 性能比 synchronized 要好，并且 volatile 只能用于修饰变量，而 synchronized 可以修饰方法、代码块。目前 synchronized 在执行效率上提升明显，使用频率在增大；
* 多线程访问 volatile 不会发生阻塞，而 synchronized 会发生阻塞；
* volatile 能保证数据的可见性，但不保证原子性；synchronized 可以保证原子性，也可以间接保证可见性，因为它会将私有内存和公共内存中的数据做同步；
* volatile 解决的是变量在多个线程之间的可见性，synchronized 解决的是多个线程之间访问资源的同步性。
