#路由与URPF调试记录
##背景说明
###机器相关
|机器名称|作用|公网IP|内网IP|系统|
|--------|----|------|------|----|
|本机 |工作机|192.168.62.42| NULL|OSX|
|51|装有LVS，主要测试|192.168.60.51|192.168.10.51|centos 6.8|
|50|装有LVS，辅助测试|192.168.60.50|192.168.10.50|centos 6.8|
|48|装有Nginx,作为RS|192.168.60.48|192.168.10.48|centos 6.5|
###其他相关
62段与60段可以通过路由相通，10段与62段相隔离
LVS的VIP为60段,通过10段与RS相通

###问题背景
最近在对移植后的LVS做一些测试，这次测试的就是NAT模式下,LVS是否正常。

##问题说明
###引发问题
因为NAT模式需要更改RS的路由表，因为测试机上还跑了其他的服务，故在48上仅增加了到本机的路由:
```
route add 192.168.62.42 gw 192.168.10.51
```
此时忽然ssh链接断掉，第一反应是51上的`ip_forward`没有打开，经过检查之后发现，51上的`ip_forward`打开
后依然不起作用，在本机上`ping 192.168.60.48`,并在192.168.60.48上进行抓包，查看到192.168.60.48上能够
收到本机发来的ICMP包，但是十分诡异的是并没有发现replay的包，删除路由规则之后，可以看到ICMP回包，说明
48上的ICMP是没有问题的，那是怎么回事呢？
###定位问题
这时候只好请教老大了，老大看了一下之后说这是`rp_filter`的问题，将`rp_filter`置为0即可，但是，依然不起作用，
老大在各种测试中，我也帮不上什么忙，干脆去百度一下，`rp_filter`到底是干啥的？这时候发现了一个词`URPF`，百度
之后如下:
```
通常情况下，路由器收到数据报文后，获取到数据包中的目的IP地址，针对目的IP地址查找本地路由转发表，如果有对应转发表项则转发数据报文；
否则，将报文丢弃。由此看来，路由器转发报文时，并不关心数据包的源地址。这就给源地址欺骗攻击有了可乘之机。
源地址欺骗攻击为入侵者构造出一系列带有伪造源地址的报文，频繁访问目的地址所在设备或者主机；即使响应报文不能到达攻击者，
也会对被攻击对象造成一定程度的破坏。URPF（Unicast Reverse Path Forwarding，单播逆向路径转发）的主要功能是用于防止基于源地址欺骗的网络攻击行为。
路由器接口一旦使能URPF功能，当该接口收到数据报文时，首先会对数据报文的源地址进行合法性检查，对于源地址合法性检查通过的报文，才会进一步查找去往目的地址的转发表项，
进入报文转发流程；否则，将丢弃报文。
```
什么意思这里讲的不是很清楚，看URPF的两个类型就很容易理解了
```
严格型URPF：不但要求路由器转发表中，存在去往报文源地址的路由；而且还要求报文的入接口与转发表中去往源地址路由的出接口一致，只有同时满足上述两个条件的报文，
才被认为是合法报文。在一些特殊情况下（如存在非对称路径），严格型检查会错误的丢弃非攻击报文。

松散型URPF：仅要求路由器的转发表中，存在去往报文的源地址路由即可，不再检查报文的入接口与转发表中去往源地址的路由的出接口是否一致。
当用户网络中存在非对称路径等无法保证报文入接口和设备去往源地址路由出接口一致的情况下，可以配置松散型URPF检查。
```
拿我们这里的情况距离来说，本机ping 48这台机器，是通过外网网卡输入的，对于严格型的URPF来说，就需要通过外网的网卡出去才行，否则的话，这个replay包就会被丢弃，
因为我们添加了路由规则，replay的下一跳10段，也就是通过内网网卡输出，这与进来的网卡不一致，故丢弃了该包，这就是我们抓不到包的原因。那么为什么我们置为0依然不行呢？
这是我尝试的将`rp_fitler`置为2，`rp_fitler`各字段含义如下(不同版本有不同含义，注意!)：
```
rp_filter - INTEGER
        0 - No source validation.
        1 - Strict mode as defined in RFC3704 Strict Reverse Path
            Each incoming packet is tested against the FIB and if the interface
            is not the best reverse path the packet check will fail.
            By default failed packets are discarded.
        2 - Loose mode as defined in RFC3704 Loose Reverse Path
            Each incoming packet's source address is also tested against the FIB
            and if the source address is not reachable via any interface
            the packet check will fail.

        Current recommended practice in RFC3704 is to enable strict mode
        to prevent IP spoofing from DDos attacks. If using asymmetric routing
        or other complicated routing, then loose mode is recommended.

        The max value from conf/{all,interface}/rp_filter is used
        when doing source validation on the {interface}.

        Default value is 0. Note that some distributions enable it
        in startup scripts.
```
###一波三折
尝试一下，居然可以了，我告诉老大，老大验证了一下，的确可以了，那么我们把`rp_filter`改回0呢，看是否真的是这个字段引发的问题,改回之后发现依然可以ping通，
，改为1呢?还是可以ping通，这就有点奇怪了？可是查看记录，并没有更改其他字段，到底是什么原因呢？老大想了一会，应该是路由cache的问题,刷新路由cache
```
ip route flush cache
```
果然，ping不通了，原因是，当一次查找路由成功后，该次路由路径就会被加入路由cache中，在查询的时候，首先会查路由cache，cache是在RUPF检查之前的，所以无所
我们怎么改都可以ping的通。
那么我们回到之前的问题，为什么该为0就不生效呢？

###最终解决
最终这个问题，我们没能解决，我们再查阅Linux协议栈源码后发现，在该版本中，`rp_filter`的0与1，在`fib_validate_source`中所扮演的作用是一样的，而内核中使用
`ip_fitler`这个字段的也就这一处。限于时间问题，并没有深究。

##结尾
在测试LVS以来，最大的感慨是，协议栈真的很复杂，就算抛去那些奇奇怪怪的协议，淡淡讨论IP,TCP,我们也仅仅了解三步握手的而已，想要了解TCP的重传，滑动窗口，定时器，
IP的分片，ICMP的各种使用，arp协议如何发挥作用，我还有很长的路需要走。
