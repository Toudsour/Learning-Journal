#Liunx日志

##Syslog 

###简介
Unix/Linux系统中的大部分日志都是通过一种叫做syslog的机制产生和维护的。syslog是一种标准的协议，分为客户端和服务器端，客户端是产生日志消息的一方，而服务器端负责接收客户端发送来的日志消息，并做出保存到特定的日志文件中或者其他方式的处理。

在Linux中，常见的syslog服务器端程序是syslogd守护程序。这个程序可以从三个地方接收日志消息：（1）Unix域套接字 /dev/log；（2）UDP端口514；（3）特殊的设备/dev/klog（读取内核发出的消息）。相应地，产生日志消息的程序就需要通过上述三种方式写入消息，对于大多数程序而言就是向/dev/log这个套接字发送日志消息。













![](http://img.blog.csdn.net/20130513133929220)








而接受到消息的syslog会根据配置文件将日志存储到配置中对应的地方，当然也有可能发送给其他主机。

###Syslog格式
要注意的是syslog其实没有统一的格式，下边的格式可以看做一个规范。
syslog的格式分为以下三部分:

	<PRI> <Header> <MSG>

现在有一个syslog消息为

	<30>Oct 9 22:33:20 hlfedora auditd[1787]: The audit daemon is exiting.

根据他我们下边进行分别说明 :
	
####Pri部分
该部分形式为: 

***<数字>***

在刚才的消息里边该部分为:

***<30>***

数字主要标记了这个消息是来自模块和这个消息的紧急程度,他们的组合方式如下 ：

> 结果 = 模块号 * 8 + 程度编号

对于模块编号定义如下:

     Numerical Code      Facility

          0             kernel messages
          1             user-level messages
          2             mail system
          3             system daemons
          4             security/authorization messages (note 1)
          5             messages generated internally by syslogd
          6             line printer subsystem
          7             network news subsystem
          8             UUCP subsystem
          9             clock daemon (note 2)
         10             security/authorization messages (note 1)
         11             FTP daemon
         12             NTP subsystem
         13             log audit (note 1)
         14             log alert (note 1)
         15             clock daemon (note 2)
         16             local use 0  (local0)
         17             local use 1  (local1)
         18             local use 2  (local2)
         19             local use 3  (local3)
         20             local use 4  (local4)
         21             local use 5  (local5)
         22             local use 6  (local6)
         23             local use 7  (local7)

       Note 1 - Various operating systems have been found to utilize
          Facilities 4, 10, 13 and 14 for security/authorization,
          audit, and alert messages which seem to be similar.
       Note 2 - Various operating systems have been found to utilize
          both Facilities 9 and 15 for clock (cron/at) messages.
          
         
对于紧急程度编号定义如下:

   	
   	Numerical Code  		Severity
        
         0       		Emergency: system is unusable
         1       		Alert: action must be taken immediately
         2       		Critical: critical conditions
         3       		Error: error conditions
         4       		Warning: warning conditions
         5       		Notice: normal but significant condition
         6       		Informational: informational messages
         7       		Debug: debug-level messages


####HEADER部分 
该部分形式如下:

***Mouth day hh:mm:ss host/ip***

前半部分为日期和时间，后半部分为主机名或者ip

在刚才的消息里，对应为:

***Oct 9 22:33:20 hlfedora***


####MSG部分 
该部分格式如下:

***(进程名[进程号]) : xxxxx***

括号内的部分是可选的，用来标记进程和pid，冒号后边的内容由程序自定义.

###syslog配置文件
syslog的配置文件根据其实现的不同可能为`/etc/syslog.conf`或者`/etc/rsyslog.conf`等，以`rsyslog`
举例:

```sh
# rsyslog v5 configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### MODULES ####

$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
#$ModLoad immark  # provides --MARK-- message capability

# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
#$ModLoad imtcp
#$InputTCPServerRun 514


#### GLOBAL DIRECTIVES ####

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# File syncing capability is disabled by default. This feature is usually not required,
# not useful and an extreme performance hit
#$ActionFileEnableSync on

# Include all config files in /etc/rsyslog.d/
$IncludeConfig /etc/rsyslog.d/*.conf

#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 *

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log

# ### begin forwarding rule ###
# The statement between the begin ... end define a SINGLE forwarding
# rule. They belong together, do NOT split them. If you create multiple
# forwarding rules, duplicate the whole block!
# Remote Logging (we use TCP for reliable delivery)
#
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
#$WorkDirectory /var/lib/rsyslog # where to place spool files
#$ActionQueueFileName fwdRule1 # unique name prefix for spool files
#$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
#$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
#$ActionQueueType LinkedList   # run asynchronously
#$ActionResumeRetryCount -1    # infinite retries if host is down
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
#*.* @@remote-host:514
# ### end of the forwarding rule ###
```

因为自带注释，所以不难理解，主要配置就是日志记录规则，根据pri，将不同的日志记录在不同文件内，规则如下:

[类型.级别 (；类型.级别) `TAB` 动作]

类型和级别在之前已经介绍过了，动作主要有如下几个:


- /filename   日志文件。由绝对路径指出的文件名，此文件必须事先建立； 如果在文件名之前加上减号(-)，则表示不将日志信息同步刷新到磁盘上(使用写入缓存)，这样可以提高日志写入性能，但是增加了系统崩溃后丢失日志的风险。
- @host       远程主机； @符号后面可以是ip,也可以是域名，默认在/etc/hosts文件下loghost这个别名已经指定给了本机。
- user1,user2 指定用户。如果指定用户已登录，那么他们将收到信息； 
- *           所有用户。所有已登录的用户都将收到信息。

##常用日志文件
###简介
这里主要列举一下常用的日志文件，以及他们的作用

- /var/log/messages：记录Linux内核消息及各种应用程序的公共日志信息，对于未使用独立日志文件的应用程序或服务，一般都可以从该文件获得相关的事件记录信息。
- /var/log/cron：记录crond计划任务产生的事件消息。 
- /varlog/dmesg：记录Linux系统在引导过程中的各种事件信息。
- /var/log/maillog：记录进入或发出系统的电子邮件活动。 
- /var/log/secure：记录用户登录认证过程中的事件信息。
- /var/log/lastlog：最近几次成功登录事件和最后一次不成功登录事件。
- /var/log/wtmp：记录每个用户登录、注销及系统启动和停机事件。
- /var/run/utmp：记录当前登录的每个用户的详细信息 

注意: 最后三个日志文件为二进制形式，不能用cat显示,需要who、w、users、last和ac等用户查询命令来获取日志信息,可以查看参考中最后一篇文章详细探究.

##参考
- [系统日志及分析](http://www.cnblogs.com/yingsong/p/6022181.html)
- [Linux系统故障分析与排查](http://os.51cto.com/art/201405/438510.htm)
- [关于syslog](http://blog.csdn.net/smstong/article/details/8919803)
- [linux下syslog使用说明](http://blog.chinaunix.net/uid-25120309-id-3359929.html)
- [Linux日志文件utmp、wtmp、lastlog、messages](http://itshine.blog.51cto.com/648476/489687/)
	
