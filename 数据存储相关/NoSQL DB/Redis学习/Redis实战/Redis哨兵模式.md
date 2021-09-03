# Redis哨兵模式（Sentinel）

官方文档：[Redis Sentinel Documentation](https://redis.io/topics/sentinel)

---



>   Redis Sentinel provides **high availability** for Redis. In practical terms this means that using Sentinel you can create a Redis deployment that **resists without human intervention certain kinds of failures**.
>
>   Sentinel为Redis提供高可用性——能够在没有人为干预的情况下自行解决某些故障



## 宏观能力

从宏观层面看，Sentinel具有如下能力：

-   监控（Monitoring）：Sentinel会不断检查Master和Slave是否正常工作
-   通知（Notification）：Sentinel能够通过相关API向外发送检查到的异常
-   自动故障迁移（Automatic failover）：自动选择Slave替换出现异常的Master
-   配置查询中心（Configuration provider）：向外提供可靠的Master地址



## 分布式特性

Sentinel的分布式特性：Sentinel从一开始就被设计成可以与多个Sentinel进程协同工作（multiple Sentinel processes cooperating together）



多Sentinel进程协作的优势：

1.   综合多个Sentinel的Master故障检测结果，降低误判的概率
2.   提高Sentinel自身的可靠性：故障迁移系统本身必备足够可靠（打铁还需自身硬）



Sentinel、Master、Slave、Client共同构成了一个大型的分布式系统。



## 使用指南



### 启动方式

1.   获取Sentinel

     Redis2.6自带的Sentinel版本现已弃用，Sentinel稳定版从Redis2.8开始发布。

     

2.   启动Sentinel

     两种效果完全相同的启动方式：

     ```
     // 1
     redis-sentinel /path/to/sentinel.conf
     // 2
     redis-server /path/to/sentinel.conf --sentinel
     ```

     注意：

     1.   启动Sentinel时必须指定配置文件，如果没有指定配置文件Sentinel将拒绝启动（后文说明）
     2.   服务器必须开启26379端口。默认情况下，Sentinel使用26379端口接收其他Sentinel实例的连接请求



### 注意事项

1.   至少需要3个Sentinel实例才能保证故障迁移系统的可靠性
2.   Sentinel实例最好部署在完全独立的服务器（进一步保障故障迁移系统的可靠性，<u>避免被一锅端</u>）
3.   由于Redis使用异步复制，Sentinel + Redis 分布式系统不保证故障期间<u>被确认的写</u>(`acknowledged writes`)被保留（后文说明）



### 配置文件

Redis源码发行版会自带一个sentinel.conf实例文件。使用sentinel最典型、最基本的配置如下所示：

```
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

只需指定监视的Master节点，并为其设置唯一的名称。无需指定Slave，Sentinel能够自动发现Master的Slave。Sentinel会自动将Slave的信息附加到配置文件，以便重启时保留这些信息。当系统发生故障迁移，某个Slave被升级为Master，或者当系统发现了新的Sentinel，配置信息将发生更新。



上面的示例配置了Sentinel对两组Redis实例的监控，两组实例的Master名称分别被设为mymaster和resque。



`sentinel monitor`语句的参数说明如下：

```
sentinel monitor <master-group-name> <ip> <port> <quorum>
```

除了`quorum`，其他参数的含义都很简单，下面对`quorum`进行说明：

-   当有`quorum`个Sentinel同时认为Master已经故障，Master才会被标记为”故障“，并条件允许时进行故障迁移
-   需要注意，`quorum`仅用于故障检测（the quorum is only used to detect the failure），检测出故障后不是一定会进行故障迁移！还需满足条件：大多数Sentinel参与了决策（majority of the Sentinel）

举个例子，如果你有5个Sentinel进程，并且将quorum设置为2，那么可能出现以下情况：

-   在同一时刻至少有2个Sentinel进程认为Master已经发生故障，其中一个Sentinel将被授权进行故障迁移
-   如果至少有3个Sentinel进程参与了决策，故障迁移就会实际开始（5 * 0.5 = 2.5 ）

实际上，这意味着在故障期间，如果大多数Sentinel进程无法进行通信，那么Sentinel永远不会启动故障迁移（也就是在少数分区中永远没有故障迁移）



其他配置基本都采用如下格式：

```
sentinel <option_name> <master_name> <option_value>
```

-   down-after-milliseconds：如果超过down-after-milliseconds毫秒时间Master还没有响应，Sentinel就任务Master发生了故障
-   parallel-syncs：故障迁移完成后，允许同时有parallel-syncs个Slave进行数据同步（切换Master、同步新Master的数据）。数值越小意味着所有Slave完成数据同步的时间越长，客户端拿到旧数据的风险越高。数值越大意味着所有Slave完成数据同步的时间越短，虽然复制过程对于Slave来说大多是非阻塞的，但有时它会停止从Master加载批量数据。



### 示例分析



### 命令手册

## 内部原理

### Quorum

### Configuration epochs

### Configuration propagation

### Consistency under partitions

### Sentinel persistent state

### TILT mode



