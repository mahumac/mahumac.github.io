---
title: 网络监控：SNMP Expoter
date: 2024-06-30 +0800
categories: [Prometheus, SNMP]
pin: false
math: true
tags: [Prometheus, SNMP] 
---

## **SNMP介绍**

SNMP由三部分组成： **`SNMP内核`** 、 **`管理信息结构SMI`** 和 **`管理信息库MIB`** 。

**`SNMP`** 内核负责协议结构分析，根据分析结果完成网管动作； **`SMI`** 是一种通用规则，用来命名对象和定义对象类型，以及把对象和对象的值进行编码的规则； **`MIB`** 在被管理的实体中创建命名对象，也就是一个实例。 **`SMI`** 规定游戏规则，在规则基础上由 **`MIB`** 实现实例化。

**常见术语**：

**企业码**：组成 **`OID`** 对象的厂商遵守的标识

> https://www.iana.org/assignments/enterprise-numbers/

**比如华为的企业码：2011**

**SMI编号结构**

> https://www.iana.org/assignments/smi-numbers/smi-numbers.xhtml

如果需要深入研究 **`SNMP`** 协议，建议读 **`TCP/IP 详解 卷1：协议`**

## **MIB介绍**

**`MIB`** 全称 **Management Information Base** ，其主要负责为所有的被管理网络节点建立一个 **`接口`** ，本质是类似IP地址的一串数字。例如我们会在使用 **`SNMP`** 的时候见到这样一组数字串：

```text
.1.3.6.1.2.1.1.5.0
```

**在这串数字中，每个数字都代表一个节点，其含义可以参考下表：**

| 1    | 3    | 6    | 1        | 2    | 1     | 1      | 5       | 0    |
| ---- | ---- | ---- | -------- | ---- | ----- | ------ | ------- | ---- |
| iso  | org  | dod  | internet | mgmt | mib-2 | system | sysName | end  |

显然，这个数字串可以直接理解为系统的名字。在实际使用中， **`我们将其作为参数可以读取该节点的值`** ，如果有写权限的话还可以更改该节点的值，因此， **`SNMP`** 对于系统管理员提供了一套极为便利的工具。但，在一般使用中，我们一般不使用这种节点的表达方式，而是使用更为容易理解的方式，对于上面的这个例子，其往往可以使用 **`SNMPv2-MIB::sysName.0`** 所替代。你可能会想，系统能理解它的含义吗？那你就多虑了，一般在下载 **`SNMP`** 工具包的时候还会下载一个 **`MIB`** 文件包，其提供了所有节点的树形结构。在该结构中可以方便的查找对应的替换表达。

建议使用 https://oidref.com 浏览所有公共MIB。

另外https://github.com/librenms/librenms/tree/master/mibs 也是 MIB 的良好来源。

## **SNMP Exporter**

SNMP Exporter是Prometheus社区提供的官方网络监控解决方案。

SNMP 使用分层数据结构，而 Prometheus 使用 n 维矩阵，两个系统完美映射，无需手动遍历数据。 `snmp_exporter`为您映射数据。

### 部署 SNMP Exporter

为了尽可能简单，直接在现有的服务器上安装 `snmp_exporter`二进制文件作为服务。

