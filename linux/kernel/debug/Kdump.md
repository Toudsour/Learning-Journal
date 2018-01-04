#使用Kdump 
##简介
	Kdump是一个基于Kexec的用来转储Linux崩溃时进行的标准工具，当Linux Kernel 崩溃的时候，Kdump会将崩溃
	崩溃日志记录下来，并将内存转储下来，保存在本地硬盘或者通过SSH,NFS协议转储到不同机器上。
##原理简介
> - Kdump分为两个组件: Kexec 和 Kdump
>  
> - Kexec是一种内核的快速启动工具，可以使新的内核在正在运行的内核（生产内核）的上下文中启动，而不需要通过耗时的BIOS检测，方便内核开发人员对内核进行调试。 

> - Kdump是一种有效的内存转储工具，启用Kdump后，生产内核将会保留一部分内存空间，用于在内核崩溃时通过Kexec快速启动到新的内核，这个过程不需要重启系统，因此可以转储崩溃的生产内核的内存镜像。
> 

##安装
使用`apt-get`,`rpm`或者`yum`进行安装即可。

以`yum`为例:

```sh
	sudo yum install kdump 
```

当然再次之前，对于你的内核需要一定的配置要求,在你的`.config`里面需要有如下配置:

```sh
CONFIG_DEBUG_INFO=y
CONFIG_CRASH_DUMP=y
CONFIG_PROC_VMCORE=y
```

##配置
对于`Kdump`而言需要不少的配置，接下来详细说明

###内核启动参数
Kdump需要在内核启动的时候预留一定的空间用来抓取内存，故需要在启动参数里边做一些设置，对于`centos`或者`ubuntu`等，需要修改一下`/etc/gurb.conf`，如下:

```
title CentOS (2.6.32-696.6.3.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-696.6.3.el6.x86_64 ro root=/dev/mapper/vg_centos6-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_LVM_LV=vg_centos6/lv_swap rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg_centos6/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
        initrd /initramfs-2.6.32-696.6.3.el6.x86_64.img
```
这里默认存在了`crashkernel=auto`,若果没有，需要加上。

但对于内存小于4G的电脑来说，如虚拟机，需要将`crashkernel`设置为具体大小，一般设置为`crashkernel=128M`即可。

###Kdump配置文件
Kdump的配置文件在`/etc/kdump.conf`，对于Kdump的配置选项都有相应的配置说明，如果没有特殊要求默认如下:

```sh
path /var/crash  
#保存文件路径
core_collector makedumpfile -c --message-level 1 -d 31  
#此行设置保存内存镜像内容的级别，-c表示使用makedumpfile压缩数据，--message-level 1表示提示信息的级别（1表示只显示进度信息）-d 31表示不复制所有可以去掉的内存页（包括zero page, cache page, cache private, user data, free page等）。
default reboot
#此行表示如果kdump转储内存镜像失败后的执行的动作，默认为挂载根文件系统并执行/sbin/init进程，可以更改为：reboot, halt, poweroff, shell等。
```

###设置为开机启动
使用`chkconfig`即可

```sh
chkconfig kdump on
/etc/init.d/kdump start
```
但这里要注意，对于更改了内核版本号的同学而言，会遇到如下错误

**Warn a user to use maxcpus=1 instead of nr_cpus=1 for older kernels**

或者

**Your kernel is old, please use maxcpus=1 instead of nr_cpus=1**

的错误，因为对于`/etc/init.d/kdump`中存在如下的如下检测:

```sh
	if echo "$KDUMP_COMMANDLINE_APPEND" | grep -q nr_cpus;
   	then
       ver=`uname -r`
       maj=`echo $ver | cut -d'-' -f1`
       min=`echo $ver | cut -d'-' -f2`
       min=${min%%.*}
       if [ "$maj" = "2.6.32" ] && [ $min -lt 171 ]
       then
           echo "Your kernel is old, please use maxcpus=1 instead of nr_cpus=1"
           return 1
       fi
    fi
```
于是我们的版本号是自定义的，所以就被误杀了，注释掉就好。

最后重启电脑就行。

###验证

```
	echo c > /proc/sysrq-trigger
```


##参考
- [深入探索 Kdump，第 1 部分：带你走进 Kdump 的世界](https://www.ibm.com/developerworks/cn/linux/l-cn-kdump1/)
- [archlinux-wiki-kdump](https://wiki.archlinux.org/index.php/Kdump)
- [Kdump和Crash的配置方法与内核故障原因分析](https://wenku.baidu.com/view/c04b6f13bcd126fff7050bf0.html)