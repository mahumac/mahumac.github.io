---
title: BGP中的AS Path Prepending
date: 2023-11-24 +0800
categories: [网络, BGP]
pin: false
math: true
tags: [网络, BGP] 
---

**[AS Path](https://www.internetworks.in/2019/03/bgp-as-path-prepending.html)** 是一个 BGP wellkown-mandatory  属性 。BGP 所做的只是根据 AS Path长度选择它认为最短的路径。（首选最短的 AS 路径）

与大多数其他属性不同，对 AS Path 的更改会传播到下游 AS。这使得使用 AS Path 可以实现 BGP Traffic Engineering

## 什么是 AS Path Prepending

AS Path Prepending（AS路径预置）是一种BGP路由策略技术，通过在BGP路由的AS路径属性中**重复添加自身AS号码**，人为增加路径长度，从而影响其他自治系统（AS）的路由选择决策。由于BGP协议在选择最佳路径时优先考虑较短的AS路径，增加路径长度可降低该路由的优先级，使流量转向其他更优路径。

具体来说：

- **工作原理**：在向外通告路由时，发送方主动在AS路径中多次追加自己的AS号。例如，原本路径为AS100→AS200，若AS100执行预挂（如重复3次），路径将变为AS100→AS100→AS100→AS200。
- **主要用途**：
  - **流量引导**：在多宿主网络（连接多个ISP）中，通过延长特定方向的AS路径，将入站流量引导至其他连接点。
  - **负载均衡**：在多条链路上分配流量，避免单一链路拥塞。
  - **控制返回路径**：确保返回流量与出站流量路径一致，避免非对称路由问题。
- **配置要点**：
  - 通常需要在边界路由器上配置策略，明确 prepending 的AS号及重复次数。
  - 需验证接收方路由器的处理逻辑，避免因路径过长被错误过滤。
- **优缺点**：
  - **优点**：配置简单、无需与其他AS协商，适用于快速调整流量。
  - **风险**：过度预置可能导致路由被忽略；部分网络可能过滤异常长的AS路径，影响可达性。

实际应用中，AS Path Prepending被广泛部署，尽管存在争议，但其灵活性和即时生效的特点使其成为网络工程师常用的基础流量管理工具之一。

## 配置举例

配置步骤：

1. **创建IP前缀列表**：定义需要调整的目标路由（如192.168.1.0/24）。
2. **创建路由策略**：
   - 匹配步骤1中的前缀。
   - 应用AS Path Prepending（多次添加自身AS号）。
3. **将策略应用到BGP邻居的出方向**：使邻居接收到的路由AS路径被修改。
4. **验证配置**：检查BGP表，确认路径长度变化及选路结果。



**关键细节**：

- **方向选择**：若需让本地路由器（如AR4）避开来自AR5的路由，通常在AR5的出方向（或AR4的入方向）应用策略，增加AS路径长度。
- **预挂次数**：一般重复添加3-4次即可显著影响选路，但需避免过度导致路由被过滤。
- **厂商差异**：华为设备使用apply as-path，思科设备通过set as-path prepend实现类似功能。



需要注意不同厂商设备命令可能不同，但逻辑步骤类似。



华为 路由器上，实现此目的的配置如下：

```
#
ip ip-prefix 192.168.1.0 index 10 permit 192.168.1.0 24

#
route-policy rp_AS-Path-Prepend permit node 20
    if-match ip-prefix 192.168.1.0                # 匹配目标路由
    apply as-path 65001 65001 65001 additive      # 重复添加自身AS号（如AS100添加三次：100 100 100）

#
bgp 65001
    peer <邻居IP> route-policy rp_AS-Path-Prepend export   # 出方向修改发送的路由
```



Cisco 路由器上，实现此目的的配置如下：

```
router bgp 65001
neighbor 198.51.100.90 remote-as 65002
neighbor 198.51.100.90 description IX peer
neighbor 198.51.100.90 route-map prepend out
!
route-map prepend permit 10
set as-path prepend 65001 65001 65001   << 追加3次AS号
!
```



Juniper路由器上，配置如下：

```bash
set interfaces xe-0/0/0:1 unit 0 family inet address 10.1.23.1/24
set interfaces xe-0/0/0:0 unit 0 family inet address 172.16.0.1/24
set interfaces xe-0/0/0:0 unit 0 family inet address 10.200.1.1/24
set interfaces lo0 unit 0 family inet address 192.168.0.1/32

set policy-options policy-statement ps_AS-Prepend-01 term prependterm1 from protocol direct
set policy-options policy-statement ps_AS-Prepend-01 term prependterm1 from route-filter 172.16.0.0/16 orlonger
set policy-options policy-statement ps_AS-Prepend-01 term prependterm1 from route-filter 192.168.0.0/24 orlonger
set policy-options policy-statement ps_AS-Prepend-01 term prependterm1 then as-path-prepend "65001 65001 65001"
set policy-options policy-statement ps_AS-Prepend-01 term prependterm1 then accept
set policy-options policy-statement ps_AS-Prepend-01 term else from protocol direct
set policy-options policy-statement ps_AS-Prepend-01 term else from route-filter 10.200.0.0/16 orlonger
set policy-options policy-statement ps_AS-Prepend-01 term else then accept

set routing-options autonomous-system 65001
set routing-options router-id 192.168.0.1

set protocols bgp group ebgp type external
set protocols bgp group ebgp export ps_AS-Prepend-01
set protocols bgp group ebgp peer-as 65000
set protocols bgp group ebgp neighbor 10.1.23.2
```

查看结果：

```bash
user@R1#[edit]
user@R1# show policy-options 
policy-statement prependpolicy1 {
    term prependterm1 {
        from {
            protocol direct;
            route-filter 172.16.0.0/16 orlonger;
            route-filter 192.168.0.0/24 orlonger;
        }
        then {
            as-path-prepend "65001 65001 65001";
            accept;
        }
    }
    term else {
        from {
            protocol direct;
            route-filter 10.200.0.0/16 orlonger;
        }
        then accept;
    }
}

[edit]
user@R1# show protocols bgp 
group ebgp {
    type external;
    export direct;
    peer-as 65000;
    neighbor 10.1.23.2;
}

[edit]
user@R1# show routing-options 
autonomous-system 65001;
router-id 192.168.0.1

```

## AS-Path 防环

AS Path Prepending是通过在AS路径中重复添加自己的AS号，使得路径看起来更长，从而影响路由选择，主要用来控制流量方向，比如负载均衡或者引导流量走特定路径。

AS环路通常是通过eBGP的防环机制来处理的，比如检查AS路径中是否包含自己的AS号，如果包含则丢弃路由，防止环路。而AS Path Prepending是主动添加自己的AS号，这可能会影响这个机制。

AS Path Prepending在出方向添加自己的AS号，当路由被发出去后，如果再次传回来，会检查到自己的AS号在路径中，从而拒绝接收，这样就避免了环路。比如，假设AS100向AS200发送路由，AS路径被预挂为100 100 100。如果这个路由因为某种原因又被传回AS100，AS100在收到时会检查到自己的AS号在路径里，就会丢弃，防止环路。

**因为AS Path Prepending通常是 '出方向' 添加AS号，而防环机制是 '入方向' 检查。**



## Prepending 的风险

通常，AS AS Path Prepending 不会太大风险。

但是，当网络执行过多级别的Prepending 时，“恶意”AS 可以通过在 AS Path 中对预置的 AS 执行本地剥离来设置意外结果，并将缩短的 AS 路径传播到其邻居。该路径看起来完全合理，并且下游 AS 看不到前置的剥离，结果可能是流量重定向。

