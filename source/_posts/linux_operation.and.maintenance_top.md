---
title: Linux运维命令 —— top
date: 2022-05-02
categories:
- linux
- operation and maintenance
tags:
- linux
- 运维
- top
- load average

description: 记录Linux运维命令 —— top的参数和用法
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283093&auto=1&height=32"></iframe></div>

`top` 用于交互式监控Linux上动态实时进程，按线程数从大到小排序。

# 1. top的输出
在shell终端输入`top`回车，就可以查看许多系统运行的实时信息。如下图所示：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/linux_command.top.default.jpg?raw=true" width=90% title="top默认输出示例" align=center/>

**<p align="center">Figure 1. top默认输出示例</p>**

top的默认输出包含两个区域：水平信息的汇总区（红框部分）和垂直信息的任务区（蓝框部分）。汇总区显示有关进程和资源使用情况的统计信息，而任务区显示当前正在运行的所有进程的列表。

下面一一说明各个参数的含义：

## 1.1. 汇总区
汇总区共有5行。

1. 第一行：任务队列信息。同`uptime`命令输出一致。

|显示|含义|
|---|---|
|top - 15:30:36|系统当前时间(system time)|
|up 29 days, 54 min|系统运行时长(uptime)|
|7 users|当前登录的用户/会话数量(user sessions)|
|load average: 4.92, 4.90, 4.96|系统平均负载，即任务队列的平均长度。三个数值分别为1min，5min，15min前到现在的平均值|

2. 第二行：任务进程信息 Tasks

|显示|含义|
|---|---|
|498 total|进程总数|
|3 running|正在运行的进程数|
|316 sleeping|睡眠状态的进程数|
|0 stopped|停止状态的进程数|
|0 zombie|僵尸状态的进程数|

3. 第三行：CPU信息 %Cpu(s)。如果有多个CPU，可能会有多行，每行单独显示一个CPU。

|显示|含义|
|---|---|
|6.1 us|用户空间占用CPU百分比|
|0.2 sy|内核空间占用CPU百分比|
|0.0 ni|用户进程空间内改变过优先级的进程占用CPU百分比|
|93.7 id|空闲CPU百分比|
|0.0 wa|等待输入输出的CPU时间百分比|
|0.0 hi|硬件CPU中断占用百分比|
|0.0 si|软中断占用百分比|
|0.0 st|虚拟机占用百分比|

4. 第四行：内存信息 KiB Mem

|显示|含义|
|---|---|
|33012627+total|物理内存总量|
|93989232 free|空闲内存总量|
|27405860 used|使用的物理内存总量|
|20873120+buff/cache|用作内核缓存的内存量|

5. 第五行：交换区信息 KiB Swap

|显示|含义|
|---|---|
|63998972 total|交换区总量|
|63280912 free|空闲交换区总量|
|718060 used|使用的交换区总量|
|30034761+avail Mem|缓冲的交换区总量。内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小,相应的内存再次被换出时可不必再对交换区写入|

## 1.2. 任务区
任务区每行代表一个进程的信息。

这里把每一列的含义列在这里：

|序号|列名|含义|
|---|---|---|
|1|PID|进程id，即标识进程的唯一的身份证号|
|2|PPID|父进程id|
|3|RUSER|真实用户名Real user name|
|4|UID|进程所有者的用户id|
|5|USER|进程所有者的用户名|
|6|GROUP|进程所有者的组名|
|7|TTY|启动进程的终端名。不是从终端启动的进程则显示为 ?|
|8|PR|优先级|
|9|NI|nice值。负值表示高优先级，正值表示低优先级|
|10|P|最后使用的CPU，仅在多CPU环境下有意义|
|11|%CPU|上次更新到现在的CPU时间占用百分比|
|12|TIME|进程使用的CPU时间总计，单位秒|
|13|TIME+|进程使用的CPU时间总计，单位1/100秒|
|14|%MEM|进程使用的物理内存百分比|
|15|VIRT|进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES|
|16|SWAP|进程使用的虚拟内存中，被换出的大小，单位kb。|
|17|RES|进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA|
|18|CODE|可执行代码占用的物理内存大小，单位kb|
|19|DATA|可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb|
|20|SHR|共享内存大小，单位kb|
|21|nFLT|页面错误次数|
|22|nDRT|最后一次写入到现在，被修改过的页面数。|
|23|S|进程状态(D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程)|
|24|COMMAND|命令名/命令行|
|25|WCHAN|若该进程在睡眠，则显示睡眠中的系统函数名|
|26|Flags|任务标志，参考 sched.h|

