---
title: 网络探测：Blackbox Exporter
date: 2023-10-24 +0800
categories: [Prometheus, Blackbox]
pin: false
math: true
tags: [Prometheus, Blackbox] 
---

Blackbox Exporter是Prometheus社区提供的官方黑盒监控解决方案，其允许用户通过：HTTP、HTTPS、DNS、TCP以及ICMP的方式对网络进行探测。

##  下载安装 Blackbox Exporter

```bash
cd ~
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
```

```bash
mkdir /etc/blackbox_exporter/
tar -xvf  blackbox_exporter-0.25.0.linux-amd64.tar.gz
cp blackbox_exporter-0.25.0.linux-amd64/blackbox_exporter  /usr/local/bin/
cp blackbox_exporter-0.25.0.linux-amd64/blackbox.yml  /etc/blackbox_exporter/
```

设置systemd 启动服务

```bash
cat > /etc/systemd/system/blackbox_exporter.service << EOF
[Unit]
Description= Prometheus blackbox_exporter
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox_exporter/blackbox.yml
Restart=on-failure
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=10s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
EOF
```

设置防火墙，并启动 blackbox_exporte

```bash
systemctl enable --now blackbox_exporter
firewall-cmd --add-port=9115/tcp --permanent
firewall-cmd --reload
```

### ICMP 权限

ICMP 探测需要提升的权限才能运行：

- *Windows*：需要管理员权限。

- *Linux*：需要具有`cap_net_raw`用户组权限 或者 root 权限 。执行以下命令，修改`net.ipv4.ping_group_range`

  ```bash
  sysctl  net.ipv4.ping_group_range = 0  2147483647
  ```

## ICMP 监控

### blackbox.yml 配置

```yaml
modules:
  icmp:
    prober: icmp
    icmp:
      preferred_ip_protocol: ip4  # <<< 优先IPv4
  icmp_ttl5:
    prober: icmp
    timeout: 5s
    icmp:
      ttl: 5
```

## 与Prometheus集成

假设有三个机房，分别位于GZ、SH、BJ。在每个机房都各部署一台blackbox_exporter 探针（Prober）,监控100+个目标（targets）:

```tex
                               -----------
                        +--   | Prober-BJ |   --+
                        |      -----------      |
                        |                       |
 -------------------    |      -----------      |         --------- 
| prometheus server | --+--   | Prober-GZ |   --+---  ---| Targets |
 -------------------    |      -----------      |         --------- 
                        |                       |
                        |      -----------      |
                        +--   | Prober-SZ |   --+
                               -----------
```

如果使用普通的方法，一个简单的 Prometheus 作业配置如下所示

```yaml
scrape_configs:
  - job_name: 'blackbox-1'
    metrics_path: /probe
    params:
      module: [icmp]  # Look for icmp response.
    static_configs:
      - targets:
        - 8.8.8.8
        - 9.9.9.9 
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # Blackbox exporter's address.（地区1）

  - job_name: 'blackbox-2'
    metrics_path: /probe
    params:
      module: [icmp]  # Look for icmp response.
    static_configs:
      - targets:
        - 8.8.8.8
        - 9.9.9.9 
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 2.2.2.2:9115  # Blackbox exporter's address.（地区2）
    ... ...
```

如果在一个Blackbox 导出器的配置中定义了多个URL 和模块，且在 The World 的不同位置有 20+ 个 Blackbox 导出器，该怎么办？

若**在 Prometheus 中为每个 Blackbox _exporter 定义具有所有目标 （URL） 的作业**，则 需要总共将近 20+ 个 Job 和 2000 多行配置。

### 使用 单个 Job 访问多个模块和目标

正如上面提供的 Job 示例中所看到的，其中模块名称、目标和导出器的地址是静态的，因此想要更改某些内容时，则需要更改整个配置。

