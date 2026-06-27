title: 网络探测：使用 Prometheus 替代 Smokeping
date: 2023-11-07 +0800
last_modified_at: 2025-02-13 +0800
categories: [Prometheus, Smokeping]
pin: false
math: true
tags: [Prometheus, Smokeping] 

# 使用 Prometheus 替代 Smokeping

### Smokeping 的局限

- **架构封闭**：Smokeping 是一个独立的、专用的工具，其数据存储在 RRD 中，难以与其他监控数据（如服务器负载、业务吞吐量）进行关联分析（数据孤岛）。
- **难以扩展**：Smokeping也支持Master-Slave架构（支持分布式），当设备节点的大量增加，导致维护 Smokeping 的 `config` 文件变得非常吃力。
- **告警能力弱**：Smokeping 的告警配置相对刻板，而 Prometheus 的 **Alertmanager** 可以处理更复杂的告警逻辑（如抑制、分组、静默）。

**Smokeping 的核心价值在于那种包含“延迟分布（Min/Max/Avg/StdDev/Packet Loss）”的精美图形，而简单的 Prometheus /`blackbox_exporter` 默认只提供基础指标。**

# Smokeping是如何运作的

`Smokeping` 执行 `fping` 命令并收集输出， 保存到 RRD 数据库：

```
fping -c $count -q -b $backoff -r $retry -4 -b $packetsize -t $timeout -i $mininterval -p $hostinterval $host [ $host ...]
```

其中这些参数默认值为：

- `$count`是20（数据包个数）
- `$backoff`是1（避免指数退缩）
- `$timeout`是1.5秒
- `$mininterval`是0.01秒（任意目标之间的最小等待间隔）
- `$hostinterval`是1.5秒（单个目标探针间最短等待时间）

Smokeping 的核心原理是基于 **RRDtool (Round Robin Database)** 和 **多重探测机制**。它不仅仅是一个简单的 Ping 工具，而是一个通过**统计分布**来展示网络质量的监控系统。

它的工作原理可以分解为以下四个步骤：

### 1. 并行探针机制 (Parallel Probing)

Smokeping 并不是发出一个 Ping 包就等待返回，而是采用**高频率、高并发**的方式。

- **分片发送**：它在每次采集周期内，会向目标地址发送一组探测包（默认通常是 20 个）。
- **多探针支持**：Smokeping 支持多种 Probe 类型，包括 `FPing` (标准的 ICMP Ping)、`Curl` (HTTP 探测)、`EchoPing` (TCP 端口探测) 等。
- **并行执行**：为了保证在大规模网络下的效率，它会同时启动多个子进程或线程对不同的目标进行探测，这使得它能同时监控成百上千个节点。

### 2. 时序数据的处理与存储 (RRDtool)

这是 Smokeping 区别于普通 Ping 工具的核心：

- **固定存储**：它使用 RRDtool 数据库。RRD（循环存储）的特点是**空间占用恒定**。无论你运行一天还是十年，它占用的磁盘空间都不会增加，因为旧的数据会根据预设的归档策略（Step/Heartbeat）被平均化处理。
- **数据压缩**：它不是记录每一个原始数据包，而是将一组探测包的结果计算出 `Min`、`Max`、`Avg` 和 `Packet Loss`，然后存入数据库。

### 3. 统计可视化 (烟雾)

Smokeping 最著名的图形展示（那种带有阴影的折线图）其实是一种统计分布的视觉映射：

- **阴影区 (The Smoke)**：图表中的阴影部分反映了**抖动（Jitter）**。阴影的宽度由该周期内探测包的 RTT 最大值与最小值之差决定。
- **核心线**：中间的深色线代表探测包的平均 RTT（Average），Smokeping 显示为“烟雾”（因此得名）。
- **丢包显示**：如果发生了丢包，Smokeping 会在图表底部用特定的颜色（通常是红色）进行标记，丢包越多，标记越明显。

### 4. 分布式架构 (Master-Slave)

在处理大规模网络时，Smokeping 采用 Master-Slave 架构：

- **Slave（边缘节点）**：部署在不同的网络区域。它们负责实际的 Ping 操作，并将收集到的原始数据同步给 Master。
- **Master（中心节点）**：负责维护全局配置、处理 RRD 数据存储、生成网页报表，并执行预设的告警逻辑。



# 如何在Grafana中绘制烟雾图

在Prometheus/Grafana中，无法实现类似 smokeping 中的“烟雾”效果。

有以下两种方法：

## 使用 blackbox_exporter

使用“分位数” 实现烟雾图。

配置blackbox_exporter抓取间隔15秒，每5分钟获取20个数据包，在Grafana面板中创建11个查询：

```
min_over_time(probe_icmp_duration_seconds{instance="$instance",phase="rtt"}[5m])
quantile_over_time(0.1, probe_icmp_duration_seconds{instance="$instance",phase="rtt"}[5m])
quantile_over_time(0.2, probe_icmp_duration_seconds{instance="$instance",phase="rtt"}[5m])
... etc ...
quantile_over_time(0.9, probe_icmp_duration_seconds{instance="$instance",phase="rtt"}[5m])
max_over_time(probe_icmp_duration_seconds{instance="$instance",phase="rtt"}[5m])
```

将这些图例设置为“0”、“0.1”、“0.2” ..."0.9", "1".然后用“fill down to ...”的系列覆盖，最终会得到看起来很像“烟雾”的东西！

> 可以参考我的另外一篇文档 “网络探测：Blackbox-Exporter”



## 使用 smokeping_exporter

Prometheus的一位维护者 [SuperQ](https://github.com/SuperQ) 为此编写了一个专门的导出器，叫做 [smokeping_prober](https://github.com/SuperQ/smokeping_prober/)]  (参见:[this discussion in the blackbox exporter](https://github.com/prometheus/blackbox_exporter/issues/115))。

也是使用prometheus中的"buckets" 或者 “百分位数”实现类似效果:

```
histogram_quantile(0.9 rate(smokeping_response_duration_seconds_bucket[$__interval]))
```



以下是SuperQ实现的理由：

> Yes, I know about smokeping's bursts of pings. IMO, smokeping's data model is flawed that way. This is where I intentionally deviated from the smokeping exact way of doing things. This prober sends a smooth, regular series of packets in order to be measuring at regular controlled intervals.
>
> Instead of 20 packets, over 10 seconds, every minute. You send one packet per second and scrape every 15. This has the same overall effect, but the measurement is, IMO, more accurate, as it's a continuous stream. There's no 50 second gap of no metrics about the ICMP stream.
>
> Also, you don't get back one metric for those 20 packets, you get several. Min, Max, Avg, StdDev. With the histogram data, you can calculate much more than just that using the raw data.
>
> For example, IMO, avg and max are not all that useful for continuous stream monitoring. What I really want to know is the 90th percentile or 99th percentile.
>
> This smokeping prober is not intended to be a one-to-one replacement for exactly smokeping's real implementation. But simply provide similar functionality, using the power of Prometheus and PromQL to make it better.
>
> [...]
>
> one of the reason I prefer the histogram datatype, is you can use the heatmap panel type in Grafana, which is superior to the individual min/max/avg/stddev metrics that come from smokeping.
>
> Say you had two routes, one slow and one fast. And some pings are sent over one and not the other. Rather than see a wide min/max equaling a wide stddev, the heatmap would show a "line" for both routes.