# 2. top的交互命令
当使用`top`命令显示汇总区和任务区后，还可以使用交互命令进行特定显示。
## 2.1. 常用交互命令
1. q：退出程序
- 要退出程序，可以按键盘上的“q”。
2. 上下箭头键/PgUp和PgDn键：任务区的翻页
3. f/F：字段/列管理
- 默认情况下任务区仅显示部分字段/列： PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND 。
- 通过 f/F 键可以显示任务区所有列的列表，被激活的字段会标记星号(*)并粗体显示。
- 字段/列的添加或删除：上下箭头键来移动到字段，空格键进行选中或去粗选中。
- q退出界面。 
4. h或者?：显示交互命令的帮助界面
5. k：终止一个进程
- 系统将提示输入需要终止的进程PID，以及需要发送给该进程什么样的信号。
- 一般的终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束该进程。默认值是信号15。在安全模式中此命令被屏蔽。 
6. u：显示特定用户的进程
- 系统将提示输入用户名(留空代表所有用户)，回车后即可显示指定用户的进程。
7. r：即renice，改变一个进程的优先级别(NI值)
- 系统提示用户输入需要改变的进程PID(默认是第一个PID进程)，然后提示输入需要改变的进程优先级值(NI值，一般修改前为0)，在已有NI值上加上这个输入值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10。
- 试了下，输入正值(优先级降低)可以生效，输入负值(优先级升高)显示Permission denied。
8. s：改变两次刷新之间的延迟时间
- 系统将提示用户输入新的时间，单位为秒(s)。如果有小数，就换算成毫秒(ms)。输入0值则系统将不断刷新，默认值是3s。
- 需要注意的是如果设置太小的时间，很可能会引起不断刷新，从而根本来不及看清显示的情况，而且系统负载也会大大增加。
9. o或者O：激活流程过滤
- 系统会提示输入过滤器表达式，回车后只显示符合表达式的进程。
- 过滤器表达式是指定属性和值之间关系的语句。示例：`COMMAND=getty: 过滤 COMMAND 属性中包含“getty”的进程`,`!COMMAND=getty: 过滤 COMMAND 属性中没有“getty”的进程`,`%CPU>3.0：过滤 CPU 使用率超过 3% 的进程`。
- 清除添加的任何过滤器，按"="。
10. W：保存当前设置到`~/.toprc`设置文件中
- 如果对top的输出做了任何更改，可以将当前设置写入`~/.toprc`文件中以供以后使用。

## 2.2. 切换显示内容
1. l：切换显示平均负载和启动时间信息(汇总区第一行)。
2. t：切换显示进程和CPU状态信息(汇总区第二、三行)，共四种视图。  
3. m：切换显示内存信息(汇总区第四、五行)，共四种视图。
4. c：切换显示命令名称和绝对路径的完整命令行。 
5. z：以红色突出显示正在运行的进程的开关。 
6. i：忽略空闲，睡眠和僵死进程的开关。
7. R：任务区默认降序排列，切换到升序排列的开关。
8. S：切换累计时间模式的开关。 
9. 1：数字1为切换 汇总区第三行CPU信息的两种显示方式，总CPUs信息或者单个CPU信息。
- 也可以用这个方式确定服务器的逻辑CPU数量。
- 但当系统的CPU数量过多(比如超过36)无法显示全部逻辑CPU信息，还没找到怎么翻页。
- 所以最好还是用`cat /proc/cpuinfo| grep "processor"| wc -l`命令查看当前系统的逻辑CPU数量。
10. H：任务区的进程显示切换为线程显示。
11. V：在正常显示和父子级层次结构显示间切换。

## 2.3. 排序任务区
1. M：根据驻留内存大小排序任务区。 
2. P：根据CPU利用率排序任务区。 
3. T：根据运行累计时间排序任务区。
4. N：根据进程ID排序任务区。 

# 3. top的常用参数
top命令的常用参数中许多都可以在交互式命令中实现。

`top`命令的常用参数：top [-] [d] [p] [q] [c] [C] [S] [s] [n]
1. -d 5：延迟时间，指定每两次屏幕信息刷新之间的时间间隔为5秒。 
2. -p 51524：通过指定进程ID(PID)来仅仅监控某个进程的状态，可用多个-p PID来指定多个进程ID。 
3. -u username：显示特定用户的进程。
4. -q：该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。 
5. -S：打开累计时间模式。
6. -s：打开安全操作模式。安全模式下用户无法更改和杀死任务，这将避免交互命令所带来的潜在危险。 
7. -i：使top不显示任何闲置或者僵死进程。 
8. -c：显示绝对路径命令行而不只是显示命令名。
9. -o %CPU：任务区按CPU使用率对进程进行降序排列显示。除了CPU使用率，还支持所有其他属性值。
10. -b：批处理模式。批处理模式程序基本上允许您将 top 命令输出结果发送到文件或其他程序。它将主要用于编写脚本和故障排除。
11. -n 5：5个循环显示后自动退出top界面。

# 4. top的应用
top常用的运维应用案例
## 4.1. 通过平均负载(load average)评估系统运行状态
1. 平均负载(load average)
- 平均负载是指系统正在执行的计算工作占系统计算资源的比例。
- 完全空闲的计算机的平均负载为 0，使用或等待 CPU 资源的每个正在运行的进程都会在平均负载上加 1。
- 单CPU情况下，0代表空载，1代表满载，超过1即超载。
- 单CPU情况下，良好的运行状况期望是平均负载在0-1之间，或不要超过1太多。
2. 查看平均负载
- `top`命令的汇总区的第一行中load average显示的三个数字即为1min，5min，15min前到现在的平均负载。这里的平均负载是所有CPU的总平均，还要根据CPU数量来计算单CPU平均负载。
3. 查看逻辑CPU数量(processor数量)
- `cat /proc/cpuinfo| grep "processor"| wc -l`可获得当前系统的逻辑CPU数量
- `lscpu`列出的CPU(s)信息也是逻辑CPU数量
4. 计算单CPU平均负载，评估系统运行现状
- 假设`top`命令中平均负载值为54，系统CPU数量为36，单个CPU的平均负载为1.5(=54/24)，代表系统过载率为50%(=54/36-1)。
- 假设`top`命令中平均负载值为18，系统CPU数量为36，单个CPU的平均负载为0.5(=18/24)，代表系统空闲率为50%(=1-18/36)。

即达到，可根据`top`命令的平均负载和CPU数量快速判断系统运行状态。

# 5. references
1. top的manual：https://man7.org/linux/man-pages/man1/top.1.html
2. top命令指南：https://www.booleanworld.com/guide-linux-top-command/
3. 博客：top输出详解：https://www.jianshu.com/p/af584c5a79f2
4. 博客：平均负载：https://www.howtogeek.com/194642/understanding-the-load-average-on-linux-and-other-unix-like-systems/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>