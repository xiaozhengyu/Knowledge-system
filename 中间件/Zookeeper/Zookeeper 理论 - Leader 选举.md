# Leader 选举

Zookeeper 的 Leader 选举有两种情况：

1.   服务器启动时选举 Leader
2.   运行过程中 Leader 服务器宕机，重新选举 Leader



在分析选举原理前，先介绍几个重要的参数。

-   服务器 ID(myid)：编号越大在选举算法中权重越大
-   事务 ID(zxid)：值越大说明数据越新，权重越大
-   逻辑时钟(epoch-logicalclock)：同一轮投票过程中的逻辑时钟值是相同的，每投完一次值会增加

