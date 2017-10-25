#Linux下的解压命令
##file.tar.gz/file.tar.bz2
###命令
    tar
###参数
- c: create,创建一个压缩文件
- x: extract,解压一个压缩文件夹
- t: list,列出压缩文件中的文件,(x/t/c)三者只能有一个

------------------------

- z: 文件使用gzip格式进行处理
- j: 文件使用bzip2格式进行处理
- v: 是否显示解压过程,或者压缩过程
- f: 使用压缩文档名,f后边需要紧接着文档名
- p: 使用原文件原来的属性
- P: 使用绝对路径来压缩
- N: 后边紧接着日期(yyyy/mm/dd)才会被打包

##file.rpm
###命令
    rpm2cpio : 将rpm文件解压成cpio，并直接输出到电脑屏幕上，没有参数
    cpio : 处理cpio格式的压缩文件
###参数
- i: 进入read-out模式，也就是解压模式，默认是从标准输入流中解压文件
- v: 列出解压过程
- d: 在需要的地方创立目录
- m: 保持原先文件的跟新时间
- f: 从文件中解压缩