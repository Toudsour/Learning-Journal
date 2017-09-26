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
|-r|file|从file中获取抓包||。
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


###辅助参数
|参数| 输入|作用|TIP|
|---|----|----|---|
|-d|NULL|将表达式转换为汇编形式|没看出有什么用|
|-dd|NULL|将表达式转换为c语言端的形式||
|-F|file|将表达式从外部文件输入||
|-L|NULL|列出某个设备所支持数据链路层的类型|在wifi情况下抓包有关|
|-C |file_size|在将抓包结果写入文件的时候，将结果拆分为指定大小的文件|需要配合 -W使用|
|-O	|NULL|不启用进行包匹配时所用的优化代码|当怀疑某些bug是由优化代码引起的, 此选项将很有用.|
|-P|NULL|不把设备设置为杂乱模式||
|-s|snaplen|设置抓包的长度|在默认情况下，tcpdump会截断过长的包，如果设置的过长能抓的包较少|
|-U|NULL|文件写入与包的保存同步|不是等文件缓冲区满再写入|






##最后

	之前一直用Wireshark来抓包，也是用的挺顺畅的，工作以后发现，在服务器上根本没机会用什么图形界面，迫于无奈，只能学习用tcpdump来抓包，其实已经拖了好久，最近试用了几次，发现其实也不算特别难，但需要记忆的比较多，现根据man手册整理出此文。