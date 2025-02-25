---
title: Wireshark小技巧 | TCP 会话完整性分析
date: 2022-11-21 +0800
categories: [网络, Wireshark]
pin: false
math: true
tags: [Wireshark, 抓包]
---



## TCP Conversation Completeness 会话完整性

来自于 Wireshark 新版本 3.6.0 的功能说明，详见 ：[Wireshark 3.6.0 Released](https://www.wireshark.org/news/20211122.html) 摘引如下：

> TCP conversations now support a completeness criteria, which facilitates the identification of TCP streams having any of opening or closing handshakes, a payload, in any combination. It can be accessed with the new tcp.completeness filter.

TCP 会话目前支持完整性策略，对于分析 TCP 流有着很方便的帮助作用。可以通过 **tcp.completeness** 显示过滤器表达式进行相关过滤。 

**什么是 TCP 会话完整性 ？**TCP Conversation Completeness ，理论上一个完整的 TCP 会话应该同时包含最常理解的打开握手和关闭握手，而并不依赖于有或者没有任何数据传输。 

但考虑到实际大多数 TCP 会话场景，包含数据更多才像是一次完整会话，因此可以通过以下 **tcp.completeness** 字段来构建会话过滤值：

- 1 : SYN
- 2 : SYN-ACK
- 4 : ACK
- 8 : DATA
- 16 : FIN
- 32 : RST

以上字段值可以灵活组合，最终形成不同的 **TCP会话完整性** 显示过滤表达式。 

## 举例

### TCP 三次握手

如果想在数据包文件中过滤出一个仅包含标准 TCP 三次握手的会话，那么可以使用表达式： `'tcp.completeness == 7'` ，因为 1 (SYN) + 2 (SYN/ACK) + 4 (ACK) = 7 。

**需要特别注意的是**，表达式`tcp.completeness == 7`只能过滤出仅有 TCP 三次握手的数据包，所以如果在某条 TCP 流中包含有任何非 TCP 三次握手的数据包时，该表达式是不生效，即无法过滤出该条 TCP 流。 （ 该 TCP 流包含 TCP 三次握手、数据、TCP 四次挥手完整数据包。）



### TCP 四次挥手

同样，如果想在数据文件中过滤出一个仅包含标准 TCP 四次挥手的会话，那么可以使用表达式 `'tcp.completeness == 20'` ，因为 4 (ACK) + 16 (FIN)= 20 。

**同样需要特别注意的是**，这里为什么使用表达式 **`'tcp.completeness==20'`** ，而不是使用表达式 **`'tcp.completeness==16'`** ，这也是在标准 TCP 四次挥手上理论和实际上的有所不同。



### TCP RST 关闭握手

在 TCP 会话关闭连接的方式，可以通过 FIN ，也可以通过 RST 。简单以一个连接对端未开通端口的示例，如下

同样此处使用的表达式是 **`'tcp.completeness==37'`** ，1 (SYN) + 4 (ACK) + 32 (RST)= 37 。



### TCP 完整会话

最后回到真正的 TCP 会话案例，一个完整的带有数据传输的会话将通过一个更多组合的表达式被过滤，考虑到关闭连接可以与 FIN 或 RST 数据包关联，甚至说两者都关联的情况，最完整的过滤表达式为：

```c
tcp.completeness == 31 or tcp.completeness == 47 or tcp.completeness == 63
```

因为关闭连接的数据包可能会包含 与 FIN 或 RST 标识位，或者同时包含2者。

考虑到展示方便，以下分别单独就 3 个表达式做简单示例。 

- 最标准的 TCP 完整会话， `'tcp.completeness==31'` ，因为 1 (SYN) + 2 (SYN/ACK) + 4 (ACK) + 8 (DATA) + 16 (FIN) = 31 。

- 以 RST 方式关闭连接的 TCP 完整会话， `'tcp.completeness==47'` ，因为 1 (SYN) + 2 (SYN/ACK) + 4 (ACK) + 8 (DATA) + 32 (RST) = 47 。

- FIN + RST 同时存在的 TCP 完整会话， `'tcp.completeness==63' `，因为 1 (SYN) + 2 (SYN/ACK) + 4 (ACK) + 8 (DATA) + 16 (FIN)+ 32 (RST) = 63 。



### TCP 会话特定值

过滤特定TCP对话值的另一种方法是按单个 标志、summary 字段或它们的组合。 例如：

- 过滤 TCP  'Complete, WITH_DATA'  会话

  `'(tcp.completeness.fin==1 || tcp.completeness.rst==1) && tcp.completeness.str contains "DASS"'`

- 过滤 TCP  'Complete, NO_DATA'  会话

  `'(tcp.completeness.fin==1 || tcp.completeness.rst==1) && tcp.completeness.data==0 && tcp.completeness.str contains "ASS"' `



 **参考：**

- [Wireshark 提示和技巧 - TCP 会话完整性分析 - 知乎](https://zhuanlan.zhihu.com/p/460755199)
- [Wireshark Docs 7.5. TCP Analysis ](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvTCPAnalysis.html)