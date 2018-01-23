#Linux init
#init系统
##简介
init是系统的第一个程序，当然对于较新的系统而言，有可能不是init，而是systemd，这次我们介绍init，init大致会做这几件事:
- 读取/etc/inittab
- 执行rc.sysinit里边的脚本
- 根据/etc/inittab中的文件执行指定/etc/rc.d/rc\*目录下的脚本
- 执行rc.local
以下就是详解讲解
##inittab
这个脚本比较简单，也比较重要，我们先看一下它长什么样:
```sh
# inittab is only used by upstart for the default runlevel.
#
# ADDING OTHER CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# System initialization is started by /etc/init/rcS.conf
#
# Individual runlevels are started by /etc/init/rc.conf
#
# Ctrl-Alt-Delete is handled by /etc/init/control-alt-delete.conf
#
# Terminal gettys are handled by /etc/init/tty.conf and /etc/init/serial.conf,
# with configuration in /etc/sysconfig/init.
#
# For information on how to write upstart event handlers, or how
# upstart works, see init(5), init(8), and initctl(8).
#
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
#
id:3:initdefault:
```
基本上都是注释,注释上说明了不同层次的运行等级控制使用什么文件，而inittab决定了整个系统默认等级，运行等级总共有7个，使用了6个，在注释上有说明:
- 0 : 关机，不要使用
- 1 : 单用户模式
- 2 : 多用户，但是不能联网
- 3 : 完全的多用户，大多数服务器都运行在这个等级
- 4 : 未使用
- 5 : X11界面，一般家庭主机都是这个等级
- 6 : 重启，不要使用
至于不同的运行等级有什么左右，在其他文章中再讲

## sysinit
这个比较复杂，和下边要将的rc.d类似，不同之处在于在与，rc.d分为不同等级加载，但是sysinit无论那个等级都会加载，主要内容如下:

- 激活 udev 和 selinux
- 设置定义在/etc/sysctl.conf 中的内核参数
- 设置系统时钟
- 加载 keymaps
- 使能交换分区
- 设置主机名(hostname)
- 根分区检查和 remount
- 激活 RAID 和 LVM 设备
- 开启磁盘配额
- 检查并挂载所有文件系统
- 清除过期的 locks 和 PID 文件

