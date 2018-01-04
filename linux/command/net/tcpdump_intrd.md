#tcpdump简明手册

##主要用法
```
tcpdump [ -AdDefIJKlLnNOpqRStuUvxX ] [ -B buffer_size ] [ -c count ]
	[ -C file_size ] [ -G rotate_seconds ] [ -F file ]
	[ -i interface ] [ -j tstamp_type ] [ -m module ] [ -M secret ]
	[ -Q|-P in|out|inout ]
	[ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
	[ -W filecount ]
	[ -E spi@ipaddr algo:secret,...  ]
	[ -y datalinktype ] [ -z postrotate-command ] [ -Z user ]
	[ expression ]
```
上边看起比较复杂，化简如下:

```
	tcpdump 参数  表达式
```
##参数整理
> 这里我将tcpdump分了一下类，便于理解和记忆。

###功能参数
|参数| 输入|作用|TIP|
|---|----|----|---|
|-B |buffer_size|设置抓取缓冲区的大小|在流量较大的时候比较有用|
|-c |count|抓取到指定个数包的时候停止|在抓取ICMP，TCP握手等等的时候相当有用|
|-i|interface|指定抓包的设备|可以指定为any，或者ifconfig中的设备|
|-Q\|-P|in,out，inout|选择抓包的方向||
|-r|file|从file中获取抓包进行分析||。
|-w|file|将抓包结果写入file内|可以配合-r使用|

###显示参数
|参数| 输入|作用|TIP|
|---|----|----|---|
|-D|NULL|显示有哪些可以被抓包的设备|在没有ifconfig的情况使用|
|-f|NULL|外部IP的时候使用数字而非主机名|貌似是为了解决某个缺陷，没什么用|
|-l|NULL|使用stdout作为buffer，可以收到一行就看一行||
|-n|NULL|不对地址(比如, 主机地址, 端口号)进行数字表示到名字表示的转换|对于习惯用ip的很有用|
|-nn|NULL|不对协议和端口转换||
|-N	|NULL|	不打印出host 的域名部分|Host显示'nic' 而不是 'nic.ddn.mil'.|
|-q|NULL|简短输出|在简略分析的时候很有用|
|-t|NULL|不要显示是时间参数||
|-tt|NULL|不对时间做处理|显示Unix时间戳|
|-ttt|NULL|显示行间时间增量|以ms为单位|
|-tttt|NULL|显示时间||
|-ttttt|NULL|显示与首行的时间增量|以ms为单位|
|-v|NULL|显示详细的输出|与-w配合可以每10s报告抓包数量|
|-vv|NULL|显示更详细的输出||
|-vvv|NULL|显示更更详细的输出||
|-x|NULL|以十六进制打印出每个包的头部数据|一般用X|
|-xx|NULL|以十六进制打印出包括链路层的头部数据|一般用XX|
|-X|NULL| 同时会以16进制和ASCII码形式打印出每个包的数据|分析option和http的时候很实用|
|-XX|NULL|同时会以16进制和ASCII码形式打印出每个包的数据,包括链路层||

###辅助参数
|参数| 输入|作用|TIP|
|---|----|----|---|
|-d|NULL|将表达式转换为汇编形式|没看出有什么用|
|-dd|NULL|将表达式转换为c语言端的形式||
|-F|file|将表达式从外部文件输入||
|-L|NULL|列出某个设备所支持数据链路层的类型|在wifi情况下抓包有关|
|-C |file_size|在将抓包结果写入文件的时候，将结果拆分为指定大小的文件|需要配合 -W使用|
|-W|filecout|此选项与-C选项配合使用, 设置一个filecount的缓冲池|后边的文件可能替换掉前边的文件|
|-O	|NULL|不启用进行包匹配时所用的优化代码|当怀疑某些bug是由优化代码引起的, 此选项将很有用.|
|-P|NULL|不把设备设置为杂乱模式||
|-s|snaplen|设置抓包的长度|在默认情况下，tcpdump会截断过长的包，如果设置的过长能抓的包较少|
|-U|NULL|文件写入与包的保存同步|不是等文件缓冲区满再写入|




##条件表达式
###形式
tcpdump的条件表达式可以通过两种方式输入一种是直接用空格分割直接跟在参数后边，或者使用`' '`当成一个字符串传入

tcpdump的条件表达式一般为如下形式

```
	表达式 = 表达元 逻辑运算符 表达式
	表达元 = proto | dir | type  id
	逻辑运算符 = and | or | not | && | || | ! 
```

###表达元
对于表达元补充如下:

