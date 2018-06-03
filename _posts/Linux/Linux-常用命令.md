---
title: Linux-常用命令
date: 2018-04-15 15:58:31
categories:
tags: [Linux, Shell]
---
**查看内存使用情况**
1. free 命令：man 解释：Display amount of free and used memory in the system。
free -m：显示内存情况：已用、未用、总量等信息。
ps：free 命令读取的信息就来源于 /proc/meminfo。其中的 buffers 列为块设备 inode 管理的所有 page cache 数量；cached 为内核总缓存-buffers-交换缓存总和。
2. /proc/meminfo
cat /proc/meminfo：显示内存的详细情况，包括以上信息。
3. top
top [-u 用户名]：当前用户进程占用资源情况，包括 进程所有者、NI（nice值）、CPU、内存、S（进程状态：S：休眠，R：正在运行，Z：僵死，N：进程优先级为负）等信息。
默认以 CPU 使用率排序，输入大写 M 可以切换成以内存使用率排序，大写 P 以 CPU 占用率排序。
4. pmap 命令：man 解释：report memory map of a process。
pmap -d 进程号：根据进程查看该进程相关信息占用的内存情况。


**查看电池状态**
upower -i `upower -e | grep 'BAT'`