多目标导出器（例如 snmp_exporter、blackbox_exporter）可以通过 [scrape 配置](https://github.com/prometheus/snmp_exporter#prometheus-configuration)将一组动态目标传递给它们。

下面使用 Prometheus 提供的 [**file_sd_config**](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config) 和 [**relabel_config**](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config) 功能，实现**使用单个 Job 访问多个模块和目标**。

```yaml
# Prometheus 配置
scrape_configs:
  - job_name: "blackbox_icmp-01"
    scrape_interval: 1s
    metrics_path: /probe
    params:
      module: [icmp]
    file_sd_configs:
      - files: ["/etc/prometheus/file_sd_config.d/blackbox_icmp_targets.yml"]
        refresh_interval: 1m
    relabel_configs:
      - source_labels: [__address__]
        regex: '.*;.*;.*;.*;(.*)'        # 提取Targets模板中的第5段
        target_label: __param_target     # 传递 `__param_target`的值 给 Blackbox exporter 
      - source_labels: [__param_target]  # 重写 `__param_target` 为`instance`
        target_label: instance
      - source_labels: [__param_module] # 将内置标签`__param_module`修改为`module`,并将其添加到 labelset
        target_label: module
      - source_labels: [__address__]
        regex: '.*;(.*);.*;.*;.*'       # 提取Targets模板中的第2段, Prober_Name
        target_label: prober
      - source_labels: [__address__]
        regex: '.*;.*;(.*);.*;.*'       # 提取Targets模板中的第3段, IDC
        target_label: idc
      - source_labels: [__address__]
        regex: '.*;.*;.*;(.*);.*'       # 提取Targets模板中的第4段, Profile
        target_label: Profile
      - source_labels: [__address__]
        regex: '(.*);.*;.*;.*;.*'       # 提取Targets模板中的第1段,注意顺序不能错。 Blackbox_IP_Port
        target_label: __address__
```

第一个重要部分是`file_sd_configs`自动发现服务，这意味着可以动态更改文件的内容，而无需重新加载 Prometheus 服务器。

### icmp_targets.yml 文件配置

```yaml
###################################################################
# 这里定义了Blackbox exporter需要监控的所有目标                       #
# 分号作为分隔符，该模板的结构如下:                                    #
#  <Blackbox_IP_Port>;<Prober_Name>;<IDC>;<Profile>;<Target_URL>  #
###################################################################
- targets:
  # Prober-BJ
  - 1.1.1.1:9115;BJ;TYO;CN2;154.82.64.2
  - 1.1.1.1:9115;BJ;TYO;CN2;154.82.65.2
  - 1.1.1.1:9115;BJ;TPE;CN2;156.248.76.2
  - 1.1.1.1:9115;BJ;TPE;CMI;156.248.75.2
  - 1.1.1.1:9115;BJ;TPE;CMI;156.248.77.2
  - 1.1.1.1:9115;BJ;HK1;CN2;154.82.72.2
  - 1.1.1.1:9115;BJ;HK1;CN2;154.82.73.2

  # Prober-SH
  - 2.2.2.2:9115;GZ;TYO;CN2;154.82.64.2
  - 2.2.2.2:9115;GZ;TYO;CN2;154.82.65.2
  - 2.2.2.2:9115;GZ;TPE;CN2;156.248.76.2
  - 2.2.2.2:9115;GZ;TPE;CMI;156.248.75.2
  - 2.2.2.2:9115;GZ;TPE;CMI;156.248.77.2
  - 2.2.2.2:9115;GZ;HK1;CN2;154.82.72.2
  - 2.2.2.2:9115;GZ;HK1;CN2;154.82.73.2

  # Prober-SZ
  - 3.3.3.3:9115;SZ;TYO;CN2;154.82.64.2
  - 3.3.3.3:9115;SZ;TYO;CN2;154.82.65.2
  - 3.3.3.3:9115;SZ;TPE;CN2;156.248.76.2
  - 3.3.3.3:9115;SZ;TPE;CMI;156.248.75.2
  - 3.3.3.3:9115;SZ;TPE;CMI;156.248.77.2
  - 3.3.3.3:9115;SZ;HK1;CN2;154.82.72.2
  - 3.3.3.3:9115;SZ;HK1;CN2;154.82.73.2
```

此处，每个 `target` 都包含所有元数据，这些元数据将在抓取后提取到时间序列的最终标签集中。如上所示，使用了`;`符号作为分隔符。(分隔符可以是任何有效字符)。

通过提供的分隔符拆分行，则每个目标将获得 5 个字段。这些字段如下：

```
<Blackbox_IP_Port>;<Prober_Name>;<IDC>;<Profile>;<Target_URL>
```

1. Blackbox 导出器地址 （IP：PORT）
2. 在 Blackbox 导出器配置中定义的``Prober`名称
3. 需要监控的目标所在的 `IDC` 名称
4. 需要监控的目标的 `Profile` 名称
5. 需要监控的目标 URL。

第二个重要的部分是`relable_configs` , 重新标记是一个强大的工具，可在目标被抓取之前动态重写目标的标签集。在此示例中，有 5 个重新标记的步骤。所有步骤的逻辑几乎相同。

### 重新标记导出器地址

在此步骤中，我们应该告诉 Prometheus 目标字符串中的哪个字段负责标签。

默认情况下，当Prometheus加载Target实例完成后，这些Target时候都会包含一些默认的标签：

- `__address__`：其值为 当前Target实例的地址`<host/ip>:<port>`
- `__scheme__`：其值为 抓取目标服务访问地址的HTTP Scheme，HTTP或者HTTPS
- `__metrics_path__`：其值为 抓取目标的 Metrics 端点
- `__param_<name>`：其值为 传递的URL参数中第一个名称为`<name>`的参数的值
  - 如果路径中存在任何URL参数，则它们的前缀会设置为`__param_<name>`


以“__”开头的所有标签是内置的特殊标签：

```yaml
      - source_labels: [__address__]
        regex: '(.*);.*;.*;.*;.*'       # 提取Targets模板中的第1段, Blackbox_IP_Port
        target_label: __address__
        replacement: "${1}"         # (default = $1)，可以省略
        action: replace             # (default = replace)，可以省略
```

与`__address__`标签对应的值是通过 matcher 提取的。`regex` 是一个有效的 RE2 正则表达式

> 输入文本：
> `1.1.1.1:9115;BJ;TYO;CN2;154.82.64.2`
>
> 正则表达式：
> `(.*);.*;.*;.*;.*`
>
> 输出：
> `${1}` = `1.1.1.1:9115`

### 重新标记instance标签

在此步骤中，将在标签下提取最后一个字段（目标 URL）。`instance`

```yaml
      - source_labels: [__address__]
        regex: '.*;.*;.*;.*;(.*)'       # 提取Targets模板中的第5段
        target_label: __param_target    # 传递 `__param_target`的值 给 Blackbox exporter 
      - source_labels: [__param_target] # 重写 `__param_target` 为`instance`
        target_label: instance
```

> 输入文本：
> `1.1.1.1:9115;BJ;TYO;CN2;154.82.64.2`
>
> 正则表达式：
> `.*;.*;.*;.*;(.*)`
>
> 输出：
> `${1}` = `154.82.64.2`



> - `__param_<name>`：其值为 传递的URL参数中第一个名称为`<name>`的参数的值
>   - 如果路径中存在任何URL参数，则它们的前缀会设置为`__param_<name>`



其他的`relable_configs`就不一一解释了，所有的步骤和逻辑都相同。



## 查询 ICMP Loss/RTT/Jitter

### 使用PromQL 计算 ICMP 丢包

* 通过metric   `probe_success`查询 icmp ping 是否成功 (success = 1 , faile = 0)

```
probe_success{job=~"blackbox_icmp.*"}
```

* 通过以下以下PromQL，计算 60 秒内的 icmp ping 平均丢包率

```bash
1- avg_over_time(probe_success{job=~"blackbox_icmp.*",instance=~".+"}[60S])
```



###  使用PromQL 计算 ICMP RTT

* 通过以下以下PromQL, 计算60秒内的 平均 icmp rtt 值。

**注意：**如果出现 icmp 丢包，blackbox 会返回：

```tex
probe_success{} = 0                 # 其值为 0 说明探测失败，这个是预期的正确结果
probe_icmp_duration_seconds{} = 0   # icmp探测失败(即发生了丢包)，预期的结果应该是空值 或者一个无限大的值。这里返回值为`0`,不是预期的结果。
```

比方说，发送了5个 icmp request，中间出现2次丢包，收到了3个 icmp reply :

````bash
# * 代表出现丢包
11ms  *   *  9ms  10ms
````

如果直接使用 `avg_over_time ( probe_icmp_duration_seconds {} )[5s]` 来计算5秒内的 平均 ICMP RTT，结果如下：
$$
(11 + 0 + 0 + 9 + 10) / 5 = 30 / 5 = 6  毫秒
$$
明显得到一个错误的结果。预期的平均 RTT  应该是：
$$
(10 + 9 + 10) / 3 = 30 / 3 = 10  毫秒
$$
需要排除 `probe_icmp_duration_seconds{} = 0` 的 情况。

应该使用以下PromQL 公式：

```bash
# 排除probe_icmp_duration_seconds{}=0的情况, 使用内部子查询（Subquery，1s精度）
avg_over_time(
    ( probe_icmp_duration_seconds{
         job=~"blackbox_icmp.*", phase="rtt"
      } > 0
	) [$interval:1s]
)

# 或者：
sum_over_time(
    probe_icmp_duration_seconds{
        job=~"blackbox_icmp.*", phase="rtt"
    } [$interval]
) 
/ ignoring(phase) 
sum_over_time(probe_success{job=~"blackbox_icmp.*"}[$interval])
```

### 计算 ICMP Jitter

使用`标准方差`，单位是 ‘平方秒’，没有实际意义，反映时间窗口内数据的 “波动幅度

```bash
# 计算 ICMP抖动 (标准方差，单位是 ‘平方秒’，没有实际意义，反映时间窗口内数据的 “波动幅度”)
# 排除probe_icmp_duration_seconds{}=0的情况, 使用内部子查询（Subquery，1s精度）
stddev_over_time(
	( probe_icmp_duration_seconds{
    	job=~"blackbox_icmp.*", phase="rtt"
      } > 0
     ) [$interval:1s]
 ) > 0.015
```

使用`平均差`，单位是 ‘秒’，反映时间窗口内数据的 “平均水平”

```bash
# 计算 ICMP抖动 (平均差，单位是 ‘秒’，反映时间窗口内数据的 “平均水平”)
# 使用内部子查询（Subquery，1s精度）
avg_over_time(
    abs(
        probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"} - probe_icmp_duration_seconds offset 1s
    )  [$interval:1s]
) > 0.015
```



### 使用 Record 规则 计算 ICMP 丢包

也可以使用 promtheus 记录规则来计算 ICMP 丢包情况：

```yaml
#  prometheus 配置
rule_files:
  - "/etc/prometheus/rule_files.d/*.yml"
```

rule_files文件配置：

```yaml
groups:
  - name: ICMP Reachability
    interval: 1s
    rules:
    # icmp loss 丢包率
    - record: blackbox_icmp:probe_success:avg_over_time
      expr: avg_over_time(probe_success{job=~"blackbox_icmp.*"}[60s])

    # 60秒内 最小 rtt , 使用Subquery表达式（1s精度），排除掉丢包的情况（排除probe_success{} = 0）
    - record: blackbox_icmp:probe_icmp_duration_seconds:min_over_time
      expr: min_over_time( (probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"}>0) [60s:1s])
    # 60秒内 最大 rtt
    - record: blackbox_icmp:probe_icmp_duration_seconds:min_over_time
      expr: min_over_time(probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"}[60s])

    # 直方图,使用Subquery表达式（1s精度），排除掉丢包的情况（排除probe_success{} = 0）
    - record: blackbox_icmp:probe_icmp_duration_seconds:quantile_over_time_25
      expr: quantile_over_time(0.25, (probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"}>0) [60s:1s])
    - record: blackbox_icmp:probe_icmp_duration_seconds:quantile_over_time_50
      expr: quantile_over_time(0.5,  (probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"}>0) [60s:1s])
    - record: blackbox_icmp:probe_icmp_duration_seconds:quantile_over_time_75
      expr: quantile_over_time(0.75, (probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"}>0) [60s:1s])
    - record: blackbox_icmp:probe_icmp_duration_seconds:quantile_over_time_75
      expr: quantile_over_time(0.95, (probe_icmp_duration_seconds{job=~"blackbox_icmp.*", phase="rtt"}>0) [60s:1s])

  - name: ICMP Loss Alert
    interval: 30s
    rules:
    - alert: PacketLoss
      expr: (1 - avg_over_time(blackbox_icmp:probe_success:avg_over_time[5m])) * 100 > 10
      labels:
      annotations:
        description: "ICMP packet loss for {{ $labels.instance }} - has been over 10% ({{ $value }}%) for 5 minutes."
        summary: "ICMP packet loss over 10%"
```



## 已知问题

#### 不适合超低延迟的icmp ping

- [ICMP RTT 不准确 ·问题 #315 ·普罗米修斯/blackbox_exporter](https://github.com/prometheus/blackbox_exporter/issues/315)

- [Investigate using SO_TIMESTAMP for packet timing · Issue #12 · prometheus-community/pro-bing](https://github.com/prometheus-community/pro-bing/issues/12) 

  (该库由 Prometheus 社区积极维护中，go-ping 已经弃用)

- [[inputs.ping\] Native method gives higher results comparing to exec · Issue #9729 · influxdata/telegraf](https://github.com/influxdata/telegraf/issues/9729#issuecomment-915119506)



> [alkuzad](https://github.com/alkuzad) [4 年前](https://github.com/influxdata/telegraf/issues/9729#issuecomment-915119506) · edited by [alkuzad](https://github.com/alkuzad)
>
> @manio 我研究了 go-ping 和 Linux curl 的源代码，发现它们的核心机制存在差异：
>
> - Linux 的 ping 使用套接字和 SO_TIMESTAMP 从内核获取时间戳，获得更准确的 RTT 结果。
> - Go-ping 使用 [time.now()](https://github.com/go-ping/ping/blob/ff8be3320020b96856d3d48505bcf8926809752e/ping.go#L586)来保存和计算时间 
>
> 对于较长的时间段来说，它的精度足够，但对于较短的时间段可能就没那么精确了。此外，发送数据包的时间生成并非在数据包发送前完美完成。这可能会引入进一步的延迟。



> 我研究了 Go 上最流行的 pinger 实现，不幸的是，它们都使用用户空间生成的时间戳来计算 RTT（往返时间）。这可能会导致严重的测量误差，尤其是在节点上由于节流或高利用率而导致 CPU 时间不足的情况下。