```
type 修饰符指定id 所代表的对象类型, id可以是名字也可以是数字. 可选的对象类型有: host, net, port 以及portrange(nt: host 表明id表示主机, net 表明id是网络, port 表明id是端而portrange 表明id 是一个端口范围).  如, 'host foo', 'net 128.3', 'port 20', 'portrange 6000-6008'(nt: 分别表示主机 foo,网络 128.3, 端口 20, 端口范围 6000-6008). 如果不指定type 修饰符, id默认的修饰符为host.

dir 修饰符描述id 所对应的传输方向, 即发往id 还是从id 接收（nt: 而id 到底指什么需要看其前面的type 修饰符）.可取的方向为: src, dst, src 或 dst, src并且dst.(nt:分别表示, id是传输源, id是传输目的, id是传输源或者传输目的, id是传输源并且是传输目的). 例如, 'src foo','dst net 128.3', 'src or dst port ftp-data'.(nt: 分别表示符合条件的数据包中, 源主机是foo, 目的网络是128.3, 源或目的端口为 ftp-data).如果不指定dir修饰符, id 默认的修饰符为src 或 dst.对于链路层的协议,比如SLIP(nt: Serial Line InternetProtocol, 串联线路网际网络协议), 以及linux下指定'any' 设备, 并指定'cooked'(nt | rt: cooked 含义未知, 需补充) 抓取类型, 或其他设备类型,可以用'inbound' 和 'outbount' 修饰符来指定想要的传输方向.

proto 修饰符描述id 所属的协议. 可选的协议有: ether, fddi, tr, wlan, ip, ip6, arp, rarp, decnet, tcp以及 upd.(nt | rt: ether, fddi, tr, 具体含义未知, 需补充. 可理解为物理以太网传输协议, 光纤分布数据网传输协议,以及用于路由跟踪的协议.  wlan, 无线局域网协议; ip,ip6 即通常的TCP/IP协议栈中所使用的ipv4以及ipv6网络层协议;arp, rarp 即地址解析协议,反向地址解析协议; decnet, Digital Equipment Corporation开发的, 最早用于PDP-11 机器互联的网络协议; tcp and udp, 即通常TCP/IP协议栈中的两个传输层协议).

    例如, `ether src foo', `arp net 128.3', `tcp port 21', `udp portrange 7000-7009'分别表示 '从以太网地址foo 来的数据包','发往或来自128.3网络的arp协议数据包', '发送或接收端口为21的tcp协议数据包', '发送或接收端口范围为7000-7009的udp协议数据包'.

    如果不指定proto 修饰符, 则默认为与相应type匹配的修饰符. 例如, 'src foo' 含义是 '(ip or arp or rarp) src foo' (nt: 即, 来自主机foo的ip/arp/rarp协议数据包, 默认type为host),`net bar' 含义是`(ip  or  arp  or rarp) net bar'(nt: 即, 来自或发往bar网络的ip/arp/rarp协议数据包),`port 53' 含义是 `(tcp or udp) port 53'(nt: 即, 发送或接收端口为53的tcp/udp协议数据包).(nt: 由于tcpdump 直接通过数据链路层的 BSD 数据包过滤器或 DLPI(datalink provider interface, 数据链层提供者接口)来直接获得网络数据包, 其可抓取的数据包可涵盖上层的各种协议, 包括arp, rarp, icmp(因特网控制报文协议),ip, ip6, tcp, udp, sctp(流控制传输协议).

    对于修饰符后跟id 的格式,可理解为, type id 是对包最基本的过滤条件: 即对包相关的主机, 网络, 端口的限制;dir 表示对包的传送方向的限制; proto表示对包相关的协议限制)

    'fddi'(nt: Fiber Distributed Data Interface) 实际上与'ether' 含义一样: tcpdump 会把他们当作一种''指定网络接口上的数据链路层协议''. 如同ehter网(以太网), FDDI 的头部通常也会有源, 目的, 以及包类型, 从而可以像ether网数据包一样对这些域进行过滤. 此外, FDDI 头部还有其他的域, 但不能被放到表达式中用来过滤

    同样, 'tr' 和 'wlan' 也和 'ether' 含义一致, 上一段对fddi 的描述同样适用于tr(Token Ring) 和wlan(802.11 wireless LAN)的头部. 对于802.11 协议数据包的头部, 目的域称为DA, 源域称为 SA;而其中的 BSSID, RA, TA 域(nt | rt: 具体含义需补充)不会被检测(nt: 不能被用于包过虑表达式中).
    
```
##最后

	之前一直用Wireshark来抓包，也是用的挺顺畅的，工作以后发现，在服务器上根本没机会用什么图形界面，迫于无奈，只能学习用tcpdump来抓包，其实已经拖了好久，最近试用了几次，发现其实也不算特别难，但需要记忆的比较多，现根据man手册整理出此文。