#Scapy发送自定义option的TCP报文
##scapy简介
scapy是一个使用python编写的网络工具，scapy根本上来说干三件事:构造,侦听和解析报文，这三者相结合基本可以满足网络中大部分我们所需要的功能。
更为详细可以查看scapy[官网的介绍](http://scapy.readthedocs.io/en/latest/introduction.html),如果英文阅读较为困难，这里[pdcxs007的译文](http://blog.csdn.net/pdcxs007/article/details/46686843)
##scapy使用
scapy的使用可以参看其官网上的[Usage](http://scapy.readthedocs.io/en/latest/usage.html),这里也给出一个[译文](https://wenku.baidu.com/view/e2e5f41acd7931b765ce0508763231126edb77f9.html)。

如果对python较为熟悉可以直接参照发送自定义的option的过程学习使用scapy的用法，发送自定义包的过程主要分为三步:

- 构造SYN包，并发送接受SYN_ACK
- 根据SYN_ACK构造ACK，并发送
- 构造RST包，并发送

以下是详细介绍:

###第一步--发送SYN包
在scapy中构造一个包是非常容易的，你只需要提供哪些你需要的关注的值就好了，不需要的scapy会自动帮你填充的

```py
	#构造一个syn包，`/`用于组合不同的协议层
    syn = IP(src = src_ip, dst = dst_ip)/TCP(dport = dst_port, sport = src_port, flags = "S")
   #发送syn包，并接受响应并保存为syn_ack
	syn_ack = sr1(syn)
```
第一步就完成了，是不是非常的简单？

###第二步--构造ACK，完成握手
下边我们读取SYN_ACK的ack和seq，填入到ACK包中，并加入我们自定义的option

在scapy中TCP 所有的option为list，list中的元素为一个truble，tuble中包含两个内容，一个为code，还有就是对应的内容了，至于len会由scapy自动帮我们填充的。

```py
	//构造我们想要的option内容，注意opt是二进制字节，scapy会将opt中的内容当成bit数组发送出去
	 opt = create_opt(opt_len - 2, ip, port)
	 //构造ACK,flag为"A"
    ack = IP(src = src_ip, dst = dst_ip)/TCP(dport = dst_port, sport = src_port, flags = "A")
    //给TCP的option赋值，注意IP也有option，所以我们要用引用进行索引，否则会将IP的option赋值
    ack["TCP"].options = [(opt_code, opt)]
    //读取syn_ack的seq和ack填充到ack报文中
    ack.seq = syn_ack.ack
    ack.ack = syn_ack.seq + 1
    //发送ack，因为三部握手中最后一个ack没有回包，所以只能用send
    send(ack)
```

注意这里，因为scapy只能侦听包而不能截取包，所以服务器发送回来的SYN_ACK还是会被协议栈接受，所以协议栈发送RST给服务器，导致三部握手失败，怎么办呢？

这里我们需要借助iptables这个工具，将RST包drop掉

```
iptables -A OUTPUT -p tcp --tcp-flags --sport xxxx RST RST -j DROP
```
这里xxxx就是我们的scapy构造包中填写的源端口了，这样从这个端口发送的RST包都会被drop掉

###第三步--发送RST，断开连接
因为完成了三部握手已近发送了我们相要的option，已经完成了我们的历史使命了，为了不浪费服务器资源，所以需要断开连接了，这里我们偷了个懒，因为4部挥手比较复杂，而且我们不确定服务器何时发送FIN包，我们这里采用RST直接断开连接。

```py
    rst = IP(src = src_ip, dst = dst_ip)/TCP(dport = dst_port, sport = src_port, flags = "R")
    rst.seq = syn_ack.ack
    rst.ack = syn_ack.seq + 1
    send(rst)
```

这样整个目标就大功告成了

##完整代码
```py
#!/usr/bin/python
from scapy.all import *
src_ip = "192.168.60.51"
dst_ip = "192.168.60.48"
src_port = 63214
dst_port = 8889

def connect_dst(opt_code, opt_len, ip, port):
    syn = IP(src = src_ip, dst = dst_ip)/TCP(dport = dst_port, sport = src_port, flags = "S")
    syn_ack = sr1(syn)
    //自定义create_opt函数，注意scapy填充option长度时是整个opt的长度+2(code和len的长度)
    opt = create_opt(opt_len - 2, ip, port)
    ack = IP(src = src_ip, dst = dst_ip)/TCP(dport = dst_port, sport = src_port, flags = "A")
    ack["TCP"].options = [(opt_code, opt)]
    ack.seq = syn_ack.ack
    ack.ack = syn_ack.seq + 1
    send(ack)
    rst = IP(src = src_ip, dst = dst_ip)/TCP(dport = dst_port, sport = src_port, flags = "R")
    rst.seq = syn_ack.ack
    rst.ack = syn_ack.seq + 1
    send(rst)

```

##参考
- [How to Build a TCP Connection in Scapy](https://www.fir3net.com/Programming/Python/how-to-build-a-tcp-connection-in-scapy.html)
- [add option in tcp with scapy](https://stackoverflow.com/questions/30098954/add-option-in-tcp-with-scapy)