#Linux系统开机过程
##加载内核
这一部分不详细展开讲，这一部分涉及到硬件底层，在介绍Linux内核的时候再展开讲解，大致过程为BIOS加载grub，grub引导系统内核,内核建立虚拟内存和根目录系统，并启动init
##init系统
###简介
init是系统的第一个程序，当然对于较新的系统而言，有可能不是init，而是systemd，这次我们介绍init，init大致会做这几件事:
- 读取/etc/inittab
- 执行rc.sysinit里边的脚本
- 根据/etc/inittab中的文件执行指定/etc/rc.d/rc\*目录下的脚本
- 执行rc.local
以下就是详解讲解
###inittab
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

##sysinit
这个比较复杂，和下边要将的rc.d类似，不同之处在于在与，rc.d分为不同等级加载，但是sysinit无论那个等级都会加载，主要内容涉及系统的字体，日志，网络等等，在系统加载界面中输出的很多内容都是在这个脚本里边的

##rc.d












#引用
- [rc.sysinit 解析](https://www.cnblogs.com/lpfuture/p/5707658.html)
- [Linux启动过程详解（inittab、rc.sysinit、rcX.d、rc.local](https://www.cnblogs.com/sysk/p/4778976.html)
- [Linux如何实现开机启动程序详解](https://www.cnblogs.com/gzggyy/archive/2012/08/07/2626574.html)
- [linux rc.local profile.d/file 执行区别  ](http://blog.163.com/jiangh_1982/blog/static/121950520163734650715/)