[这里有详细的讲解](https://www.cnblogs.com/lpfuture/p/5707658.html)
##rc.d
首先介绍一下/etc/rc.d目录下的结构
rc.d
|-- init.d
|-- rc
|-- rc.local
|-- rc.sysinit
|-- rc0.d
|-- rc1.d
|-- rc2.d
|-- rc3.d
|-- rc4.d
|-- rc5.d
`-- rc6.d

rc.sysinit 就是我们之前所说的sysinit,rc负责在runlevel改变的时候调用，其余的慢慢讲解
###init.d
init.d/
├── auditd
├── blk-availability
├── crond
├── fdfs_storaged
├── fdfs_trackerd
├── functions
├── halt
├── influxdb
├── ip6tables
├── iptables
`── ....

这是init.d下面的内容，对于init.d里边的文件都是一个个脚本，这些脚本会在rc\*.d里边被使用

###rc\*.d
这里用\*代表不同数字，这里的数字是和运行等级相对应的，不同的runlevel决定了系统会调用那个rc\*.d下边的内容，举rc3.d目录内容如下:
rc3.d/
├── K10saslauthd -> ../init.d/saslauthd
├── K14zabbix-agent -> ../init.d/zabbix-agent
├── K15svnserve -> ../init.d/svnserve
├── K25squid -> ../init.d/squid
├── K50netconsole -> ../init.d/netconsole
├── K75ntpdate -> ../init.d/ntpdate
├── K87multipathd -> ../init.d/multipathd
├── K87restorecond -> ../init.d/restorecond
├── K89rdisc -> ../init.d/rdisc
├── K92ip6tables -> ../init.d/ip6tables
├── K92iptables -> ../init.d/iptables
├── S01sysstat -> ../init.d/sysstat
├── S02lvm2-monitor -> ../init.d/lvm2-monitor
├── S07iscsid -> ../init.d/iscsid
├── S10network -> ../init.d/network
├── S11auditd -> ../init.d/auditd
├── S12rsyslog -> ../init.d/rsyslog
├── S13irqbalance -> ../init.d/irqbalance
├── S13iscsi -> ../init.d/iscsi
`── ....

从tree中我们可以看到，这里的脚本是对init.d的软连，K开头说明这是关闭的时候调用的，而S开头则说明是启动时被调用，后面的数字代表了他们的执行循序，为什么要有执行顺讯呢?因为服务之间存在这依赖关系，所以必须依次启动。
###rc.local
这里边是用户自定义的内容，如果你想要对于全用户自定义一些内容就可以在这里添加。
在rc.local执行完后，就会显示登录界面，接下来就是单个用户的启动内容了，顺序如下:
- /etc/profile.d/file
- /etc/profile
- /etc/bashrc
- /root/.bashrc
- /root/.bash_profile


##Sysvinit 的管理和控制功能
此外，在系统启动之后，管理员还需要对已经启动的进程进行管理和控制。原始的 sysvinit 软件包包含了一系列的控制启动，运行和关闭所有其他程序的工具。
- halt
停止系统。
- init
这个就是 sysvinit 本身的 init 进程实体，以 pid1 身份运行，是所有用户进程的父进程。最主要的作用是在启动过程中使用/etc/inittab 文件创建进程。
- poweroff
等于 shutdown -h –p，或者 telinit 0。关闭系统并切断电源。
- reboot
等于 shutdown –r 或者 telinit 6。重启系统。
- runlevel
读取系统的登录记录文件(一般是/var/run/utmp)把以前和当前的系统运行级输出到标准输出设备。
- shutdown
以一种安全的方式终止系统，所有正在登录的用户都会收到系统将要终止通知，并且不准新的登录。
- killall5
就是 SystemV 的 killall 命令。向除自己的会话(session)进程之外的其它进程发出信号，所以不能杀死当前使用的 shell。
- last
回溯/var/log/wtmp 文件(或者-f 选项指定的文件)，显示自从这个文件建立以来，所有用户的登录情况。
- lastb
作用和 last 差不多，默认情况下使用/var/log/btmp 文件，显示所有失败登录企图。
- mesg
控制其它用户对用户终端的访问。
- pidof
找出程序的进程识别号(pid)，输出到标准输出设备。
- sulogin
当系统进入单用户模式时，被 init 调用。当接收到启动加载程序传递的-b 选项时，init 也会调用 sulogin。
- telinit
实际是 init 的一个连接，用来向 init 传送单字符参数和信号。
- utmpdump
以一种用户友好的格式向标准输出设备显示/var/run/utmp 文件的内容。
- wall
向所有有信息权限的登录用户发送消息。

不同的 Linux 发行版在这些 sysvinit 的基本工具基础上又开发了一些辅助工具用来简化 init 系统的管理工作。比如 RedHat 的 RHEL 在 sysvinit 的基础上开发了 initscripts 软件包，包含了大量的启动脚本 (如 rc.sysinit) ，还提供了 service，chkconfig 等命令行工具，甚至一套图形化界面来管理 init 系统。其他的 Linux 发行版也有各自的 initscript 或其他名字的 init 软件包来简化 sysvinit 的管理。

只要您理解了 sysvinit 的机制，在一个最简的仅有 sysvinit 的系统下，您也可以直接调用脚本启动和停止服务，手动创建 inittab 和创建软连接来完成这些任务。因此理解 sysvinit 的基本原理和命令是最重要的。您甚至也可以开发自己的一套管理工具。

#引用
- [rc.sysinit 解析](https://www.cnblogs.com/lpfuture/p/5707658.html)
- [Linux启动过程详解（inittab、rc.sysinit、rcX.d、rc.local](https://www.cnblogs.com/sysk/p/4778976.html)
- [Linux如何实现开机启动程序详解](https://www.cnblogs.com/gzggyy/archive/2012/08/07/2626574.html)
- [linux rc.local profile.d/file 执行区别  ](http://blog.163.com/jiangh_1982/blog/static/121950520163734650715/)


