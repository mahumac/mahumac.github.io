# 完全基于BGP的五大国内骨干网分流IP List

发布于 2024-03-09 1355 次阅读

------

## 背景介绍

家里拉了三条宽带，电不信联不通移不动各一条

- 移动 下1000M上40M
- 电信 下1000M上50M
- 联通 下2000M上100M

起因是我使用苍狼山庄的IP列表进行分流时，联通宽带几乎没有流量（苍狼山庄的CM、OtherIP走移动，其余CT、CU各走各家）

在使用Mikrotik进行分流时，我发现搜索到的大部分是根据APNIC的IP属地为CN直接划分为CNIP（whois中的inet记录或route记录），github也有一些列表有对三大的IP进行划分，但是似乎还是基于WHOIS并且更新频率较低（几周-几个月），并且划分粒度也不够小，比如有的列表划分了GWBN，但是缺少对教育网和科技网的划分，甚至基于WHOIS会把一些非国内使用的IP划入国内IP。

综上所述，正好我运营着一张BGP网络，我决定自己从BGP网络中获取需要的IP列表，**预生成的纯IP列表和Mikrotik配置文件可在文末获取**

## 原理介绍

对于现有的分流技术有：IP分流、OSPF分流、BGP分流，后两者需要路由器自身建立对其他具有BGP表的路由器，在不经特殊配置的情况下，与BGP上游服务器断开连接将会导致路由器分流失效，具有依赖和耦合性，于是技术方案果断选择IP分流

针对IP列表获取，首先从bird导出当前全球IPV4和IPV6路由表，并使用正则表达式匹配组匹配前缀和对应的ASPATH，根据中国具体国情、维基百科，能够直接与国际互联网建立BGP Session只有三大运营商、教育网和科技网，CIETNET、CGWNET、CBNET、CITICNET、长宽鹏博士、北京宽带通、珠江宽频、各省子ASN等一车骨干网和运营商则是购买三大的转接宽带，因此不计入独立的IP列表，根据维基百科，需要划分的IP组及其包含ASN如下：

- 'ChinaTelecom': ['4134', '4809', '23764'],
- 'ChinaUnicom': ['4837', '9929', '10099'],
- 'ChinaMobile': ['9808', '58453', '58807'],
- 'ChinaSciences': ['7497'],
- 'ChinaEducation': ['23911', '4538']

------

2024年9月2日更新

针对存在一些外企在华提供IDC服务，或一些国内ASN同时拥有直接国际互联网连接和国内连接的情况，因此将下列ASN根据其对接的国内五大主要网络，分别添加至对应列表中。特别的如38345，虽然为国内IP，但是完全没有和三大接驳的情况，将同时加入三大的列表以保证IP列表完整

- 37965 电信 Pacnet Business Solutions LTD
- 38345 电信 联通 移动 Internet Domain Name System Beijing Engineering Resrarch Center Ltd.
- 24151 电信 联通 China Internet Network Infomation Center
- 58593 电信 联通 Shanghai Blue Cloud Technology Co.,Ltd

下面是修改后的列表

- 'ChinaTelecom': ['4134', '4809', '23764', '37965', '38345', '24151', '58593'],
- 'ChinaUnicom': ['4837', '9929', '10099', '38345', '24151', '58593'],
- 'ChinaMobile': ['9808', '58453', '58807', '38345'],
- 'ChinaSciences': ['7497'],
- 'ChinaEducation': ['23911', '4538']

------

对于ASPATH的处理，使用正则表达式获得的匹配组 <cidr> 和 <as_path> ,对其中as_path组切分并反置，结合目标IP组进行二层循环遍历判断是否属于某一目标IP组，如2914 58453 9808 24547进行取反后第一匹配24547，24547不在任何一组，继续匹配9808，9808属于中国移动，于是该CIDR被输出至ChinaMobile_IPv4.txt并终止循环

以此类推处理全球IPV4和IPV6路由表后，我们得到了各运营商的未经处理的已在国际互联网中广播的前缀，使用python ipaddress包进行简单的CIDR合并，即可得到我们需要的基于BGP的IP列表，再经过一些字符串替换即可输出Mikrotik使用的配置文件

