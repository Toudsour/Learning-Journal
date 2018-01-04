##实战iptable
###iptable原理
iptable是通过一条条规则来生效的，iptable将规则分别挂在在5个表上，这五个表为:

```
1. Filter(表)
filter表是专门过滤包的，内建三个链，可以毫无问题地对包进行DROP、LOG、ACCEPT和REJECT等操作 
    1) INPUT(链)
    INPUT针对那些目的地是本地的包
        1.1) 规则rule
　　　　    ..
    2) FORWARD(链)
    FORWARD链过滤所有不是本地产生的并且目的地不是本地(即本机只是负责转发)的包
        2.1) 规则rule
　　　　    ..
    3) OUTPUT(链)
    OUTPUT是用来过滤所有本地生成的包
        3.1) 规则rule
　　　　    ..
2. Nat(表)
Nat表的主要用处是网络地址转换，即Network Address Translation，缩写为NAT。做过NAT操作的数据包的地址就被改变了，当然这种改变是根据我们的规则进行的。属于一个流的包(因为包
的大小限制导致数据可能会被分成多个数据包)只会经过这个表一次。如果第一个包被允许做NAT或Masqueraded，那么余下的包都会自动地被做相同的操作。也就是说，余下的包不会再通过这个表
，一个一个的被NAT，而是自动地完成
    1) PREROUTING(链)
    PREROUTING 链的作用是在包刚刚到达防火墙时改变它的目的地址
        1.1) 规则rule
　　　　    ..
    2) INPUT(链)
        2.1) 规则rule
　　　　    ..
    3) OUTPUT(链)
    OUTPUT链改变本地产生的包的目的地址
        3.1) 规则rule
　　　　    ..
    4) POSTROUTING(链)
    POSTROUTING链在包就要离开防火墙之前改变其源地址。
        4.1) 规则rule
　　　　    ..
3. Mangle(表) 
这个表主要用来mangle数据包。我们可以改变不同的包及包 头的内容，比如 TTL，TOS或MARK。 注意MARK并没有真正地改动数据包，它只是在内核空间为包设了一个标记。防火墙内的其他的规
则或程序(如tc)可以使用这种标记对包进行过滤或高级路由。注意，mangle表不能做任何NAT，它只是改变数据包的TTL，TOS或MARK，而不是其源目地址。NAT必须在nat表中操作的。
    1) PREROUTING(链)
    PREROUTING在包进入防火墙之后、路由判断之前改变 包
        1.1) 规则rule
　　　　    ..
    2) INPUT(链)
    INPUT在包被路由到本地之后，但在用户空间的程序看到它之前改变包
        2.1) 规则rule
　　　　    ..
    3) FORWARD(链)
    FORWARD在最初的路由判断之后、最后一次更改包的目的之前mangle包
        3.1) 规则rule
　　　　    ..
    4) OUTPUT(链)
    OUTPUT在确定包的目的之前更改数据包
        4.1) 规则rule
　　　　    ..
    5) POSTROUTING(链)
    POSTROUTING是在所有路由判断之后
        5.1) 规则rule
　　　　    ..
```

常用的两个表就是filter表（用来进行数据包过滤）和NAT表（NAT转换规则以及作用于连接跟踪）。

iptable 底层是通过netfilter来实现的，之前所说的几个表其实都是挂载在5个HOOK点,详细罗列如下:


图像如下:
![](http://img.blog.csdn.net/20150404145342019?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamFzb25jaGVuX2diZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
更直接的次序图如下:
![](http://images.cnitblog.com/i/532548/201405/050914109325291.png)

###iptable用法
[详细用法](https://www.ibm.com/developerworks/cn/linux/network/s-netip/index.html)