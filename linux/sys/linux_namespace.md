#Linux Namespace
##简介
最近公司服务器升级，将centos 6.8 升级到了 7.0，自然内核版本也升级了，在学习一波之后发现，3.0的内核中加入namespace的概念，将原来的chroot的隔离做的很彻底，在这里整理一波namespace的相关概念。

Namespace分为几个类型，分别为

- UTS: hostname和时间
- IPC: 进程间通信 
- PID: "chroot"进程树
- NS: 挂载点，首次登陆Linux
- NET: 网络访问，包括接口
- USER: 将本地的虚拟user-id映射到真实的user-id

那么namespace到底是什么呢，拿网络来说把，net namespace的实现就是在内核里划分了多个网络子系统，不同namespace里边有各自的驱动，各自的/proc/sys/net参数，不同的socket。其余的namespace也是类似，将系统的子系统进行抽象，而一个namespace就是一个实例,这样就做到了在不同类型如网络，pid下做到隔离的效果。

那么namespace怎么使用呢?

系统API层面,主要是三个接口:

```c
int clone(int (*child_func)(void *), void *child_stack
            , int flags, void *arg);
/*
flags：
    指定一个或者多个上面的CLONE_NEW*（当然也可以包含跟namespace无关的flags），
    这样就会创建一个或多个新的不同类型的namespace，
    并把新创建的子进程加入新创建的这些namespace中。
*/

int setns(int fd, int nstype);

/*
fd：
    指向/proc/[pid]/ns/目录里相应namespace对应的文件，
    表示要加入哪个namespace

nstype：
    指定namespace的类型（上面的任意一个CLONE_NEW*）：
    1. 如果当前进程不能根据fd得到它的类型，如fd由其他进程创建，
    并通过UNIX domain socket传给当前进程，
    那么就需要通过nstype来指定fd指向的namespace的类型
    2. 如果进程能根据fd得到namespace类型，比如这个fd是由当前进程打开的，
    那么nstype设置为0即可
*/

int unshare(int flags);

/*flags：
    指定一个或者多个上面的CLONE_NEW*，
    这样当前进程就退出了当前指定类型的namespace并加入到新创建的namespace
*/
```

当然有一些软件集成了系统的API来方便我们使用，有以下两个软件:

- nsenter：加入指定进程的指定类型的namespace，然后执行参数中指定的命令
- unshare：离开当前指定类型的namespace，创建且加入新的namespace，然后执行参数中指定的命令

那么如何查看namespace呢，首先要提到的是namespace是和pid绑定的，并且子进程会默认情况下的会继承父进程的namespace，另一方面，当namespace下所有pid都退出后，这个namespace也就退出了。

因为pid的相关信息都在/proc/\$pid/下，故namespace的相关信息就在/proc/\$pid/ns下了,例如：

```shell
➜  ns ll /proc/$$/ns
总用量 0
lrwxrwxrwx 1 Toud Toud 0 1月  31 22:56 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 Toud Toud 0 1月  31 22:56 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 Toud Toud 0 1月  31 22:56 net -> net:[4026531956]
lrwxrwxrwx 1 Toud Toud 0 1月  31 22:56 pid -> pid:[4026531836]
lrwxrwxrwx 1 Toud Toud 0 1月  31 22:56 user -> user:[4026531837]
lrwxrwxrwx 1 Toud Toud 0 1月  31 22:56 uts -> uts:[4026531838]
```

接下来我们就对不同的namespace进行介绍。

##详解
###UTS

这大概是所有namespace里边最简单的一个，UTS namespace主要干什么的呢？隔离HostName和时区星期。

有人可能问，这有什么用啊？单一一个UTS是没什么用的，但是它起到了一个标识的作用。当你创建一个完全隔离的namespace时，你总要与之前有所区分吧，之前是root，现在还是叫root，这时UTS就派上了用场。

如何使用UTS呢，只要创建一个新的namespace就隔离了，UTS没有继承关系。

###IPC
IPC我们很熟悉，进程间通信，那么IPC namespace 主要针对以下几种进程间通信

