#shell中的符号
##`&`与操作符
###`&`作用
- & 表示后台执行
- && 表示命令串行执行 例如`date&&ls&&date`，可以查看ls的执行时间

##`|`或操作符
###`|`作用
- | 作为管道命令符，将上一个命令的stdout作为下一个命令的stdin
- |&              将一个标准错误通过管道 输送 到另一个命令作为输入



##`>`比较
###`>`作用
- \> 输出重定向到一个文件或设备 覆盖原来的文件
- \>               输出重定向到一个文件或设备 覆盖原来的文件
- \>!           输出重定向到一个文件或设备 强制覆盖原来的文件
- \>>             输出重定向到一个文件或设备 追加原来的文件
- 2>             将一个标准错误输出重定向到一个文件或设备 覆盖原来的文件 
- 2>>           将一个标准错误输出重定向到一个文件或设备 追加到原来的文件
- 2>&1         将一个标准错误输出重定向到标准输出 注释:1 可能就是代表 标准输出
