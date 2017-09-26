#printk使用简介
##printk概述

在调试过程我们少不了打log，在c语言中我们通常用printf,这是glibc为我们实现的，那么我们再内核中如何输出log呢，那就是printf的兄弟printk了。

printk有着广泛的兼容性，几乎可以在内核的任何地方使用，可以使用在中断，进程等等，并且无需考虑加锁，即使终端没有完成初始化，printk也会将内容输出到自带的ringbuffer中，待初始化后一并输出。

当然printk也有力所不逮的时候，那就是系统最开始启动时，当然还是有解决的方法，我们可以调用early\_printk进行输出。

##printk使用

printk的使用相当简单，以下是printk的一个简单例子

```c
printk(KERN_INFO "HELLO KERNEL\n");
```

除了前面加上了`KERN_INFO`，除此之外并无不同，事实也是如此，而`KERN_INFO`的作用就是表明日志等级，Linux中日志等级总共有如下几个:

```c
#define KERN_EMERG "<0>" /* 系统不可使用 */
#define KERN_ALERT "<1>" /* 需要立即采取行动 */
#define KERN_CRIT "<2>" /* 严重情况 */
#define KERN_ERR "<3>" /* 错误情况 */
#define KERN_WARNING "<4>" /* 警告情况 */
#define KERN_NOTICE "<5>" /* 正常情况, 但是值得注意 */
#define KERN_INFO "<6>" /* 信息型消息 */
#define KERN_DEBUG "<7>" /* 调试级别的信息 */

/* 使用默认内核日志级别 */
#define KERN_DEFAULT "<d>"
/*
* 标注为一个“连续”的日志打印输出行(只能用于一个
* 没有用 \n封闭的行之后). 只能用于启动初期的 core/arch 代码
* (否则续行是非SMP的安全).
*/
#define KERN_CONT "<c>
```
如何没有定义的话就使用`KERN_DEFAULT`,系统默认为`KERN_WARNING`。

##查看日志
首先要注意的是，printk和printf一样只有遇到`\n`才会输出，所以输出的时候一定要注意。

可以使用dmesg查看所有等级的printk的输出，当高于终端设定等级的printk会直接打印到终端上。
对于终端的输出等级可以在`/proc/sys/kernel/printk`中设置，分别对应
- 当前控制台日志级别
- 未明确指定日志级别的默认消息日志级别
- 最小（最高）允许设置的控制台日志级别
- 引导时默认的日志级别

除此之外`/proc/kmsg`和`/var/log/messages.log`也可以看到。
##printk实现

##参考
[](http://blog.csdn.net/chinacodec/article/details/3913154)
[](http://blog.chinaunix.net/uid-20543672-id-3211832.html)
[](https://www.ibm.com/developerworks/cn/linux/l-kernel-logging-apis/index.html)
