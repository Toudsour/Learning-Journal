##Netfilter框架
###Netfilter嵌入协议栈的方式
- 通过NF_HOOK钩子函数的方式嵌入
- 嵌入的地点主要有五处，代码和示意图如下:
![](http://img.blog.csdn.net/20150404151707144?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamFzb25jaGVuX2diZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

```c
//ip_rcv()函数中调用:
int ip_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt, struct net_device *orig_dev)
    return NF_HOOK(PF_INET, NF_INET_PRE_ROUTING, skb, dev, NULL, 
        ip_rcv_finish);

//NF_INET_LOCAL_OUT : 在ip包接收到本地后调用：
//ip_local_deliver()函数中的调用：
int ip_local_deliver(struct sk_buff *skb)
    return NF_HOOK(PF_INET, NF_INET_LOCAL_IN, skb, skb->dev, NULL, 
        ip_local_deliver_finish);

//NF_INET_FORWARD: 在对接收的sk_buff包完成路由分类判断是需要进行转发的包进行处理后调用。
//在ip_forward()函数中的调用:
int ip_forward(struct sk_buff *skb)
    return NF_HOOK(PF_INET, NF_INET_FORWARD, skb, skb->dev, rt->u.dst.dev,
               ip_forward_finish);
        
//在对自身发出的包进行处理调用。
// _ip_local_out()函数中的调用：
//调用关系：ip_build_and_send_pkt()->ip_local_out()->_ip_local_out()
int __ip_local_out(struct sk_buff *skb) 
    return nf_hook(PF_INET, NF_INET_LOCAL_OUT, skb, NULL, skb->dst->dev,
               dst_output);
    
//在IP栈成功接收sk_buff包后处理调用。
// ip_output()函数中的调用：
int ip_output(struct sk_buff *skb)
    return NF_HOOK_COND(PF_INET, NF_INET_POST_ROUTING, skb, NULL, dev,
                ip_finish_output,!(IPCB(skb)->flags & IPSKB_REROUTED));
```

###NK_HOOK的实现方式

```c
#define NF_HOOK(pf, hook, skb, indev, outdev, okfn) \
         NF_HOOK_THRESH(pf, hook, skb, indev, outdev, okfn, INT_MIN)

#define NF_HOOK_THRESH(pf, hook, skb, indev, outdev, okfn, thresh)                \
({int __ret;                                                                                \
if ((__ret=nf_hook_thresh(pf, hook, &(skb), indev, outdev, okfn, thresh, 1)) == 1)\
         __ret = (okfn)(skb);                                                       \
__ret;})

//参数意义如下:
//pf : 协议号
//hook : hook点序号
//pskb : 协议栈数据结构
//indev : 输入设备
//outdev : 输出设备
//okfn : hook处理完毕调用该函数回到ip协议处理流程
//thresh : 优先级，低于该优先级的不调用
//cond : 状态，cond==0不予调用

static inline int nf_hook_thresh(int pf, unsigned int hook,
                            struct sk_buff **pskb,
                            struct net_device *indev,
                            struct net_device *outdev,
                            int (*okfn)(struct sk_buff *), int thresh,
                            int cond)
{
	if (!cond) 
		return 1;
#ifndef CONFIG_NETFILTER_DEBUG
	if (list_empty(&nf_hooks[pf][hook]))
         return 1;
#endif
	return nf_hook_slow(pf, hook, pskb, indev, outdev, okfn, thresh);
}

```

这里我们可以看到，hook函数本质是委托给了`nf_hook_slow`函数，该函数实现如下:

```c

#define NF_DROP 0
#define NF_ACCEPT 1
#define NF_STOLEN 2
#define NF_QUEUE 3
#define NF_REPEAT 4
#define NF_STOP 5
#define NF_MAX_VERDICT NF_STOP

/* Returns 1 if okfn() needs to be executed by the caller,
 * -EPERM for NF_DROP, 0 otherwise. */
int nf_hook_slow(int pf, unsigned int hook, struct sk_buff *skb,
         struct net_device *indev,
         struct net_device *outdev,
         int (*okfn)(struct sk_buff *),
         int hook_thresh)
{
    struct list_head *elem;
    unsigned int verdict;
    int ret = 0;

#ifdef CONFIG_NET_NS
    struct net *net;

    net = indev == NULL ? dev_net(outdev) : dev_net(indev);
    if (net != &init_net)
        return 1;
#endif             

    /* We may already have this, but read-locks nest anyway */
    rcu_read_lock();

    //通过pf协议类型，和hook类型，找到挂载函数结构的链表头
    elem = &nf_hooks[pf][hook];    

next_hook:
    verdict = nf_iterate(&nf_hooks[pf][hook], skb, hook, indev,
                 outdev, &elem, okfn, hook_thresh);

    if (verdict == NF_ACCEPT || verdict == NF_STOP) { //若接收规则，或规则停止可以继续处理数据包
        ret = 1;
        goto unlock;
    } else if (verdict == NF_DROP) {  //若丢弃包，则直接丢弃
        kfree_skb(skb);
        ret = -EPERM;
    } else if ((verdict & NF_VERDICT_MASK) == NF_QUEUE) {    //检查是否有队列处理函数，若有调用之
        if (!nf_queue(skb, elem, pf, hook, indev, outdev, okfn,
                  verdict >> NF_VERDICT_BITS))
            goto next_hook;
    }
unlock:
    rcu_read_unlock();
    return ret;
}

```

可以看到，`nf_hook_slow`主要的内容就是对`nf_hooks`中挂载的函数进行遍历调用。`nf_hooks`相关的内容如下:

```c
enum nf_inet_hooks {
    NF_INET_PRE_ROUTING,
    NF_INET_LOCAL_IN,
    NF_INET_FORWARD,
    NF_INET_LOCAL_OUT,
    NF_INET_POST_ROUTING,
    NF_INET_NUMHOOKS
};

struct nf_hook_ops
{
    struct list_head list; 

    /* User fills in from here down. */
    nf_hookfn *hook;        //挂载函数
    struct module *owner;    //模块
    int pf;             //pf与hooknum一起索引到特定协议特定编号的挂载函数队列，用于索引nf_hooks
    int hooknum;
    /* Hooks are ordered in ascending priority. */
    int priority;         //priority决定在同一队列(pf与hooknum相同)的顺序，priority越小则排列越靠前。
};

struct list_head nf_hooks[NPROTO][NF_MAX_HOOKS] __read_mostly;

```

用图表示为:

![](http://blog.chinaunix.net/attachment/201204/4/23069658_1333533215tU7x.jpg)

###HOOK的挂载与注销
```c
//对挂载函数的注册是在nf_register_hook函数中完成的。
int nf_register_hook(struct nf_hook_ops *reg)
{
    struct nf_hook_ops *elem;
    int err;

    err = mutex_lock_interruptible(&nf_hook_mutex);
    if (err < 0)
        return err;

    //从指定pf和hooknum的nf_hooks队列遍历，按priority从小到大顺序，将reg插入相应位置。
    list_for_each_entry(elem, &nf_hooks[reg->pf][reg->hooknum], list) {
        if (reg->priority < elem->priority)    //要注册的reg要比存在的优先级数字小，放到前面
            break;
    }   
    list_add_rcu(&reg->list, elem->list.prev);
    mutex_unlock(&nf_hook_mutex);
    return 0;
}

//注销挂载函数
void nf_unregister_hook(struct nf_hook_ops *reg)
{
    mutex_lock(&nf_hook_mutex);
    list_del_rcu(&reg->list);        //把reg从对应的链表中删除
    mutex_unlock(&nf_hook_mutex);

    synchronize_net();
}
```
