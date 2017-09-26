#packetdrill使用简介
##简介
packetdrill是一款Google出品的内核协议栈测试工具，根据其[官网](https://code.google.com/archive/p/packetdrill)的介绍翻译如下:

    The packetdrill scripting tool enables quick, precise tests for entire TCP/UDP/IPv4/IPv6 network stacks, from the system call layer down to the NIC hardware. packetdrill currently works on Linux, FreeBSD, OpenBSD, and NetBSD. It can test network stack behavior over physical NICs on a LAN, or on a single machine using a tun virtual network device.
    
    packetdrill是一个脚本工具能够快速清晰的测试整个TCP/UDP/IPv4/IPv6协议栈，包括从系统调用层到网卡硬件。packetdrill现在可以工具在Linux,FreeBSD,OpenBSD,和NetBSD.它可以测试测试使用物理网卡网络协议栈，或者工作在TUN上的设备。

##安装
可以在[github](https://github.com/google/packetdrill)上载
直接到相应的目录下:

```
./configure
make
```

在test文件中有不少测试Case可以拿来直接使用,使用语法为

```
    packetdrill test_file_path
```

**PS:packetdrill需要flex和bison**

##语法
很不幸的packetdrill的语法至今没有详细的介绍，他们的官网没有详细的介绍，只有在其的研究论文上有所[介绍](https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/41316.pdf)，如果英文不好，可以看[zhangskd的介绍](http://blog.csdn.net/zhangskd/article/details/25329579)

如果觉得不够清晰的,google官网上指了一条路，查看源码中`parser.output`。

`parser.output`分为四部分:

- Grammer :packetdrill脚本语法，为BNF范式的形式，不了解的可以百度一下或者查看一个编译原理相关内容
- Terminals :终结符
- Nonterminals :非终结符
- State :解析自动机

如果你需要定制一些需求，用到详细了解packetdrill的语法，可以在这里详细查看

##总结
优点

- 使用简单方便，能够自己构造使用场景
- 提供了大量的测试case，对于协议栈的标准内容，可以直接拿来测试

缺点

- 所有的测试都是基于标准协议栈，不支持自定义，如非法的packet，自定的tcp option
- 基本上只能对本机测试，不能从对端进行测试
- 文档较少，语法坑爹
    
##参考
- [github - packetdrill](http://blog.csdn.net/dog250/article/details/51923079)
- [不错的网络协议栈测试工具 — Packetdrill](http://blog.csdn.net/zhangskd/article/details/25329579)
- [packetdrill框架点滴剖析以及TCP重传的一个细节](http://blog.csdn.net/dog250/article/details/51934338)
- [通过packetdrill构造的包序列理解TCP快速重传机制](http://blog.csdn.net/dog250/article/details/51923079)
