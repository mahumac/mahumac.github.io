---
title: uRPF(Unicast Reverse Path Forwarding)单播反向路径转发
date: 2023-11-24 +0800
last_modified_at: 2025-10-13 +0800
categories: [网络]
pin: false
math: true
tags: [网络, uRPF] 
---

# uRPF(Unicast Reverse Path Forwarding)

## 概念和原理


这是网络设备的一个安全特性，主要功能是用于防止基于源地址欺骗的网络攻击行为，说简单一点就是在IP数据包转发的时候不单基于目的地址查看路由表，对源地址同样进行查表，如未能查到路由（一般不是默认路由，但根据策略不同行为稍有区别），则不对数据包进行转发；从路由角度讲就是不仅要查找下一跳也对回包预先查找路由。

和uRPF最相关的一个路由术语是“非对称路由 (Asymmetric Routing)”也有翻译成“异步路由”的。

非对称路由指的是数据包往返的路径不一致，其实在现在网络中非对称路由非常常见，常规路由技术是无状态的只针对目的地址查表转发，网络设备配置不当会出现非对称路由甚至路由环路。

但是对于防火墙这种注重安全的网络设备可能会对数据包进行追踪（即跟踪会话状态），此时如果出现非对称路由导致数据包只有单侧经过防火墙则会导致通信异常。

- 对称路由 
    <img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161254138-2116740354.png" alt="img" style="zoom:50%;" />
    
- 非对称路由  
    <img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161316519-572557097.png" alt="img" style="zoom:50%;" />
    
- 非对称+防火墙  
    <img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161327754-1836053453.png" alt="img" style="zoom:50%;" />
    
- 伪造IP情况，Host3冒充Host2的IP地址给Host1发包，诱骗Host1回包给Host2，如果大量发起这种报文会使得Host2网络拥塞超载。Host1IP为Host2信任IP，也不会引起Host2管理员的注意，导致DoS攻击。  
    <img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161338693-938814879.png" alt="img" style="zoom:50%;" />  
    
    uRPF就是为了此种情况开发的特性，数据包转发时候同时对源ip和目的ip分别查路由表，再决定是否转发。当伪造的Host2 IP从eth1接口收到后，检查目的地址转发接口为eth3，但是检查源地址路由接口应该为eth2，与接收的接口eth1不符，与uRPF策略冲突 ---- 丢包。

## URPF处理流程

URPF检查根据检查的内容和策略不同，有严格（strict）型和松散（loose）型两种。有些设备还可以支持ACL与缺省路由的检查。

> URPF的处理流程如下：
> 
> (1)如果报文的源地址在路由器的FIB表中存在对于strict型检查，反向查找报文出接口，若其中至少有一个出接口和报文的入接口相匹配，则报文通过检查；否则报文将被拒绝。（反向查找是指查找以该报文源IP地址为目的IP地址的报文的出接口）；对于loose型检查，报文进行正常的转发。
> 
> (2)如果报文的源地址在路由器的FIB表中不存在，则检查缺省路由及URPF设置是否允许默认路由。
> 
> 对于配置了缺省路由，但未允许默认路由的情况，不管是strict型检查还是loose型检查，只要报文的源地址在路由器的FIB表中不存在，该报文都将被拒绝；
> 
> 对于配置了缺省路由，但允许默认路由的情况下，如果是strict型检查，只要缺省路由的出接口与报文的入接口一致，则报文将通过URPF的检查，正常转发；如果缺省路由的出接口和报文的入接口不一致，则拒绝转发。如果是loose型检查，报文都将通过URPF的检查，进行正常的转发。
> 
> (3)当且仅当报文被拒绝后，再去匹配ACL。如果被ACL允许通过，则报文继续进行正常的转发；如果被ACL拒绝，则报文被丢弃。（相当于是uRPF的白名单）

示例


基于下面拓扑使用H3C VSR1000，进行uRPF实验：  
![img](../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161406614-569502205.png)

```perl
#Host1
IP:192.168.1.1/24 GW:192.168.1.254

#Host2
IP:192.168.2.1/24 GW:192.168.2.254
```

```perl
#R1
interface GigabitEthernet1/0
 port link-mode route
 ip address 10.0.0.1 255.255.255.252

interface GigabitEthernet2/0
 port link-mode route
 ip address 192.168.1.254 255.255.255.0

interface GigabitEthernet8/0
 port link-mode route
 ip address 10.0.0.5 255.255.255.252
#
 ip route-static 192.168.2.0 24 10.0.0.2
```

```perl
#R2
interface GigabitEthernet1/0
 port link-mode route
 ip address 10.0.0.2 255.255.255.252

interface GigabitEthernet2/0
 port link-mode route
 ip address 192.168.2.254 255.255.255.0
#
 ip route-static 192.168.1.0 24 10.0.0.1
#

#Spoofer
interface LoopBack0
 description SpoofingIP
 ip address 192.168.2.1 255.255.255.255

interface GigabitEthernet1/0
 port link-mode route
 ip address 10.0.0.6 255.255.255.252
#
 ip route-static 192.168.1.1 32 10.0.0.5
```

```perl
# 仿冒 IP 测试
[Spoofer]ping -a  192.168.2.1 192.168.1.1
Ping 192.168.1.1 (192.168.1.1) from 192.168.2.1: 56 data bytes, press CTRL_C tok
Request time out
Request time out
Request time out
Request time out
Request time out

--- Ping statistics for 192.168.1.1 ---
5 packet(s) transmitted, 0 packet(s) received, 100.0% packet loss
```

**正常通信**  
<img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161531127-646668485.png" alt="img" style="zoom: 50%;" />  
**仿冒发包**  
<img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161542585-27315313.png" alt="img" style="zoom: 50%;" />  
**抓包**  
<img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161552561-1888748960.png" alt="img" style="zoom: 50%;" />  
**uRFP开启前后**

```x86asm
#R2 开启uRPF
ip urpf strict
```

<img src="../images/2023-06-04-uRPF(Unicast Reverse Path Forwarding单播反向路径转发).assets/1456291-20240410161422797-1474395548.png" alt="img" style="zoom: 50%;" /> 
图上显示仅从G8/0口收到了数据包并未进行转发。

参考：

[https://www.h3c.com/cn/d\_200805/605753\_30003\_0.htm](https://www.h3c.com/cn/d_200805/605753_30003_0.htm) 
[https://www.rfc-editor.org/rfc/rfc8704.html](https://www.rfc-editor.org/rfc/rfc8704.html)

