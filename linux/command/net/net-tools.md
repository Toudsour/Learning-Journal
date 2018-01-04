#网络管理工具
##简介
在日常网络开发过程中，免不了碰到一些问题去定位，比如为什么这个包发布出去，或者这个包发出去但是对端没有收到等等。

但是很多时候网络对于我们来说只是socket种收到的一段段字符串，对于路由，arp映射，网卡设备这些都被藏在内核的协议栈了，我们难以了解到其中的情况。

系统提供了一系列系统工具便于我们查看网络的情况，主要常用的套件有以下几个:

- net-tools套件
- iproute2套件
- ping 
- traceroute
- tcpdump



在这里我只是简单的介绍一下上面提到的工具，以在解决网络问题时提供一些解决思路，在用到的时候可以再详细的研究

##net-tools
net-tools并非是一个工具的名字，而是一组工具的名字，具体列出来就很熟悉了:

- ifconfig :用来配置和查看网络设备
- arp : 用来操作arp缓存
- route : 用来操作内核路由缓存表
- netstat : 用来输出内核网络子系统的信息，包括很多方面
- hostname : 用来查看设置本机的hostname，除此之外hostname还有一些变形用来设置域名

但是net-tools在2001年就已经停止开发，现在net-tools的功能都被包含在iproute2套件中，但是在实际使用中，net-tools仍然不失为一个有力的工具。

##iproute2
iproute2是一套比net-tools更为强大先进的套件，主要包含以下几个工具:

- ip : 用于管理网络表和网络接口
- tc : 用于流量控制管理
- ss : 用于转储套接字统计信息
- lnstat : 用于转储Linux网络统计信息
- brige : 用于管理网桥地址和设备

iproute2虽然包含5个工具，但是我们常用的往往只有ip工具一个，因为ip工具的强大，net-tools中大部分工具的功能都被其承包了，所以我们需要花一些时间去熟悉工具。

这里给出一张net-tools和iproute2命令的对照表:

![et-tools和iproute2命令的对照表](http://www.linuxidc.com/upload/2014_06/14060411029186.png)
##ping
当我们网络不通的时候，我们往往会使用`ping baidu.com`来查看我们的网络状况，ping本身是一个发送ICMP echo请求的工具，但是这个功能可以帮我们定位不少问题，在这里ping还可以指定packet大小等等。

##traceroute
traceroute用来显示报文被转发的路径，可以用来定位一些路由问题。要注意的是，traceroute除了用ICMP协议，还可以使用TCP或者UDP进行探测。

##tcpdump
可能在平时，我们用到最多的网络工具就是tcpdump，主要用于根据我们制定的筛选规则将符合条件的包以指定的方式显示出来。tcpdump支持各种协议，可定制各种规则，可能唯一不太好的方式，查看包不如图形功能那么方便，不过可以与wireshark相结合，用tcpdump抓包，使用wireshark分析

##补充
- [Linux常用网络工具总结](http://int32bit.me/2016/05/04/Linux常用网络工具总结/)
