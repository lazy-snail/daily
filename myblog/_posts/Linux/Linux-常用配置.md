---
title: Linux-常用配置
date: 2018-04-15 13:56:30
tags: [Linux, Shell]
---
**设置服务开机自启动**
基础知识：系统启动过程中，内核被加载后，执行的第一个程序是 /sbin/init，init 程序会读取 inittab 文件的内容，从而确定系统的运行级（0-6，即 rc0.d - rc6.d。ps：rc 一般指：run command）。确定运行级后执行 /etc/rc.d/rc.sysinit，对系统进行一些初始化。之后启动内核模块，启动内核模块之后执行相应的运行级文件（rc0.d - rc6.d），然后执行 /etc/rc.d/rc.local，最后执行 /bin/login 进入登陆状态。
从上述内容可知，在系统启动过程中有两种任务启动方法：
1. 将启动脚本放在 /etc/rc.d/init.d 文件中，并建立到 rc0.d - rc6.d 的链接。
_init.d 中存放着一些系统启动时要运行的服务的脚本，但并不是每个脚本都会被执行。Linux 把 init.d 中的服务链接到运行级 rc0.d - rc.6.d 中，在确定系统的运行机制后执行相应运行级的 rc?.d。这是推荐使用的配置方式。_
2. 将启动脚本放在 /etc/rc/local 中，这是在其他的初始化脚本执行完后才执行的，用户可以在此进行个性化操作，配置需要启动的服务等。

而在用户成功登陆后，也可以进行自启动任务的配置。分别在相关的文件中：
3. /etc/profile：全局环境设置，对系统中每个用户都有效。每个用户登陆后都会立即执行该脚本，因此，任何用户登陆后都需要执行的任务就放在这里。
4. /etc/.bashrc：对所有用户有效，每次打开 shell 时会执行该脚本，保存的是系统 bash shell 的信息。
5. ~/.bash_profile：当前用户有效，登陆后执行该脚本。
6. ~/.bashrc：当前用户有效，每次打开 shell 时执行该脚本。

/etc/profile 和 ~/.bash_profile，/etc/bashrc 和 ~/.bashrc 的区别可以理解成全局变量（影响所有用户）和局部变量（影响当前用户）。而 \*profile 和 \*bashrc 的区别是，前者时交互式、login 方式进入 bash 运行的，后者是非交互式 non-login 方式进入 bash 运行的。
_交互模式：shell 等待用户输入，并且执行提交的命令。即：登陆、执行命令、退出。非交互模式：shell 不与用户进行交互，而是读取存放在文件中的命令并执行，当执行到文件结尾，shell 也就终止了。_

**定时任务**
参考链接：[Linux 下添加定时任务](https://blog.csdn.net/hi_kevin/article/details/8983746 "Linux 下添加定时任务")
周期执行的任务一般由守护进程 cron 来处理。它读取一个/多个配置文件，文件中包含了命令行及其调用时间。配置文件名为 crontab（cron table 的缩写）。
cron 在3个地方查找配置文件：
1. /var/spool/cron：包括 root 的所有用户的 crontab 任务，以创建者的名字命名，比如 neil 创建的 crontab 对应的就是 /var/spool/cron/neil。一般一个用户最多只有一个 crontab 文件。
2. /etc/crontab：负责安排由系统管理员制定的维护系统以及其他任务的 crontab。
3. /etc/cron.d：存放任何要执行的 crontab 文件/脚本。

权限：管理在 /var/adm/cron 下，文件 cron.allow 和 cron.deny 用法：
1. 如果两个文件都不存在，则只有 root 才能使用 crontab 命令。
2. 如果 cron.allow 存在而 cron.deny 不存在，则只有列在 cron.allow 中的用户才能使用 crontab 命令，如果 root 用户不在里面，则 root 用户也不能使用。
3. 如果 cron.allow 不存在而 cron.deny 存在，则只有列在 cron.deny 中的用户不能使用 crontab 命令，其他用户均可用。
4. 如果两个文件都存在，则列在 cron.allow 中的可以使用 crontab 命令，列在 crontab.deny 的不能使用，如果某用户同时出现在两个文件中，以 cron.allow 为准，可以使用。

crontab 文件：
七个域：minute  hour  day-of-month  month  day-of-week  user  commands
合法值：00-59    00-23    01-31          01-12   0-6                 neil   脚本命令
特殊值：“\*”：代表所有取值范围内的数字，"/"：代表“每”的意思（“/5”表示每5个单位），"-"：代表范围，从某个数字到某个数字，","：分开几个离散的数字。
_cron 一词来源于希腊语的 “chronos”，意为时间，用来管理现行时间顺序。_

**查看系统信息**
cat /etc/issue　　查看发行版本号
cat /etc/lsb_release　 内核版本等
lsb_release -a　　详细的系统信息(lsb：Linux Standard Base)
uname -r　　查看内核版本号
uname -a      详细信息
cat /proc/version      内核版本、编译内核的 gcc 版本、编译时间、编译者的用户名等
dmesg | grep "Linux"：打印内核信息（gmesg：display message or driver message）