上述方法使用基于BGP路由表生成IP列表，并使用了ASN作为IP划分的依据，较使用WHOIS生成的IP列表具有更佳的有效性和实时性，因为基于BGP生成，产生的IP是一定可以通过互联网路由到的，使用WHOIS会包含一些没有被广播的前缀，这种IP就算添加至分流列表也没有实际意义，同时基于ASN的判断方法确保了导出的IP列表一定是中国互联网的IP，使用WHOIS导出可能会包含IP的inet或route记录属于中国，实际上在国外广播的IP（比如一车BGPlayer的IP）

同时生成过程全部为python命令，可以定时更新（一小时一更），解决了时效性的问题，将生成的IP列表上传至网页服务器，即可路由器与BGP服务器脱耦，即使BGP服务器或网页服务器下线，Mikrotik在无法获取最新IP列表的情况下也不会影响本地已有的IP列表

## 预生成IP列表

- 已聚合纯CIDR列表
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaAllNetwork_IPv4.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaAllNetwork_IPv6.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaEducation_IPv4.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaEducation_IPv6.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaMobile_IPv4.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaMobile_IPv6.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaSciences_IPv4.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaSciences_IPv6.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaTelecom_IPv4.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaTelecom_IPv6.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaUnicom_IPv4.txt
  - https://file.bairuo.net/iplist/output/Aggregated_ChinaUnicom_IPv6.txt
- 未聚合纯CIDR列表
  - https://file.bairuo.net/iplist/output/ChinaAllNetwork_IPv4.txt
  - https://file.bairuo.net/iplist/output/ChinaAllNetwork_IPv6.txt
  - https://file.bairuo.net/iplist/output/ChinaEducation_IPv4.txt
  - https://file.bairuo.net/iplist/output/ChinaEducation_IPv6.txt
  - https://file.bairuo.net/iplist/output/ChinaMobile_IPv4.txt
  - https://file.bairuo.net/iplist/output/ChinaMobile_IPv6.txt
  - https://file.bairuo.net/iplist/output/ChinaSciences_IPv4.txt
  - https://file.bairuo.net/iplist/output/ChinaSciences_IPv6.txt
  - https://file.bairuo.net/iplist/output/ChinaTelecom_IPv4.txt
  - https://file.bairuo.net/iplist/output/ChinaTelecom_IPv6.txt
  - https://file.bairuo.net/iplist/output/ChinaUnicom_IPv4.txt
  - https://file.bairuo.net/iplist/output/ChinaUnicom_IPv6.txt
- Mikrotik配置文件
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaAllNetwork_IPv4.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaAllNetwork_IPv6.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaEducation_IPv4.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaEducation_IPv6.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaMobile_IPv4.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaMobile_IPv6.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaSciences_IPv4.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaSciences_IPv6.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaTelecom_IPv4.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaTelecom_IPv6.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaUnicom_IPv4.rsc
  - https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaUnicom_IPv6.rsc

## Mikrotik更新脚本(分运营商）

```
/file/remove [/file/find name~"Aggregated"]
 
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaTelecom_IPv4.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaUnicom_IPv4.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaMobile_IPv4.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaSciences_IPv4.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaEducation_IPv4.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaTelecom_IPv6.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaUnicom_IPv6.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaMobile_IPv6.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaSciences_IPv6.rsc" mode=https
/tool/fetch url="https://file.bairuo.net/iplist/mikrotik_output/Aggregated_ChinaEducation_IPv6.rsc" mode=https
 
/import Aggregated_ChinaEducation_IPv4.rsc
/import Aggregated_ChinaEducation_IPv6.rsc
/import Aggregated_ChinaMobile_IPv4.rsc
/import Aggregated_ChinaMobile_IPv6.rsc
/import Aggregated_ChinaSciences_IPv4.rsc
/import Aggregated_ChinaSciences_IPv6.rsc
/import Aggregated_ChinaTelecom_IPv4.rsc
/import Aggregated_ChinaTelecom_IPv6.rsc
/import Aggregated_ChinaUnicom_IPv4.rsc
/import Aggregated_ChinaUnicom_IPv6.rsc
```

**免责提醒：本文所有内容为基于作者自身环境编写，请根据自身情况对数据进行处理，作者不承担因本文数据或内容引发的任何责任**