#LVS学习

##LVS与Nginx的不同
- LVS处于内核态,Nginx处于用户态，故LVS拥有更小的开销
- Nginx工作与应用层，而LVS工作在传输层
- LVS能够支持承载与IP协议上的任何协议，而Nginx只支持HTTP协议

##如何使用LVS
- LVS+Keepalived
- LVS负责负载均衡，Keepalived负责路由选择

##LVS的负载均衡方法
###VS/NAT方法
- VS/DNAT----通过更改IP帧中的目的地址实现负载均衡
- VS/FULLNAT----更改IP帧中的目标地址，并将原地址更改为LVS服务器地址，此时将源IP的地址写入TCP的opt中，并在RealServer中安装插件，在IP报给上层TCP时进行转换

###VS/DR
- 虚拟一个VIP绑定在LVS机器上，并在RealSever上将VIP绑定在Non-ARP设备上
- 当客户端请求VIP的时候会被LVS接收，LVS将IP的Mac地址改为RealServer集群中的一台，并进行转发，RealServer接收到帧后检查VIP在本地网络设备上后，就会进行处理，并将并响应发送到客户端上

###VS/IPTU
- 将IP报文封装在另一个IP报文的方法发送


##Keepalived
- Keepalived的功能覆盖了ipvsadm的功能，Keepalived能够通过配置文件的方式对LVS进行配置，包括绑定VIP,设置RealSever服务器，负载均衡方式，轮训算法等等
- Keepalived还能到做到主从备份，当Master的LVS挂掉了之后，BackUp的LVS能够自动启用，Master和BackUp通过VRRP协议进行沟通
- Keepalived有三种方式监控Real Sever，分别通过ICMP，TCP,HTTP的方式

##LVS实战
###VS/FULL NAT
**TIPS:公司的Keepalive有些不同，注意。**

- 安装keepalived，并写好配置文件，注意要方式选择NAT，作为FULL NAT还要在内核中设置相关参数，除此之外，需要设置IP池，因为LVS时与后端通信的时候要占用一个端口，一个TCP PORT只能和一个Real Sever连接
- 后端启动服务，并绑定VIP，并将VIP的arp_ignore置为1，arp_announce置为2

###VS/DR
- 相当简单，如果使用keepalived只需修改配置文件即可
- 对于Real Server与FULL NAT也是相同的

##keepalived配置文件
**Keepalived官网中有用户手册，可以参考上边的内容**

```bash
#全局定义模块
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc #邮件报警，可以不设置，后期nagios统一监控。
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL  #此处注意router_id为负载均衡标识，在局域网内应该是唯一的。
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
  ++++++++++++++++我是分隔符++++++++++++++++++++++

 #VRRP实例定义块

vrrp_instance VI_1 {
    state MASTER #状态只有MASTER和BACKUP两种，并且要大写，MASTER为工作状态，BACKUP是备用状态。
    interface eth0
        lvs_sync_daemon_inteface eth0  #这个默认没有，相当于心跳线接口，DR模式用的和上面的接口一样，也可以用机器上的其他网卡eth1，用来防止脑裂。
    virtual_router_id 51 #虚拟路由标识，同一个vrrp_instance的MASTER和BACKUP的vitrual_router_id 是一致的。
    priority 100  #优先级，同一个vrrp_instance的MASTER优先级必须比BACKUP高。
    advert_int 1 #MASTER 与BACKUP 负载均衡器之间同步检查的时间间隔，单位为秒。
    authentication {
        auth_type PASS  #验证authentication。包含验证类型和验证密码。类型主要有PASS、AH 两种，通常使用的类型为PASS，\
        auth_pass 1111   据说AH 使用时有问题。验证密码为明文，同一vrrp 实例MASTER 与BACKUP 使用相同的密码才能正常通信。
    }
    virtual_ipaddress { #虚拟ip地址,可以有多个地址，每个地址占一行，不需要子网掩码，同时这个ip 必须与我们在lvs 客户端设定的vip 相一致！
        192.168.200.100
        192.168.200.101
        192.168.200.102
    }
}
  ++++++++++++++++我是分隔符++++++++++++++++++++++
 
 #虚拟服务器定义块

virtual_server 192.168.200.100 443 {   #虚拟IP，来源与上面的虚拟IP地址，后面加空格加端口号
    delay_loop 6  #健康检查间隔，单位为秒
    lb_algo rr    #负载均衡调度算法，一般用wrr、rr、wlc
    lb_kind NAT   #负载均衡转发规则。一般包括DR,NAT,TUN 3种。
    persistence_timeout 50 #会话保持时间，会话保持，就是把用户请求转发给同一个服务器，不然刚在1上提交完帐号密码，就跳转到另一台服务器2上了。
    protocol TCP  #转发协议，有TCP和UDP两种，一般用TCP，没用过UDP。

    real_server 192.168.201.100 80 { #真实服务器，包括IP和端口号
        weight 1  #权重，数值越大，权重越高 
        TCP_CHECK {  #通过tcpcheck判断RealServer的健康状态
            connect_timeout 3 #连接超时时间
            nb_get_retry 3 #重连次数
            delay_before_retry 3 #重连时间间隔
            connect_port 80  #检测端口
        }
    }
}
```