- 信号量
- 消息队列
- 共享内存

为什么只有这几种呢？因为管道在NS做了隔离，socket在Net中隔离，而信号天然就是隔离的。

和UTS类似，创建一个新的namespace就可以了。

###NS
NS 也叫 FS Namespace,这样就很明了，显然是用来隔离文件的。
从NS开始，接下来的namespace存在一个继承的关系，也就是新创建的namespace会继承父namespace的一些东西，在创建一个NS namespace的时，新创建的NS会拷贝父NS里的挂摘点列表，在这之后，子Namespace就与父Namespace无关了。在各自的挂载点里mount或者unmout，都不会影响对方。

当然也有特殊的时候，如果想要做到文件共享怎么办么，这时候就要用到[Shared subtrees](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)了，比较复杂，在此不做深究。

###PID
PID namespaces用来隔离进程的ID空间，使得不同pid namespace里的进程ID可以重复且相互之间不影响。

PID namespace可以嵌套，也就是说有父子关系，在当前namespace里面创建的所有新的namespace都是当前namespace的子namespace。父namespace里面可以看到所有子孙后代namespace里的进程信息，而子namespace里看不到祖先或者兄弟namespace里的进程信息。

目前PID namespace最多可以嵌套32层，由内核中的宏MAX\_PID\_NS\_LEVEL来定义

Linux下的每个进程都有一个对应的/proc/PID目录，该目录包含了大量的有关当前进程的信息。 对一个PID namespace而言，/proc目录只包含当前namespace和它所有子孙后代namespace里的进程的信息。

在Linux系统中，进程ID从1开始往后不断增加，并且不能重复（当然进程退出后，ID会被回收再利用），进程ID为1的进程是内核启动的第一个应用层进程，一般是init进程（现在采用systemd的系统第一个进程是systemd），具有特殊意义，当系统中一个进程的父进程退出时，内核会指定init进程成为这个进程的新父进程，而当init进程退出时，系统也将退出。

除了在init进程里指定了handler的信号外，内核会帮init进程屏蔽掉其他任何信号，这样可以防止其他进程不小心kill掉init进程导致系统挂掉。不过有了PID namespace后，可以通过在父namespace中发送SIGKILL或者SIGSTOP信号来终止子namespace中的ID为1的进程。

由于ID为1的进程的特殊性，所以每个PID namespace的第一个进程的ID都是1。当这个进程运行停止后，内核将会给这个namespace里的所有其他进程发送SIGKILL信号，致使其他所有进程都停止，于是namespace被销毁掉。

###Net

这大概是所有namespace里边最复杂的一个了，所有和网络相关的device,socket,route,iptable等等都进行了隔离。在从2.6到3.0变化最大的就是网络模块。

Net Namespace相当复杂，在此不做深究，在之后研究协议栈在进行详细的整理。

##实现

这里整理下Namespace大致的实现：

在新的Linux内核中，在每个进程对应的task结构体struct task_struct中，增加了一个叫nsproxy的字段，类型是struct nsproxy

```c


struct task_struct {
  ...
  /* namespaces */
  struct nsproxy *nsproxy;
  ...
}

struct nsproxy {
  atomic_t count;
  struct uts_namespace *uts_ns;
  struct ipc_namespace *ipc_ns;
  struct mnt_namespace *mnt_ns;
  struct pid_namespace *pid_ns_for_children;
  struct net       *net_ns;

```

一个nsproxy实例中包含了指向五种namespace结构的指针，一个进程包含一个nsproxy，代表了这个process所在的各个namespace。当process调用unshare()函数时，内核就会根据是否有新的Namespace分配一个新的nsproxy结构，并且根据参数，新建部分namespace，并复制保留其余的namespace，而如果没有的话，则会复用旧的。

而这里不同的Namespace 在struct nsproxy下的不同实现啦。


##参考
- [Linux Namespace和Cgroup](https://segmentfault.com/a/1190000009732550)
- [Linux命名空间学习教程](http://dockone.io/article/82)
- [LinuxkernelNamespace源码分析](https://www.2cto.com/net/201612/581701.html)

