#VIM多窗口
##新建窗口
**横向切割**
- new  窗口名
- split/sp 文件名

**纵向切割**
- vnew 文件名
- vsplit/vsp 文件名

**一次打开多个**
- vim -o a b c 横向切割展示所有文件
- all 横向切割暂时所有文件
- vertical all 很想

##关闭窗口
- 普通的关闭文件的命令:q!,qw,ZZ
- 仅仅关闭窗口:close
- 关闭当前窗口:tabc
- 关闭所有窗口:tabo
- 保留当前窗口:only

##窗口大小调整
**横向调整**
- ctrl w + :行数增大
- ctrl w - :行数减小
- res(ize) num :行数固定为num行
- res(ize) + num :行数增大num行
- res(ize) - num :行数减小num行

**纵向调整**
- vertical res(ize) num :列数固定为num行
- vertical res(ize) + num :列数增大num行
- vertical res(ize) - num :列数减小num行

##切换窗口
- ctrl + w 两下:自动切下一个
- ctrl + w :进入窗口切换模式，使用hjkl或者上下左右切换
- ctrl + b : 调到最下边的窗口
- ctrk + t : 调到最上边的窗口

##移动窗口
- ctrl + w + 上下左右/hjkl

##多文件
- n : 调到下一个要编辑的文件
- e# : 回到刚才编辑的文件

##文件浏览
- Ex : 开启目录浏览器
- Sex : 水平开启目录浏览器


##参考
[高效编辑器vim之窗口分割](http://blog.csdn.net/shallnet/article/details/14519771)
[vim多窗口使用技巧](http://blog.csdn.net/devil_2009/article/details/7006113)

