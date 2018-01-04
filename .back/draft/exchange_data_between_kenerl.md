#内核态和用户态通信
##引言
> 因为内核态和用户态使用不同的地址空间，故内核态和用户态通信需要一些专门的通信手段
> 

##内核启动参数
###简介
Linux 提供了一种通过 bootloader 向其传输启动参数的功能，内核开发者可以通过这种方式来向内核传输数据，从而控制内核启动行为。

通常的使用方式是，定义一个分析参数的函数，而后使用内核提供的宏 __setup把它注册到内核中，该宏定义在 linux/init.h 中，因此要使用它必须包含该头文件.
###用法
```c
__setup("para_name=", parse_func)
//para_name 为参数名
//parse_func 为分析参数值的函数
```
使用grub进行相关配置进行解析
###实现
```c
#define __setup_param(str, unique_id, fn, early)            /
    static char __setup_str_##unique_id[] __initdata = str;    /
    static struct obs_kernel_param __setup_##unique_id    /
        __attribute_used__                /
        __attribute__((__section__(".init.setup")))    /
        __attribute__((aligned((sizeof(long)))))    /
        = { __setup_str_##unique_id, fn, early }
        
#define __setup(str, fn)                    /
    __setup_param(str, fn, fn, 0)
    
//举例
__setup("foo=" , foo);
//展开后
static char __setup_str_foo[] __initdata = "foo=";    
static struct obs_kernel_param __setup_foo    
        __attribute_used__                
        __attribute__((__section__(".init.setup")))    
        __attribute__((aligned((sizeof(long)))))    
        = { __setup_str_foo, foo, 0 };//"foo=",foo,0
        
//也就是说,启动参数(函数指针)被封装到obs_kernel_param结构中,
//所有的内核启动参数形成内核映像.init.setup段中的一个

```
###扩展
```c
//除了__setup_param还有early_param同样用于解析参数
//不同点在于early_param会有限调用

#define early_param(str, fn)                    /
    __setup_param(str, fn, fn, 1)

//对于early_param的调用点在parse_early_param
static int __init do_early_param(char *param, char *val)
{
    struct obs_kernel_param *p;
    for (p = __setup_start; p < __setup_end; p++) {
        if (p->early && strcmp(param, p->str) == 0) {
            if (p->setup_func(val) != 0)
                printk(KERN_WARNING
                       "Malformed early option '%s'/n", param);
        }
    }
    /* We accept everything at this stage. */
    return 0;
}
//对于__setup_param会在以下函数调用
static int __init obsolete_checksetup(char *line)
{
    struct obs_kernel_param *p;
    int had_early_param = 0;
    p = __setup_start;
    do {
        int n = strlen(p->str);
        if (!strncmp(line, p->str, n)) {
            if (p->early) {
                /* Already done in parse_early_param?
                 * (Needs exact match on param part).
                 * Keep iterating, as we can have early
                 * params and __setups of same names 8( */
                if (line[n] == '/0' || line[n] == '=')
                    had_early_param = 1;
            } else if (!p->setup_func) {
                printk(KERN_WARNING "Parameter %s is obsolete,"
                       " ignored/n", p->str);
                return 1;
            } else if (p->setup_func(line + n))//调用支持函数
                return 1;
        }
        p++;
    } while (p < __setup_end);
    return had_early_param;
}

```
##sysfs
###简介
> sysfs is a ram-based filesystem initially based on ramfs. It provides a means to export kernel data structures, their attributes, and the linkages between them to userspace.

**sysfs与sysctl的区别**

> - sysctl 是内核的一些控制参数，其目的是方便用户对内核的行为进行控制
> - sysfs 仅仅是把内核的 kobject 对象的层次关系与属性开放给用户查看
> 
> 因此 sysfs 的绝大部分是只读的，模块作为一个 kobject 也被出口到 sysfs，模块参数则是作为模块属性出口的.然后用户就可以通过sysfs 来查看和设置模块参数，从而使得用户能在模块运行时控制模块行为。

