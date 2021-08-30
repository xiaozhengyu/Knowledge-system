# Redis哨兵模式（Sentinel）

官方文档：[Redis Sentinel Documentation](https://redis.io/topics/sentinel)

---

从宏观层面看，Sentinel能够提供如下功能：

-   监控（Monitoring）：监控Master和Slave是否正常工作
-   通知（Notification）：Sentinel可以通过API向系统管理员或其他计算机程序通知监视到的Redis实例异常
-   自动故障迁移（Automatic failover）：自动替换失效的Master
-   Configuration provider：向客户端提供Master的地址



Sentinel运行分布式部署，这样做有如下好处：

1.  提高故障判断的准确率：综合多个Sentinel的监控结果 ，有效避免单个Sentinel的主观误判
2.  提高故障迁移系统的可靠性：Sentinel本身也存在发生故障的可能性，单节点的故障迁移系统不够可靠

