#Shell 补充
##echo 输出
    - echo输出默认换行
    - 如果想要不换行使用`echo -e "xxxx\c"`
    - 要是需要多此换行使用 `echo -e "xxxxx\n"`
##shell 循环
###类C语言
```
for ((i=0; i<100; i++))
do
    echo $i
done
```

###for in方法
```
#注意这种方法没法控制变量
for i in {1..10}
do
    echo $i
done
```
###for seq方法
```
for i in `seq 1 10`
do
    echo $i
done
```
##shell 数组
###获取数组长度
```
    array_len=${array_list[*]}
#或者
    array_len=${array_list[@]}
```
###删除操作
```
#删除单个
    unset array[1]
#清空数组
    unset array
```
###分片访问
```
#注意对于不同iterm，支持的下标不同
#zsh支持负数，bash只支持正数
    array[*]:start_index:end_index
```
###模式匹配
```
#匹配替换    
    ${array[@]/x/y}    # 最小匹配替换，每个元素只替换一次
    ${array[@]//x/y}   # 最大匹配替换，每个元素可替换多次
    ${array[@]/#x/y}   # 从左往右匹配替换，只替换每个元素最左边的字符
    ${array[@]/%x/y}   # 从右往左匹配替换，只替换每个元素最右边的字符
#匹配删除
    ${arrat[@]#a*b}    # 每个元素,从左向右进行最短匹配
	${arrat[@]##a*b}   # 每个元素,从左向右进行最长匹配
    ${arrat[@]%a*b}    # 每个元素,从右向左进行最短匹配
    ${arrat[@]%a*b}    # 每个元素,从右向左进行最长匹配
```

##shell数值计算
###expr表达式
```
RG1 | ARG2          如果没有参数是null或0，返回ARG1；否则返回ARG2 
ARG1 & ARG2         如果没有参数是null或0，返回ARG1；否则返回0 
ARG1 < ARG2         如果ARG1小于ARG2，返回1；否则返回0 
ARG1 <= ARG2        如果ARG1小于或等于ARG2，返回1；否则返回0 
ARG1 = ARG2         如果ARG1等于ARG2，返回1；否则返回0 
ARG1 != ARG2        如果ARG1不等于ARG2，返回1；否则返回0 
ARG1 >= ARG2        如果ARG1大于或等于ARG2，返回1；否则返回0 
ARG1 > ARG2         如果ARG1大于ARG2，返回1；否则返回0 
ARG1 + ARG2         返回ARG1和ARG2的算术运算和 
ARG1 - ARG2         返回ARG1和ARG2的算术运算差 
ARG1 * ARG2         返回ARG1和ARG2的算术乘积 
ARG1 / ARG2         返回ARG1和ARG2的算术商 
ARG1 % ARG2         返回ARG1和ARG2的算术余数 
STR : EXP           如果EXP匹配到STR的某个模式，返回该模式匹配 
match STR EXP       如果EXP匹配到STR的某个模式，返回该模式匹配 
substr STR POS LEN  返回起始位置为POS（从1开始计数）、长度为LEN个字符的字符串 
index STR CHARS     返回在STR中找到CHARS字符串的位置；否则，返回0 
length STR          返回字符串STR的数值长度 
+ TOKEN             将TOKEN解释成字符串，即使是个关键字 
(EXPRESSION)        返回EXPRESSION的值 
```

###中括号
```
#注意shell会将结果直接进行替换
echo $[1 + 5]
```

###bc
```
var1=10.46 
var2=43.67 
var3=33.2 
var4=71

var5=`bc << EOF 
scale = 4 
a1 = ($var1 * $var2) 
    b1 = ($var3 * $var4) 
    a1 + b1 
    EOF 
```
##shell数组作为函数参数
###修改数组
```
function f()
{
    local opts=($@)
    local len=$[$#-1]
    local args_more=${opts[$len]}
    local args_array=${opts[*]:0:$len}

}
```
###传递名称
```
#! /bin/bash
function f() {
    name=$1[@]
    b=$2
    a=("${!name}")

    for i in "${a[@]}" ; do
        echo "$i"
    done
    echo "b: $b"
}

x=("one two" "LAST")
b='even more'

f x "$b"

```
##
