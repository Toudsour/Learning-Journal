#Linux内核符号---system.map
##简介
对于我们普通的程序而言，我们再编译程序的过程中，有经验的同学都知道会生成一个函数符号表的东西，用于在程序编译最后定位函数位置，那么在内核开发的时候同样有这么一个东西，也就是我们再编译内核时生成的System.map-xxx了。

##用途
那么这个东西有什么用？与编译普通程序生成的符号表一样，他也是用来表示内核中函数所在的内存位置的，但是有同学可能会问，普通程序在编译后，符号表便可以被剔除掉，那么内核的符号表存在的意义又是什么呢？这和普通程序也是一样的，普通程序在调试的时候，符号表是一定要留着的，那么system.map也是同样的作用，他不是给内核自己使用的，他是提供给klogd在记录内核发生panic等错误时进行名字-地址之间的解析用的，当然还有其他一些程序会使用system.map。

那么修改system.map会影响内核么？答案自然是不会，我就干过这样的事，又一次因为修改了内核中一个const常量引发了panic，我就想着修改system.map是否能够修复这个bug，结果显然是没用的，system.map只是给其他程序使用的，内核中有自己的一份函数符号表，存放于`/proc/vmstat`。

##格式
虽然system.map对于内核没什么用，但是能够读懂system.map能够有利于定位一些问题，这是我机器上的一份system.map :

```py
0000000000000000 A VDSO32_PRELINK
0000000000000000 D __per_cpu_start
0000000000000000 D per_cpu__irq_stack_union
0000000000000000 A xen_irq_disable_direct_reloc
0000000000000000 A xen_save_fl_direct_reloc
0000000000000040 A VDSO32_vsyscall_eh_frame_size
00000000000001e7 A kexec_control_code_size
00000000000001f0 A VDSO32_NOTE_MASK
0000000000000400 A VDSO32_sigreturn
0000000000000410 A VDSO32_rt_sigreturn
0000000000000420 A VDSO32_vsyscall
0000000000000430 A VDSO32_SYSENTER_RETURN
0000000000004000 D per_cpu__gdt_page
0000000000005000 d per_cpu__exception_stacks
000000000000b000 d per_cpu__idt_desc
000000000000b010 d per_cpu__xen_cr0_value
000000000000b018 D per_cpu__xen_vcpu
000000000000b020 D per_cpu__xen_vcpu_info
000000000000b060 d per_cpu__mc_buffer
```
system.map分为三列，从左到右分别是符号地址，符号类型，符号名称，相当容易理解，其中符号类型定义如下:

 - A：表示全局函数或变量 例如: `extern void agp_frontend_cleanup(void);`
 - B：表示为初始化的变量  例如: ` atomic_t irq_err_count;`
 - b: 表示为为初始化的数组  例如: `static struct proc_dir_entry * root_irq_dir;`  
 - D: 已经初始化过的变量 例如: `int kstack_depth_to_print = 24;`    
 - d: 表示已经初始化的数组 例如: ``
 - R: 表示常量数组 例如: `extern const ide_pio_timings_t ide_pio_timings[6];`
 - r: 表示常量变量 例如： `static const int usb_bandwidth_option = ..;`
 - T: 符号是代码区中的符号 例如: `kmem_cache_t *kmem_cache_create(...)`
 - ？: 符号的类型未知，或者与具体文件格式有关

##参考
- [System.map](http://blog.csdn.net/luckywang1103/article/details/50401287)
- [System.map解析](http://blog.csdn.net/tommy_wxie/article/details/8039695)