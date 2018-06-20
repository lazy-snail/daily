---
title: OS
date: 2018-04-10 23:20:06
categories: OS
tags: [OS]
---
[toc]
# 中断/异常
操作系统是由中断驱动的，也可以说是由事件驱动的。
中断/异常 是 CPU 对系统发生的某个事件作出的一种反应。事件的发生改变了 CPU 的控制流：CPU 暂停正在执行的程序，保留现场后 **自动** 转去执行相应的事件处理程序，处理完成后返回断点继续执行被打断的程序。这里的“自动”是说，该操作是由硬件来完成的，硬件完成控制流的转移工作。

## 中断（外中断）
外部事件，正在运行的程序所不期望的，是异步的。其引入是为了支持 CPU 和设备之间的并行操作：当 CPU 启动设备进行 I/O 后，设备便可以独立工作，CPU 就可以转去处理与此次 I/O 无关的事情，当设备完成 I/O 后，通过向 CPU 发送中断报告此次 I/O 的结果，让 CPU 决定如何处理后续操作。
包括：
* I/O 中断
* 时钟中断
* 硬件故障

##异常（内中断）
由正在执行的指令引发，是同步的。表示 CPU 执行指令时本身出现的问题：算术溢出、内存访问越界、除零等。这时硬件改变了 CPU 当前的执行流程，转到相应的错误处理程序或异常处理程序或执行系统调用。
包括：
* 系统调用
* 页故障/页错误
* 保护性异常
* 断点指令
* 其他程序性异常（如算术溢出）

## 硬件/软件分工
### 硬件：响应
硬件负责捕获中断源发出的中断/异常请求，以一定方式响应，将处理器控制权转交给特定的处理程序。由中断硬件部件完成。
{% asset_img 中断响应过程.PNG 中断响应过程 %}

### 软件：处理
识别中断/异常类型并完成相应的处理。系统运行时若响应中断，中断硬件部件将 CPU 控制权转交给中断处理程序，处理程序执行：
* 保存相关寄存器信息；
* 分析中断/异常的具体原因；
* 执行对应的处理功能；
* 恢复现场，返回被事件打断的程序。