要查看最新版本的`snmp_exporter`，请访问其发布页面。[Releases · prometheus/snmp_exporter](https://github.com/prometheus/snmp_exporter/releases/)

```bash
cd ~
VERSION=0.28.0
wget https://github.com/prometheus/snmp_exporter/releases/download/v${VERSION}/snmp_exporter-${VERSION}.linux-amd64.tar.gz
```

```bash
tar -xvf snmp_exporter-${VERSION}.linux-amd64.tar.gz
mkdir -p /etc/snmp_exporter
cp snmp_exporter-${VERSION}*/snmp_exporter   /usr/local/bin/
```

### **添加systemd服务管理**

> `--config.file`参数可以多次使用以加载多个文件，或者使用通配符，例如`snmp*.yaml`
>
> `--snmp.module-concurrency`参数可以在一次抓取中从多个模块中检索信息

```bash
# Ubuntu 22.04.2 LTS
cat <<EOF  > /etc/systemd/system/snmp_exporter.service
[Unit]
Description=Prometheus SNMP Exporter
After=network-online.target

[Service]
User=prometheus
Restart=on-failure
Type=simple
ExecStart=/usr/local/bin/snmp_exporter \\
         --config.file /etc/snmp_exporter/snmp*.yaml \\
         --snmp.module-concurrency=3 \\
         --log.level=info 
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no
[Install]
WantedBy=multi-user.target
EOF
```

启动服务

```bash
systemctl enable --now snmp_exporter
systemctl status snmp_exporter
```

## **SNMP Exporter Generator**

上面已经完成 **`SNMP Exporter`** 的部署，前面说了，手写 **`snmp.yaml`** 是非常不友好的。

故我们需要一款配置生成工具进行配置生成，只需要我们填写一些关键的信息即可得到我们想要的配置文件，比如想要采集交换机的指标。

**`SNMP Exporter`** 提供了一套这样的配置生成器工具 Generator，接下来就来看下如何部署，其实 **`SNMP Exporter`** 主要难点就是在处理配置生成工具和协调 **`mib`** 库上。

### 编译生成器 Generator

由于对 `NetSNMP` 的动态依赖性，必须手动构建生成器。

#### 安装依赖项

```bash
# Debian/Ubuntu distributions.
sudo apt-get install unzip build-essential libsnmp-dev # Debian-based distros

# Redhat/Centos distributions.
sudo yum install gcc make net-snmp net-snmp-utils net-snmp-libs net-snmp-devel # RHEL-based distros
```



另外，编译`generator`依赖`golang > 1.23.1`, 最好安装较新版本的`golang`。**下载安装 Golang**：

```bash
# 查看golang版本
go version
```

1. 下载`golang`二进制文件

   ```bash
   VERSION=1.24.0
   wget https://go.dev/dl/go${VERSION}.linux-amd64.tar.gz
   ```

2. 通过删除 `/usr/local/go` 文件夹**来删除任何以前的 Go 安装** （如果存在），然后将刚下载的存档解压到 `/usr/local` 中，创建一个新的 `/usr/local/go` 中的 Go 树：

   ```bash
   rm -rf /usr/local/go
   tar -C /usr/local  -xzf  go${VERSION}.linux-amd64.tar.gz
   ```

   **不要**将存档解压到之前已有的 `/usr/local/go` 树中。这是已知的 产生损坏的 Go 安装。

3. 将 `/usr/local/go/bin`目录添加到环境变量中。

   您可以通过将以下行添加到 `$HOME/.profile` 或 `/etc/profile`（用于系统范围的安装）：

   ```
   ... ...
   export PATH=$PATH:/usr/local/go/bin
   ```

   使用命令`source $HOME/.profile` 或`source /etc/profile`  使对配置文件所做的更 立即生效.

   ```bash
   source $HOME/.profile
   source /etc/profile
   ```

4. 使用命令，确认已安装的 Go 版本：

   ```bash
   go version
   ```

#### 开始构建

```bash
cd ~
# 下载snmp_exporter git库
git clone https://github.com/prometheus/snmp_exporter.git	
cd snmp_exporter/generator
# 开始编译。`mibs`目录包含需要的网络设备mib文件
make generator mibs
```

 构建generator 并下载SNMP Exporter提供的公共mib库文件。

 国内网络问题一般会报错

```bash
root@prometheus-fz:~/snmp_exporter/generator# make generator mibs
go build
go: downloading github.com/prometheus/common v0.62.0
go: downloading github.com/alecthomas/kingpin/v2 v2.4.0
go: downloading gopkg.in/yaml.v2 v2.4.0
go: downloading github.com/gosnmp/gosnmp v1.38.0
go: downloading github.com/alecthomas/units v0.0.0-20211218093645-b94a6e3cc137
go: downloading github.com/xhit/go-str2duration/v2 v2.1.0
>> if download fails please check https://www.se.com/at/de/search/?q=powernet+mib&submit+search+query=Search for the latest release
# Workaround to make DisplayString available (#867)
>> Downloading apc-powernet-mib
>> Downloading readynas
>> Downloading Cisco AIRESPACE-REF-MIB
>> Downloading Cisco AIRESPACE-WIRELESS-MIB
>> Downloading ARISTA-ENTITY-SENSOR-MIB
>> ...
```

如果上面一直不动, 基本就是网络无法下载 开源项目提供的公共mib库,  那就不下载 , 按`Ctrl + C `  直接结束。

目的主要是构建出 generate 的二进制执行文件即可。

下载自己设备对应厂商的mibs文件，我这用的华为的CE88系列设备，就以华为设备为例

[华为 CE5800, CE6800, CE7800, CE8800软件下载和补丁升级=-华为=](https://support.huawei.com/enterprise/zh/switches/cloudengine-58-68-78-88-98-pid-252837181/software)，下载CE设备的mibs文件` CE_V200R023C00SPC500_MIB.zip ` 并解压

添加mib库文件路径临时环境变量

```bash
export MIBDIRS=~/snmp_exporter/generator/mibs
export MIBDIRS=~/CE_V200R023C00SPC500_MIB/MIBFile
```

#### 生成 snmp.yaml

```bash
# 生成最终snmp.yaml文件
# 注意： generator.yaml 配置在下一章节介绍
./generator --fail-on-parse-errors --snmp.mibopts=u generate  \
    -m ~/CE_V200R023C00SPC500_MIB/MIBFile  \
    -m ~/snmp_exporter/generate/mibs  \
    -g /etc/snmp_exporter/generator.yaml  \
    -o /etc/snmp_exporter/snmp.yaml
```

 将最终生成的snmp.yaml文件移动到snmp_exporter程序配置文件读取的路径

```bash
mv snmp.yaml /opt/snmp_exporter/
```

重启 snmp_exporter

```bash
systemctl restart snmp_exporter
```

**运行过程说明：**

配置生成器从 **`generator.yaml`** 中读取简化的收集指令并把相应的配置写入 **`snmp.yaml`** 。 **`snmp_exporter`** 程序仅使用 **`snmp.yaml`** 文件从开启了 **`snmp`** 的设备收集数据。

**generator程序  `args`参数解析**:

```text
# ./generator [<flags>] <command> [<args> ...]
-m    # 生成配置 需要读取的mibs库文件目录 可同时指定多个
-g    # 生成配置 需要读取的生成器配置文件 generator.yaml
-o    # 生成的配置保存路径和文件名
```

**generator程序 `flags`参数解析**:

```text
# ./generator [<flags>] <command> [<args> ...]
--fail-on-parse-errors    # 如果存在 MIB 解析错误，则以非空的状态退出
--snmp.mibopts            # 切换控制 MIB 解析的各种默认设置 请参阅 snmpwalk --help
--log.level=info          # 输出日志信息等级 debug, info, warn, error
--log.format=logfmt       # 输出日志格式 logfmt, json
--parse_errors            # 调试：打印 NetSNMP 输出的解析错误
--dump                    # 调试：转储已解析和准备的 MIB
```

示例1：

```bash
./generator --fail-on-parse-errors --snmp.mibopts=u generate \
    -m ~/huawei/mibs \
    -m ~/snmp_exporter/generate/mibs \
    -g /etc/snmp_exporter/generator-huawei.yaml \
    -o /etc/snmp_exporter/snmp-huawei.yaml
```

```bash
./generator --fail-on-parse-errors --snmp.mibopts=u generate \
    -m ~/huawei/mibs \
    -m ~/snmp_exporter/generate/mibs \
    -g /etc/snmp_exporter/generator-h3c.yaml \
    -o /etc/snmp_exporter/snmp-h3c.yaml
```

**`--snmp.mibopts`** flag 的作用：

```text
# 切换控制 MIB 解析的各种默认值

u           # 允许在MIB符号中使用下划线
c           # 禁止使用 "--" 来终止注释
d           # 保存MIB对象的描述
e           # 当MIB符号冲突时禁用错误
w           # MIB符号冲突时启用警告
W           # MIB符号冲突时启用详细警告
R           # 替换最新模块中的MIB符号

# 具体默认值类型是和 NetSNMP 的版本相关 可以通过命令查看 mibopts
snmpwalk --help
```

这个参数具体什么作用呢，主要解决的是有些mib库文件中，某些厂商并没有按照默认标准来，而是在 **`MIB`** 文件中使用了特殊符号，我们应该指定 **`MIB`** 解析的参数，比如某些 **`MIB`** 文件描述中有下划线 **`_`** 那如果使用某个指标去解析这个库应该是失败的，需要添加 **`——snmp.mibopts=u`** ，允许使用下划线。

**mibs文件目录规划**

建议**不同类型的设备**各自新建一个目录，其中包含不同设备类型的 **`mibs`** 目录、生成器可执行文件和 **`generator.yaml`** 配置文件。这是为了避免 **`MIB`** 定义中的名称空间冲突。仅在设备的 **`mibs`** 目录中保留所需的 **`MIB文件`** 。

### Generator 配置

可以按照不同设备 或 不同模块，创建不同的`generator.yaml`文件

`generator.yaml`的配置如下：

#### 采集系统基本信息

`generator-comm.yaml`:

```yaml
auths:
  auth_1:            # 认证模块名称，可以任意自定义，如：default、huawei、h3c
    version: 2          # snmp v2c版本
    community: public   # snmp 团体名，注意修改为自己的。
  auth_2:
    version: 2
    community: public

modules:
  # SNMPv2-MIB 系统基础信息，例如： sysName, sysDescr, sysUpTime, sysLocation, etc.
  # SNMPv2-MIB::sysUpTime, 1.3.6.1.2.1.1.3 , 启动以来运行的时间，32位计数器，单位为百分之一秒。 每496天会发生重置，不等于系统正常运行时间，
  snmpv2-mib:
    walk:
      - "SNMPv2-MIB::system"    # .1.3.6.1.2.1.1 , [sysName, sysDescr, sysUpTime, sysLocation, sysContact],etc
      - "SNMP-FRAMEWORK-MIB::snmpEngineTime"    # 1.3.6.1.6.3.10.2.1.3, 运行时间，单位是秒。可以替代`SNMPv2-MIB::sysUpTime`
```

#### 采集接口信息 (if-mib)

`generator-if-mib.yaml`:

```yaml
modules:
  # Default IF-MIB interfaces table with ifIndex.
  if-mib:
    walk:
      # 如果设备接口数量太多，遍历整个ifTable需要花费较长时间(超过1分钟)，建议只采集需要的项目 / 或者拆分为多个module去采集。
      # IF-MIB::ifTable     # 1.3.6.1.2.1.2.2 , 32位计数器， 采集间隔必须大于5s。该表的索引是ifIndex.
      - ifIndex                 # 1.3.6.1.2.1.2.2.1.1
      - ifDescr                 # 1.3.6.1.2.1.2.2.1.2
      #- ifType                  # 1.3.6.1.2.1.2.2.1.3
      - ifMtu                   # 1.3.6.1.2.1.2.2.1.4
      - ifAdminStatus           # 1.3.6.1.2.1.2.2.1.7
      - ifOperStatus            # 1.3.6.1.2.1.2.2.1.8
      - ifInDiscards            # 1.3.6.1.2.1.2.2.1.13
      - ifInErrors              # 1.3.6.1.2.1.2.2.1.14
      - ifOutDiscards           # 1.3.6.1.2.1.2.2.1.19
      - ifOutErrors             # 1.3.6.1.2.1.2.2.1.20

      # IF-MIB::ifXTable    # 1.3.6.1.2.1.31.1.1 , 64位计数器，该表是ifTable的补充, 采集间隔必须大于5s。该表的索引是ifIndex
      - ifName                  # 1.3.6.1.2.1.31.1.1.1.1,   接口名称
      - ifHCInMulticastPkts     # 1.3.6.1.2.1.31.1.1.1.8,   接收的组播报文个数，64bit
      - ifHCInBroadcastPkts     # 1.3.6.1.2.1.31.1.1.1.9,   接收的广播报文个数, 64bit
      - ifHCOutMulticastPkts    # 1.3.6.1.2.1.31.1.1.1.12,  发送的组播报文个数，包括被丢弃的报文或没有送出的报文
      - ifHCOutBroadcastPkts    # 1.3.6.1.2.1.31.1.1.1.13,  发送的广播报文个数，包括被丢弃的报文或没有送出的报文
      - ifHCInOctets            # 1.3.6.1.2.1.31.1.1.1.6,   接口上接收到的字节总数，包括成帧字符。
      - ifHCOutOctets           # 1.3.6.1.2.1.31.1.1.1.10,  接口上发送出的字节总数，包括成帧字符。
      - ifHighSpeed             # 1.3.6.1.2.1.31.1.1.1.15,  接口当前带宽,单位为1,000,000 bit/s
      - ifAlias                 # 1.3.6.1.2.1.31.1.1.1.18,  由网络管理员指定的接口别名/备注

    max_repetitions: 30   # 使用GET/GETBULK,一次可以请求的最大objects。值为60时，一个snmp resposne udp包有可能 >1500byte,不建议过大。默认为25。
    retries: 3
    timeout: 5s           # 每个SNMP request的超时时间, defaults to 5s.

    lookups:    # 针对Table类型的 OID 做标签‘插入/修改’操作
      - source_indexes: [ifIndex]
        lookup: IF-MIB::ifAlias             # 1.3.6.1.2.1.31.1.1.1.18   接口自定义描述信息
        drop_source_indexes: false          # 如果为 true，则从metric中删除此查找的source_index labels。需要确保索引唯一
      - source_indexes: [ifIndex]
        lookup: IF-MIB::ifName              # 1.3.6.1.2.1.31.1.1.1.1    接口名称
      - source_indexes: [ifIndex]
        lookup: IF-MIB::ifDescr             # 1.3.6.1.2.1.31.1.1.1.2
      - source_indexes: [ifIndex]
        lookup: IF-MIB::ifAdminStatus 
      - source_indexes: [ifIndex]
        lookup: IF-MIB::ifOperStatus

    overrides:
      ifAlias:
        ignore: true    # if ture: 意思是该OID对象不会生成单独的metric, 而是作为标签插入到metric中。目的是易于人类阅读，也减少数据库中的metric数量。
      ifName:
        ignore: false   # 保留ifName的metric
      ifDescr:
        ignore: true
      ifAdminStatus:
        ignore: true
      ifOperStatus:
        ignore: true
      #ifType:
      # type: EnumAsInfo

    filters:
      # 根据接口当前状态收集接口指标，过滤器目前仅仅支持oid ,不支持 oid name。可以减少snmpwalk/get请求数量
      # static:
      #   - targets:
      #     - ifIndex
      #     indices: ["2","3","4"]

      # 动态过滤器 常用于为 ifAlias、ifSpeed 或 ifAdminStatus、ifOperStatus状态中具有特定名称的接口指定过滤器。
      # 例如：仅获取 `ifAdminStatus = up` 或者 `ifHighSpeed > 1G`的接口
      dynamic:
        #- oid: 1.3.6.1.2.1.2.2.1.7 # ifAdminStatus
        - oid: 1.3.6.1.2.1.2.2.1.8  # ifOperStatus
          targets:
            - 1.3.6.1.2.1.2.2.1.1       # ifIndex
            - 1.3.6.1.2.1.2.2.1.2       # ifDescr
            - 1.3.6.1.2.1.2.2.1.4       # ifMtu
            - 1.3.6.1.2.1.2.2.1.7       # ifAdminStatus
            - 1.3.6.1.2.1.2.2.1.8       # ifOperStatus
            - 1.3.6.1.2.1.2.2.1.13      # ifInDiscards
            - 1.3.6.1.2.1.2.2.1.14      # ifInErrors
            - 1.3.6.1.2.1.2.2.1.19      # ifOutDiscards
            - 1.3.6.1.2.1.2.2.1.20      # ifOutErrors

            # IF-MIB::ifXTable
            - 1.3.6.1.2.1.31.1.1.1.1    # ifName
            - 1.3.6.1.2.1.31.1.1.1.8    # ifHCInMulticastPkts 
            - 1.3.6.1.2.1.31.1.1.1.9    # ifHCInBroadcastPkts 
            - 1.3.6.1.2.1.31.1.1.1.12   # ifHCOutMulticastPkts
            - 1.3.6.1.2.1.31.1.1.1.13   # ifHCOutBroadcastPkts
            - 1.3.6.1.2.1.31.1.1.1.6    # ifHCInOctets
            - 1.3.6.1.2.1.31.1.1.1.10   # ifHCOutOctets
            - 1.3.6.1.2.1.31.1.1.1.15   # ifHighSpeed
            - 1.3.6.1.2.1.31.1.1.1.18   # ifAlias
          values: ["1"]         # ifAdminStatus值：Up(1), down(2), testing(3)
          
        - oid: 1.3.6.1.2.1.31.1.1.1.1       # ifName
          targets:
            - 1.3.6.1.2.1.2.2.1.1       # ifIndex
            - 1.3.6.1.2.1.2.2.1.2       # ifDescr
            - 1.3.6.1.2.1.2.2.1.4       # ifMtu
            - 1.3.6.1.2.1.2.2.1.7       # ifAdminStatus
            - 1.3.6.1.2.1.2.2.1.8       # ifOperStatus
            - 1.3.6.1.2.1.2.2.1.13      # ifInDiscards
            - 1.3.6.1.2.1.2.2.1.14      # ifInErrors
            - 1.3.6.1.2.1.2.2.1.19      # ifOutDiscards
            - 1.3.6.1.2.1.2.2.1.20      # ifOutErrors

            # IF-MIB::ifXTable
            - 1.3.6.1.2.1.31.1.1.1.1    # ifName
            - 1.3.6.1.2.1.31.1.1.1.8    # ifHCInMulticastPkts 
            - 1.3.6.1.2.1.31.1.1.1.9    # ifHCInBroadcastPkts 
            - 1.3.6.1.2.1.31.1.1.1.12   # ifHCOutMulticastPkts
            - 1.3.6.1.2.1.31.1.1.1.13   # ifHCOutBroadcastPkts
            - 1.3.6.1.2.1.31.1.1.1.6    # ifHCInOctets
            - 1.3.6.1.2.1.31.1.1.1.10   # ifHCOutOctets
            - 1.3.6.1.2.1.31.1.1.1.15   # ifHighSpeed
            - 1.3.6.1.2.1.31.1.1.1.18   # ifAlias
          # '(?i)^(?!vlanif|tunnel)|vlanif(1|12)$'  精确匹配vlanif1、vlanif12,排除其他vlanif|tunnel开头的字符串 
          values: ["(?i)^(?!vlanif|tunnel)|vlanif(1|12)$"]  # (注意：此表达式无效，目前golang不支持否定预查 '?!')

        #- oid: 1.3.6.1.2.1.31.1.1.1.18      # ifAlias , 自定义的接口描述信息
        #  targets:
        #    - 1.3.6.1.2.1.2.2.1.1       # ifIndex
        #    - 1.3.6.1.2.1.2.2.1.2       # ifDescr
        #    - 1.3.6.1.2.1.2.2.1.4       # ifMtu
        #    - 1.3.6.1.2.1.2.2.1.7       # ifAdminStatus
        #    - 1.3.6.1.2.1.2.2.1.8       # ifOperStatus
        #    - 1.3.6.1.2.1.2.2.1.13      # ifInDiscards
        #    - 1.3.6.1.2.1.2.2.1.14      # ifInErrors
        #    - 1.3.6.1.2.1.2.2.1.19      # ifOutDiscards
        #    - 1.3.6.1.2.1.2.2.1.20      # ifOutErrors

        #    # IF-MIB::ifXTable
        #    - 1.3.6.1.2.1.31.1.1.1.1    # ifName
        #    - 1.3.6.1.2.1.31.1.1.1.8    # ifHCInMulticastPkts 
        #    - 1.3.6.1.2.1.31.1.1.1.9    # ifHCInBroadcastPkts 
        #    - 1.3.6.1.2.1.31.1.1.1.12   # ifHCOutMulticastPkts
        #    - 1.3.6.1.2.1.31.1.1.1.13   # ifHCOutBroadcastPkts
        #    - 1.3.6.1.2.1.31.1.1.1.6    # ifHCInOctets
        #    - 1.3.6.1.2.1.31.1.1.1.10   # ifHCOutOctets
        #    - 1.3.6.1.2.1.31.1.1.1.15   # ifHighSpeed
        #    - 1.3.6.1.2.1.31.1.1.1.18   # ifAlias
        #  values: ["^.+"]      # ifAlias字段值不为空（至少包含一个字符）
```

#### 采集光模块、CPU、风扇等信息（HUAWEI）

`generator-huawei-entity-extent-mib.yaml`:

```yaml
modules:
  huawei-entity-extent-mib:     # 索引是`ENTITY-MIB::entPhysicalIndex`，依赖公共`ENTITY-MIB`文件
    walk:
      ## entPhysicalTable , 每一个物理实体以及实体的类型和信息。
      - ENTITY-MIB::entPhysicalIndex    # 1.3.6.1.2.1.47.1.1.1.1.1 该表的索引
      - ENTITY-MIB::entPhysicalName     # 1.3.6.1.2.1.47.1.1.1.1.7 实体名

      ## hwEntityStateTable ，索引是entPhysicalIndex，位于`ENTITY-MIB`
      # 该表描述了实体的一些状态，包括管理状态、操作状态、备份状态、告警状态、CPU使用率及使用门限，内存利用率及使用门限等。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityAdminStatus   # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.1  实体Admin状态
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOperStatus    # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.2  实体Oper状态
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityCpuUsage      # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.5  实体实时的CPU使用率，缺省值：0
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityMemUsage      # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.7  实体内存使用率，缺省值：0
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityUpTime        # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.10 实体启动时间，单位秒
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityTemperature   # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.11 实体温度，单位°C
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityMemSizeMega   # 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.19 实体内存大小，单位是MB 

      ## hwRUModuleInfoTable ，索引是entPhysicalIndex
      # 该表是描述一些生产信息，其中包括，BOM ID，BOM的英文描述和本地描述，生产制造码和更新日志等信息。（暂时忽略）
      # - HUAWEI-ENTITY-EXTENT-MIB::hwRUModuleInfoTable        # 1.3.6.1.4.1.2011.5.25.31.1.1.2

      ## hwOpticalModuleInfoTable ，索引是 entPhysicalIndex
      # 该表描述了光模块一些基本信息，其中包括光模块模式、波长、传输距离、接收光功率、发送光功率等信息。
      # HUAWEI-ENTITY-EXTENT-MIB::hwOpticalModuleInfoTable     # 1.3.6.1.4.1.2011.5.25.31.1.1.3
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalWaveLength       # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.2 光模块波长（单位：nm）
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalTemperature      # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.5 光模块温度（单位: °C）
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalVoltage          # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.6 光模块电压（单位: mV）
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalBiasCurrent      # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.6 光模块偏置电流（单位uA）
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalRxPower          # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.8 接收功率（单位: dBm），读取的值需要除以100。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalTxPower          # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.9 发送功率（单位: dBm），读取的值需要除以100。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalType             # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.10 封装类型
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalRxLowThreshold   # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.13 接收功率下限阈值 dBm），读取的值需要除以100。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalRxHighThreshold  # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.13 接收功率上限阈值 dBm），读取的值需要除以100。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalTxLowThreshold   # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.13 发送功率下限阈值 dBm），读取的值需要除以100。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalTxHighThreshold  # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.13 发送功率上限阈值 dBm），读取的值需要除以100。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalTransType        # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.42 光模块传输类型 OCTET STRING 例如“100GBASE_SR4”
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalBiasLowThreshold # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.54 电流最低值（单位: uA）
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalBiasLowThreshold # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.54 电流最高值（单位: uA）
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalPortName         # 1.3.6.1.4.1.2011.5.25.31.1.1.3.1.58 接口名称

      ## hwFanStatusTable
      # 该表用于查询风扇信息。该表的索引是hwEntityFanSlot、hwEntityFanSn。
      - HUAWEI-ENTITY-EXTENT-MIB::hwFanStatusTable       # 1.3.6.1.4.1.2011.5.25.31.1.1.10 hwFanStatusTable, 查询风扇信，槽位、状态、转速等

      ## hwPwrStatusTable
      # 该表用于查询电源信息。该表的索引是hwEntityPwrSlot、hwEntityPwrSn。
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrSlot        # 1.3.6.1.4.1.2011.5.25.31.1.1.18.1.1 , 电源的槽位号
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrSn          # 1.3.6.1.4.1.2011.5.25.31.1.1.18.1.2 ，电源编号
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrState       # 1.3.6.1.4.1.2011.5.25.31.1.1.18.1.6 , 电源工作状态 NTEGER{supply(1),notSupply(2),sleep(3),unknown(4)}
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrCurrent     # 1.3.6.1.4.1.2011.5.25.31.1.1.18.1.7 , 电源的电流，单位：mA
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrVoltage     # 1.3.6.1.4.1.2011.5.25.31.1.1.18.1.8 , 电源的电压，单位：mV
      - HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrDesc        # 1.3.6.1.4.1.2011.5.25.31.1.1.18.1.9, 电源描述

      - HUAWEI-ENTITY-EXTENT-MIB::hwDevicePowerInfoTotalPower   # 1.3.6.1.4.1.2011.5.25.31.3.2 , 系统的总功率。
      - HUAWEI-ENTITY-EXTENT-MIB::hwDevicePowerInfoUsedPower    # 1.3.6.1.4.1.2011.5.25.31.3.2 , 系统当前使用的功率。

    max_repetitions: 30
    retries: 3

    lookups:
      - source_indexes: [entPhysicalIndex]  # 根据索引`entPhysicalIndex`，查找`ENTITY-MIB::entPhysicalName`,
        lookup: ENTITY-MIB::entPhysicalName # 并将`entPhysicalName`作为标签，使 metrics 易于人类阅读

        # 光模块
      - source_indexes: [entPhysicalIndex]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalPortName
      - source_indexes: [entPhysicalIndex]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalType
      - source_indexes: [entPhysicalIndex]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalTransType
      - source_indexes: [entPhysicalIndex]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityOpticalWaveLength

        # 电源
      - source_indexes: [hwEntityPwrSlot, hwEntityPwrSn]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrState  # 电源工作状态 NTEGER{supply(1),notSupply(2),sleep(3),unknown(4)}
      - source_indexes: [hwEntityPwrSlot, hwEntityPwrSn]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityPwrDesc   # 电源描述

        # 风扇
      - source_indexes: [hwEntityFanSlot, hwEntityFanSn]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityFanPresent    # 在位状态 INTEGER{normal(1),abnormal(2)}
      - source_indexes: [hwEntityFanSlot, hwEntityFanSn]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityFanState      # 风扇状态 INTEGER{normal(1),abnormal(2)}
      - source_indexes: [hwEntityFanSlot, hwEntityFanSn]
        lookup: HUAWEI-ENTITY-EXTENT-MIB::hwEntityFanDesc       # 风扇描述
        #drop_source_indexes: false

    overrides:
      entPhysicalName:
        ignore: true

      hwEntityOpticalType:      # 光模块封装类型 NTEGER{unknown(0),sc(1),gbic(2),sfp(3),rj45(5),xfp(6)...sfp56(28)}
        ignore: true
        type: EnumAsStateSet
      hwEntityOpticalTransType:     # 传输类型，OCTET STRING 例如“100GBASE_SR4”
        ignore: true
        type: DisplayString
      hwEntityOpticalConnectType:   # 光模块的接口类型。OCTET STRING 例如 “MPO”
        ignore: true
        type: DisplayString
      hwEntityOpticalPortName:      # 光模块名称
        ignore: true
      hwEntityOpticalWaveLength:    # 光模块波长（单位：nm）。
        ignore: true
      hwEntityOpticalRxPower:   # 光模块接收功率（单位: dBm) 值需要除以100
        scale: 0.01
      hwEntityOpticalTxPower:   # 光模块发送功率（单位: dBm) 值需要除以100
        scale: 0.01
      hwEntityOpticalRxLowThreshold:
        scale: 0.01
      hwEntityOpticalRxHighThreshold:
        scale: 0.01
      hwEntityOpticalTxLowThreshold:
        scale: 0.01
      hwEntityOpticalTxHighThreshold:
        scale: 0.01

      hwEntityPwrSlot:      # 电源槽位号
        ignore: true
      hwEntityPwrSn:        # 电源编号
        ignore: true
      hwEntityPwrState:     # 电源工作状态 NTEGER{supply(1),notSupply(2),sleep(3),unknown(4)}
        ignore: true
        type: EnumAsStateSet
      hwEntityPwrDesc:      # 电源描述
        ignore: true
        type: DisplayString

      hwEntityFanSlot:      # 风扇槽位号
        ignore: true
      hwEntityFanSn:        # 风扇编号
        ignore: true
      hwEntityFanPresent:   # 风扇在位状态 {present(1),absent(2)}
        ignore: true
        type: EnumAsStateSet
      hwEntityFanState:     # 风扇工作状态 {normal(1),abnormal(2)}
        ignore: true
        type: EnumAsStateSet
      hwEntityFanDesc:      # 风扇描述
        ignore: true
        type: DisplayString

      hwEntityAdminStatus:      # 实体管理状态{notSupported(1),locked(2),shuttingDown(3),unlocked(4),up(11),down(12),loopback(13)}
        ignore: true
        type: EnumAsStateSet
      hwEntityOperStatus:       # 实体操作状态{ 1~12同上，connect(13),protocolUp(15),linkUp(16),linkDown(17),present(18),absent(19)}
        ignore: true
        type: EnumAsStateSet

    filters:        # 根据接口当前状态收集接口指标，过滤器目前仅仅支持oid ,不支持 oid name。可以减少snmpwalk/get请求数量

    # 动态过滤器由SNMP_exporter传递，导出器将对oid进行snmp遍历，并对 targets 进行snmp遍历限制为与vulues列表中的值匹配的索引 
    # 常用于为 ifAlias、ifSpeed 或 ifAdminStatus、ifOperStatus状态中具有特定名称的接口指定过滤器。
    # 例如：仅获取 `ifAdminStatus = up` 或者 `ifHighSpeed > 1G`的接口
      dynamic: 
        - oid: 1.3.6.1.2.1.47.1.1.1.1.7     # ENTITY-MIB::entPhysicalName ,实体名
          targets:
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.1    # hwEntityAdminStatus
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.2    # hwEntityOperStatus 
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.5    # hwEntityCpuUsage
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.7    # hwEntityMemUsage
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.10   # hwEntityUpTime
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.11   # hwEntityTemperature
            - 1.3.6.1.4.1.2011.5.25.31.1.1.1.1.19   # hwEntityMemSizeMega
          values: ["(FM|CE)(88|68)\\d+.*"]         # 保留实体名为 CE8850-64CQ-EI FM-8850-64CQ-EI CE8861-4C-EI 等
```

#### 采集Flash存储信息 (HUAWEI)

`generator-huawei-flash-man-mib.yaml`:

```yaml
modules:
  huawei-flash-man-mib:
    walk: 
      - hwStorageSpace      # 1.3.6.1.4.1.2011.6.9.1.4.2.1.3 , Flash设备空间的大小 单位是千字节
      - hwStorageSpaceFree  # 1.3.6.1.4.1.2011.6.9.1.4.2.1.4 , Flash设备剩余空间 单位是千字节
      - hwStorageName       # 1.3.6.1.4.1.2011.6.9.1.4.2.1.5 , Flash设备名称

    lookups:
      - source_indexes: [hwStorageIndex]
        lookup: hwStorageName
    overrides:
      hwStorageName:
        ignore: true
        type: DisplayString

#  HUAWEI-DEVICE-MIB:
     # walk:
      #- 1.3.6.1.4.1.2011.6.3.4.1.2            # hwCpuDevDuty 5秒钟内的CPU的平均使用率
      #- 1.3.6.1.4.1.2011.6.3.4.1.3            # hwCpuDuty1min 1分钟内的CPU的平均使用率
      #- 1.3.6.1.4.1.2011.6.3.4.1.4            # hwCpuDuty5min 5分钟内的CPU的平均使用率
      #- 1.3.6.1.4.1.2011.6.3.5.1.1.2          # hwMemoryDevSize 每块板上内存总量
      #- 1.3.6.1.4.1.2011.6.3.5.1.1.3          # hwMemoryDevFree 每块板上空闲的内存总量
      #- 1.3.6.1.4.1.2011.6.3.5.1.1.4          # hwMemoryDevRawSliceUsed 每块板上已占用的raw slice内存总量
```

#### 采集 ARP 信息表

```yaml
# 参考：https://info.support.huawei.com/info-finder/tool/zh/enterprise/mib/ce8850-64cq-ei-vid-262063404
modules:
  ip-mib:
    walk:
      # "ipNetToPhysicalTable"  该表用于记录地址转换表（ARP和ND），包含IP地址到物理地址的关系
      # 1.3.6.1.2.1.4.35.1.1  # ipNetToPhysicalIfIndex, 接口的索引值, 与IF-MIB::ifIndex 相同
      # 1.3.6.1.2.1.4.35.1.2  # ipNetToPhysicalNetAddressType, IP地址类型。INTEGER{unknown(0),ipv4(1),ipv6(2),ipv4z(3),ipv6z(4),dns(16)}
      # 1.3.6.1.2.1.4.35.1.3  # ipNetToPhysicalNetAddress, IP地址 OCTET STRING{(0,255)}
      # 1.3.6.1.2.1.4.35.1.4  # ipNetToPhysicalPhysAddress, MAC地址 OCTET STRING{(0,65535)}
      # 1.3.6.1.2.1.4.35.1.5  # ipNetToPhysicalLastUpdated, 表项最后一次更新的系统时间
      # 1.3.6.1.2.1.4.35.1.6  # ipNetToPhysicalType，ARP映射表项的类型 INTEGER{other(1),invalid(2),dynamic(3),static(4),local(5)}
      # 1.3.6.1.2.1.4.35.1.7  # ipNetToPhysicalState, 邻居可达性探测的状态,INTEGER{reachable(1),stale(2),delay(3),probe(4),invalid(5),unknown(6),incomplete(7)}
      # 1.3.6.1.2.1.4.35.1.8  # ipNetToPhysicalRowStatus, 表示行状态。

      # 该表的索引是 ipNetToPhysicalIfIndex、ipNetToPhysicalNetAddress、ipNetToPhysicalNetAddressType, 是一个复合索引
      - "IP-MIB::ipNetToPhysicalTable"    # 1.3.6.1.2.1.4.35 

    max_repetitions: 30   # 使用GET/GETBULK,一次可以请求的最大objects。值为60时，一个snmp resposne udp包有可能 >1500byte,不建议过大。默认为25。
    retries: 3
    timeout: 5s           # 每个SNMP request的超时时间, defaults to 5s.

    lookups:    # 针对Table类型的 OID 做标签‘插入/修改’操作
      - source_indexes: [ipNetToPhysicalIfIndex]
        lookup: IF-MIB::ifName             # 1.3.6.1.2.1.31.1.1.1.1, 接口名称
      - source_indexes: [ipNetToPhysicalIfIndex]
        lookup: IF-MIB::ifAlias            # 1.3.6.1.2.1.31.1.1.1.18, 接口自定义描述信息

      - source_indexes: [ipNetToPhysicalIfIndex,ipNetToPhysicalNetAddress]
        lookup: IP-MIB::ipNetToPhysicalType

    overrides:
      ipNetToPhysicalIfIndex:
        ignore: true            # 丢弃metric: ipNetToPhysicalIfIndex{}
      ipNetToPhysicalNetAddressType:
        ignore: true
      ipNetToPhysicalNetAddress:
        ignore: true
      ipNetToPhysicalLastUpdated:
        ignore: true
      ipNetToPhysicalType:
        ignore: true            # 丢弃metric
      ipNetToPhysicalState:
        ignore: true
      ipNetToPhysicalRowStatus:
        ignore: true

    filters:
      # 注意： 该过滤器不生效，原因未知。通过抓取Job的`metric_relabel_configs`来丢弃metric
      dynamic:
        - oid: 1.3.6.1.2.1.4.35.1.6  # ipNetToPhysicalType,
          targets:
            - 1.3.6.1.2.1.4.35.1.1  # ipNetToPhysicalIfIndex, 接口的索引值, 与IF-MIB中的ifIndex值所指定接口相同
            - 1.3.6.1.2.1.4.35.1.2  # ipNetToPhysicalNetAddressType， IP地址类型。
            - 1.3.6.1.2.1.4.35.1.3  # ipNetToPhysicalNetAddress, IP地址 OCTET STRING{(0,255)}
            - 1.3.6.1.2.1.4.35.1.4  # ipNetToPhysicalPhysAddress，MAC地址 OCTET STRING{(0,65535)}
            - 1.3.6.1.2.1.4.35.1.5  # ipNetToPhysicalLastUpdated，表项最后一次更新的系统时间
            - 1.3.6.1.2.1.4.35.1.6  # ipNetToPhysicalType，ARP映射表项的类型 INTEGER{other(1),invalid(2),dynamic(3),static(4),local(5)}
            - 1.3.6.1.2.1.4.35.1.7  # ipNetToPhysicalState, 表示邻居可达性探测的状态
            - 1.3.6.1.2.1.4.35.1.8  # ipNetToPhysicalRowStatus, 表示行状态。
          values: ["1","3","4","5"]     # 丢弃值 invalid(2), 即 无效的mac地址（00:00:00:00:00:00）
```



#### 采集 MAC地址表 FdbTable

`generator-bridge-mib.yaml`:

```yaml
## 采集 MAC地址表 （FdbTable）
# `BRIDGE-MIB::dot1dTpFdbTable` , Q-BRIDGE-MIB::dot1qVlanFdbId包含了端口的vlan信息。

#### 查询MAC地址和接口的对应关系
# `dot1dTpFdbTable`表描述了当前设备上存在的MAC地址表项。其中`dot1dTpFdbAddress`节点描述了MAC地址，`dot1dTpFdbPort`节点描述了MAC地址对应的网桥端口号。
# `dot1dBasePortIfIndex`节点描述了网桥端口号和接口索引的对应关系。ifName节点描述了接口索引和接口名的对应关系。需要链式查找。
modules:
  bridge-mib:
    walk:
      # "dot1dTpFdbTable"  该表用于记录 MAC地址和接口的对应关系（FdbTable）
      # 1.3.6.1.2.1.17.4.3.1.1  # dot1dTpFdbAddress, MAC地址信息 OCTET STRING{(6,6)}
      # 1.3.6.1.2.1.17.4.3.1.2  # dot1dTpFdbPort, dot1dTpFdbAddress对应实例的值的端口号
      # 1.3.6.1.2.1.17.4.3.1.3  # dot1dTpFdbStatus, MAC条目的状态 INTEGER{other(1),invalid(2),learned(3),self(4),mgmt(5)}
      # 链式查找：dot1dTpFdbAddress --> dot1dTpFdbPort --> dot1dBasePort --> dot1dBasePortIfIndex(IF-MIB::ifIndex)

      - "BRIDGE-MIB::dot1dTpFdbTable"       # 1.3.6.1.2.1.17.4.3, FDB表（mac转发表），该表的索引是`dot1dTpFdbAddress`(即mac地址)
      - "BRIDGE-MIB::dot1dBasePortTable"     # 1.3.6.1.2.1.17.1.4, 该表的索引是 `dot1dBasePort`。

    max_repetitions: 30   # 使用GET/GETBULK,一次可以请求的最大objects。值为60时，一个snmp resposne udp包有可能 >1500byte,不建议过大。默认为25。
    retries: 3
    timeout: 5s           # 每个SNMP request的超时时间, defaults to 5s.

    lookups:    # 针对Table类型的 OID 做标签‘插入/修改’操作，注意顺序不能错
      - source_indexes: [dot1dTpFdbAddress]
        lookup: BRIDGE-MIB::dot1dTpFdbPort      # dot1dTpFdbAddress 查找 dot1dTpFdbPort (dot1dTpFdbTable 表内查找)

      - source_indexes: [dot1dTpFdbPort]
        lookup: BRIDGE-MIB::dot1dBasePort       #（链接2个表：dot1dTpFdbTable，dot1dBasePortTable）查找 dot1dBasePort
      - source_indexes: [dot1dTpFdbPort]
        lookup: dot1dBasePortIfIndex            #（链接2个表：dot1dTpFdbTable，dot1dBasePortTable）查找 dot1dBasePortIfIndex

      - source_indexes: [dot1dBasePortIfIndex]  #（链接2个表：，dot1dBasePortTable，IF-MIB::ifXTable）
        lookup: IF-MIB::ifName             # 1.3.6.1.2.1.31.1.1.1.1, 接口名称
      - source_indexes: [dot1dBasePortIfIndex]
        lookup: IF-MIB::ifAlias            # 如果接口数量很多，避免高基数问题，不要将ifAlias作为table插入metric中

      - source_indexes: [dot1dBasePort]         #（链接3个表：，dot1dTpFdbTable，dot1dBasePortTable，IF-MIB::ifXTable）
        lookup: dot1dBasePortIfIndex

      - source_indexes: [dot1dTpFdbAddress]
        lookup: BRIDGE-MIB::dot1dTpFdbStatus

    overrides:
      dot1dTpFdbPort:
        ignore: true        # 丢弃metric: dot1dTpFdbPort{}
      dot1dTpFdbStatus:     # MAC条目的状态 INTEGER{other(1),invalid(2),learned(3),self(4),mgmt(5)}
        ignore: true
        type: EnumAsInfo

      dot1dBasePort:
        ignore: true
      dot1dBasePortIfIndex:
        ignore: true
```

#### 重新生成 `snmp.yaml`

`generator-*.yaml`文件修改完成之后，重新生成 `snmp-*.yaml`文件

```bash
~/snmp_exporter/generator/generator --fail-on-parse-errors --snmp.mibopts=u generate  \
    -m ~/CE_V200R023C00SPC500_MIB/MIBFile  \
    -m ~/snmp_exporter/generate/mibs  \
    -g /etc/snmp_exporter/generator/generator-comm.yaml  \
    -o /etc/snmp_exporter/snmp-comm.yaml
```

```bash
~/snmp_exporter/generator/generator --fail-on-parse-errors --snmp.mibopts=u generate  \
    -m ~/CE_V200R023C00SPC500_MIB/MIBFile  \
    -m ~/snmp_exporter/generate/mibs  \
    -g /etc/snmp_exporter/generator/generator-if-mib.yaml  \
    -o /etc/snmp_exporter/snmp-if-mib.yaml
```

```bash
~/snmp_exporter/generator/generator --fail-on-parse-errors --snmp.mibopts=u generate  \
    -m ~/CE_V200R023C00SPC500_MIB/MIBFile  \
    -m ~/snmp_exporter/generate/mibs  \
    -g /etc/snmp_exporter/generator/generator-huawei-entity-extent-mib.yaml  \
    -o /etc/snmp_exporter/snmp-huawei-entity-extent-mib.yaml
```

```bash
~/snmp_exporter/generator/generator --fail-on-parse-errors --snmp.mibopts=u generate  \
    -m ~/CE_V200R023C00SPC500_MIB/MIBFile  \
    -m ~/snmp_exporter/generate/mibs  \
    -g /etc/snmp_exporter/generator/generator-bridge-mib.yaml  \
    -o /etc/snmp_exporter/snmp-bridge-mib.yaml
```

然后重新启动 `snmp_exporter`

```bash
systemctl restart snmp_exporter.service
```



## Prometheus配置

按照`generator.yaml`中定义的不同module，查分成多个job 。以便优化`snmp_exporter`的性能：

- 配合 `--snmp.module-concurrency=3 `参数，提高并发性，可以在一次抓取中从多个模块中检索信息
- 根据需要，每个module的采集周期可以设置不一样。例如 接口信息1分钟收集一次, 光模块信息5分钟收集一次
- 根据不同型号、不同厂家的设备、设备性能来对应不同的module ，拆分为多个配置文件 或 job
- 根据设备 或者 module的采集时间不同，拆分为多个配置文件 或 job



举例：按照不同的采集频率来拆分job:

新建 `/etc/prometheus/file_sd_config.d/snmp_1m-huawei.yaml`文件，对应Prometheus中的1分钟轮询 Job，添加需要收集snmp信息的设备

```yaml
# 1分钟轮询的模块, 注意`module` 参数中是使用逗号分隔的模块名称列表，不是数组
- labels:
    auth: "public_v2"
    model: "CE8800"
    region: "BJ"
    module: "if-mib,snmpv2-mib"      # 名称对应generator.yaml中的module名称
  targets:
    - 192.168.1.1;BJ-Spine-CE8850    # 格式'<IP>;<HOSTNAME>'
    - 10.10.1.1;BJ-Leaf-CE6865
```

新建 `/etc/prometheus/file_sd_config.d/snmp_5m-huawei.yaml`文件，对应Prometheus中的5分钟轮询 Job，添加需要收集snmp信息的设备

```yaml
# 5分钟轮询的模块,注意`module` 参数中是使用逗号分隔的模块名称列表，不是数组
- labels:
    auth: "public_v2"
    model: "CE8800"
    region: "BJ"
    module: "ip-mib,bridge-mib,huawei-entity-extent-mib,huawei-flash-man-mib"
  targets:
    - 192.168.1.1;BJ-Spine-CE8850
    - 10.10.1.1;BJ-Leaf-CE6865
```

Prometheus Job配置：

```yaml
scrape_configs:
- job_name: "snmp-1"
    scrape_interval: 1m    # 1分钟轮询
    scrape_timeout: 1m
    file_sd_configs:
      - files: ["/etc/prometheus/file_sd_config.d/snmp_1m*.yaml"]
        refresh_interval: 1m
    metrics_path: /snmp
    relabel_configs:
      - source_labels: [__address__]
        regex: '(.*);.*'                    # 提取target模板中的第1段，instance
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - source_labels: [__address__]
        regex: '.*;(.*)'                    # 提取target模板中的第2段，hostname
      - source_labels: [auth]
        target_label: __param_auth         # 将 标签'auth'的值传递给snmp_exporter
      - source_labels: [module]
        target_label: __param_module       # 将 标签'module'的值传递给snmp_exporter
      - target_label: __address__
        replacement: 127.0.0.19:9116       # snmp_exporter服务IP和端口
      - target_label: prober
        replacement: HK1

  - job_name: "snmp-2"
    scrape_interval: 5m    # 5分钟轮询
    scrape_timeout: 5m
    file_sd_configs:
      - files: ["/etc/prometheus/file_sd_config.d/snmp_5m*.yaml"]
        refresh_interval: 1m
    metrics_path: /snmp
    relabel_configs:
      - source_labels: [__address__]
        regex: '(.*);.*'                    # 提取target模板中的第1段，instance
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - source_labels: [__address__]
        regex: '.*;(.*)'                    # 提取target模板中的第2段，hostname
        target_label: instance
      - source_labels: [auth]
        target_label: __param_auth         # 将 标签'auth'的值传递给snmp_exporter
      - source_labels: [module]
        target_label: __param_module       # 将 标签'module'的值传递给snmp_exporter
      - target_label: __address__
        replacement: 127.0.0.1:9116        # snmp_exporter服务IP和端口
      - target_label: prober
        replacement: HK1
     metric_relabel_configs:
      - source_labels: [ipNetToPhysicalType]
        regex: '2'          # 丢弃无效mac地址，如 '00:00:00:00:00:00'
        action: drop
      - source_labels: [ipNetToPhysicalPhysAddress]
        regex: '00:00:00:00:00:00'
        action: drop
      - source_labels: [ipNetToPhysicalPhysAddress]
        regex: '(.*):(.*):(.*):(.*):(.*):(.*)'
        replacement: '$1$2$3$4$5$6'      # 将mac地址格式由'11:22:33:44:55:66' 修改为 '112233445566'
        action: replace
        target_label: ipNetToPhysicalPhysAddress
      - source_labels: [dot1dTpFdbAddress]
        regex: '(.*):(.*):(.*):(.*):(.*):(.*)'
        replacement: '$1$2$3$4$5$6'      # 将mac地址格式由'11:22:33:44:55:66' 修改为 '112233445566'
        action: replace
        target_label: dot1dTpFdbAddress
```



## 性能查看

可以在`prometheus`中查看以下指标：

- `snmp_scrape_walk_duration_seconds` 

- `snmp_scrape_packets_retried`

-  `snmp_scrape_pdus_returned` 

- `snmp_scrape_packets_retried`

- `snmp_scrape_packets_send`
