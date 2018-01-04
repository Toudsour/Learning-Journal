**getconf**
##用途
将系统配置变量值写入标准输出。
##用法
    getconf [-v 规范] 变量名 [路径名]
    getconf -a [路径名]
##介绍

怎么说呢，这个命令的man手册极其坑爹，我至今不知道这些规范，路径名是干啥用的

而且不同操作系统getconf 中的变量名不同一样，又可能你这有，那儿就没有了

所以还是getconf -a然后grep吧

##参考
http://blog.csdn.net/bytxl/article/details/9296435
