# MySQL - 主从复制与读写分离

[toc]

## 主从复制

主要涉及三个线程: BinLog 线程、I/O 线程和 SQL 线程。

-   BinLog 线程：负责记录主数据库上所有修改相关（insert,update,delete,create,drop,alter）的SQL语句至二进制日志
-   I/O线程：负责读取主数据库的二进制日志，并写入从数据库的中继日志
-   SQL线程：负责读取并执行中继日志中的SQL

![img](markdown/MySQL - 主从复制与读写分离.assets/master-slave.png)



## 读写分离

|  数据库  |              职责              |
| :------: | :----------------------------: |
| 主数据库 | 写操作、实时性要求较高的读操作 |
| 从数据库 |     实时性要求不高的读操作     |

读写分离能够提高系统性能的原因：

-   主、从数据库负责各自的读和写，极大程度的缓解了锁的争用
-   从服务器可以使用MyISAM，提升查询性能，降低系统开销
-   增加数据冗余，提高系统可用性（AP with C）

![img](markdown/MySQL - 主从复制与读写分离.assets/dae709e9896b22141c8f7379b436cc73.png)

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

![img](markdown/MySQL - 主从复制与读写分离.assets/master-slave-proxy.png)