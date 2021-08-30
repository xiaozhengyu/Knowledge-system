# Redis主从复制



<img src="markdown/Redis主从复制.assets/image-20210829162405117.png" alt="image-20210829162405117" style="zoom:50%;" />



## 简单实现

![image-20210829223112751](markdown/Redis主从复制.assets/image-20210829223112751.png)

### 1. 准备Master和Slave的配置文件

Master配置文件（master-6379.conf）：

```
include /usr/local/bin/redis.conf            # 引用公共配置文件

port 6379
pidfile /var/run/redis_6379.pid
logfile /usr/local/bin/log/master_6379.log   # 日志文件
dbfilename master_6379.rdb                   # rdb文件
```



Slave 1配置文件（slave-6380.conf）：

```
include /usr/local/bin/redis.conf

port 6380
pidfile /var/run/redis_6380.pid
logfile /usr/local/bin/log/slave_6380.log
dbfilename slave_6380.rdb
```



Slave 2配置文件（slave-6381.conf）：

```
include /usr/local/bin/redis.conf

port 6381
pidfile /var/run/redis_6381.pid
logfile /usr/local/bin/log/slave_6381.log
dbfilename slave_6381.rdb
```



### 2. 启动Master和Slave数据库

```
redis-server master-6379.conf
redis-server slave-6380.conf
redis-server slave-6381.conf
```

![image-20210829221823668](markdown/Redis主从复制.assets/image-20210829221823668.png)



### 3. 设置Slave指向的Master

1.  连接Slave

    ```
    redis-cli -p 服务器端口号
    ```

2.  设置Slave指向的Master（也可以将命令写入配置文件）

    ```
    slaveof Master地址 Master端口号 
    ```
    
    

Slave1配置Master：

```
root@ubuntu:/usr/local/bin# redis-cli -p 6380
127.0.0.1:6380> slaveof 127.0.0.1 6379
OK
```

Slave2配置Master：

```
root@ubuntu:/usr/local/bin# redis-cli -p 6381
127.0.0.1:6381> slaveof 127.0.0.1 6379
OK
```



### 4. 测试配置效果

1.  查看Master和Slave是否配置正确

    ![image-20210829224614277](markdown/Redis主从复制.assets/image-20210829224614277.png)

2.  查看Master和Slave工作是否正常

    ① 初始时三个库都没有数据

    ![image-20210829224916694](markdown/Redis主从复制.assets/image-20210829224916694.png)

    ②往Master插入若干数据

    ![image-20210829224950177](markdown/Redis主从复制.assets/image-20210829224950177.png)

    ③数据被同步到Slave

    ![image-20210829225057961](markdown/Redis主从复制.assets/image-20210829225057961.png)

    ④无法在Slave插入数据

    ![image-20210829225208803](markdown/Redis主从复制.assets/image-20210829225208803.png)

## 重点问题

### 1. 一主二仆

#### Slave宕机重启

假如重启某个Slave：

-   如果没有在配置文件中指定Master，重启后需要重新手动指定Master

-   Slave重启后会重新同步Master的最新数据

关闭Salve1前：

![image-20210829232039163](markdown/Redis主从复制.assets/image-20210829232039163.png)

关闭Slave1后Master产出若干新数据：

<img src="markdown/Redis主从复制.assets/image-20210829232318649.png" alt="image-20210829232318649" style="zoom:67%;" />

重启Slave1后：由于没有在配置文件设置Master，Slave1重启后并没有自动指向Master（小弟自己成大哥了）

<img src="markdown/Redis主从复制.assets/image-20210829232438609.png" alt="image-20210829232438609" style="zoom:67%;" />

Slave1手动指向Master后：重新从Master同步最新的数据

<img src="markdown/Redis主从复制.assets/image-20210829232700161.png" alt="image-20210829232700161" style="zoom:67%;" />

#### Master宕机重启

假如重启某个Master：

-   Master关闭后，Slave照常提供服务，且继续指向该Master（大哥永远是大哥）
-   Master重启后，一切照旧

### 2. 薪火相传

所有Slave指向同一个Master的情况：

![image-20210829234559588](markdown/Redis主从复制.assets/image-20210829234559588.png)

Slave同样可以作为其他Slave的Master，接受其他Slave的连接和同步请求：

![image-20210829234608866](markdown/Redis主从复制.assets/image-20210829234608866.png)

![image-20210830122850119](markdown/Redis主从复制.assets/image-20210830122850119.png)

优点：降低Master的压力

缺点：一旦某个Slave宕机，后面的Slave就获取不到最新的数据。越末端的Slave数据越不可靠。

![image-20210830123726930](markdown/Redis主从复制.assets/image-20210830123726930.png)

### 3. 反客为主

正常的工作情况：

![image-20210830124848281](markdown/Redis主从复制.assets/image-20210830124848281.png)

此时如果Master宕机，可以通过命令将原本指向它的Slave切换成Master继续工作（大哥死了小弟上位）：



![image-20210830124856297](markdown/Redis主从复制.assets/image-20210830124856297.png)

**演示：**

1.  Master宕机前：6379（Master）←6380（Slave）←6381（Slave）

    ![image-20210830125414904](markdown/Redis主从复制.assets/image-20210830125414904.png)

2.  切换6381为Master

    ```
    slaveof on one
    ```

    ![image-20210830125625070](markdown/Redis主从复制.assets/image-20210830125625070.png)

