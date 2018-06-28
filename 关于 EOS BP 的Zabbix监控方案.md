# 关于 EOS BP 的监控方案

我们是dpos.club, 也是一个eos bp，主要介绍我们自己对EOS BP 的监控的考量。
eos 主网刚刚正式上线，按照完整的监控规范，一些监控数据项的收集方法也并不完善，本文作为0.1版本，仅仅介绍监控的一些基本情况。

从安全角度考虑，BP一般会组建集群，多跳转节点转发广播，也会有多个网络出口。对于负责签名的BP Node而言，安全需求更高，因此监控系统应尽量减少对节点安全性的影响，比如只允许节点主动暴露数据。

因为节点的集群，普遍不会超过50台机器，使用zabbix监控是一个较好的方案（基于易用性，丰富的告警，性能，服务的监控，插件的支持等方面的考虑）。

## Zabbix 的部署和配置

zabbix server 以及agent的安装不再详述，zabbix已经被广泛使用，官方网站以及搜索引擎会解决你安装时遇到的各类问题。本文也不会介绍zabbix常见的配置，仅介绍除常规配置外一些需要注意的地方。

需要强调一些组网上的问题：
1. 为保证与被监控节点的数据传输，zabbix server尽量与需要监控的机器在内网部署。同时，限制zabbix server的访问，比如只允许特定的ip来访问。
2. 对于保存着私钥的机器（BP Node），不允许zabbix server主动发起连接，只允许被监控节点去往 zabbix server 10051端口访问(zabbix server收集数据的默认端口，如果修改了默认端口，请更改为修改后的值)。
3. 配置 zabbix agent为主动模式， `zabbix_agentd.conf` 中的如下配置：
``` 
StartAgents=0    #开启主动模式，主动push数据
#Server=x.x.x.x     # 注释掉用于被动模式的Server配置
ServerActive=10.66.1.16      #zabbix server 访问的ip和端口
Hostname=fullnode-vpc-eos       #服务器的hostname 需要与zabbix server上配置的Host name 相同。
BufferSize=100          # 根据需求调整BufferSize的配置,默认100。在内存中缓存到足够数量的监控记录后上报zabbix server
```
4. 切记不要开启zabbix_agentd.conf中的 `EnableRemoteCommands=1`，保持默认值0即可。该参数为1可以允许zabbix server执行远程命令，会是很大的安全风险。
 
## TBC
1. 数据同步是否及时的告警（网络）
2. 判断正常出块的告警
    * 检查主动出块日志记录
    * 查询块的广播结果
3. 判断是否分叉（脚本）以及告警
4. 更改为完全的主动监控。zabbix agent 的主动模式依然需要从zabbix查询监控项（items），后期会提供脚本采用完全push的方式，监控项的维护也由client本身维护，尽量减少BP与外网的通信。

## eos监控zabbix模板的使用
导入zabbix模板“zbx_export_templates_for_bp.xml”，将模板“Template OS Linux Active for EOS BP”关联监控的BP主机。




