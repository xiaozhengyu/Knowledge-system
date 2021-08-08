# Redis 6

---

## 1. NoSQL 数据库简介

### 1.1 技术的发展

<img src="markdown/Redis6.assets/image-20210808131622164.png" alt="image-20210808131622164" style="zoom: 50%;" />

技术的分类：

1.  解决功能问题：Java、JSP、Tomcat、Linux、JDBC
2.  解决拓展问题：Struts、Spring、SpringMVC、JPA、MyBatis
3.  解决性能问题：==NoSQL==、MQ、Java线程、Hadoop

无论何种技术，存在的意义都是解决问题。NoSQL就是一种用来解决性能问题的技术，而Redis就是一种典型的NoSQL技术。

#### 1.1.1 Web1.0时代

Web1.0 时代，系统访问量、数据量有限（早期的访问全部来自PC且互联网用户量不多），单体服务器可以满足需求。

![image-20210803211622395](markdown/Redis6.assets/image-20210803211622395.png)

#### 1.1.2 Web2.0时代

Web2.0时代，智能移动设备普及，互联网用户数量剧增，系统访问量、数据量随之大幅提升，单体系统难以支撑，互联网平台面临巨大的挑战。

![image-20210803212043067](markdown/Redis6.assets/image-20210803212043067.png)

##### 1.1.2.1 解决CPU及内存压力

服务器集群化、分布式部署，使用负载均衡将客户端请求均匀的发送给各个服务器，缓解单个服务器的CPU和内存压力。

![image-20210803213248660](markdown/Redis6.assets/image-20210803213248660.png)

但依然存在问题，例如session共享问题。

>   session共享问题：客户在服务器1登录，服务器1创建了session对象。请求被转发到服务器1的时候可以正常处理，请求被转发到服务器2的时候由于服务器2不存在客户的session对象因此无法处理请求。

解决session问题的方案：

1.  将session存储在cookie：不安全、额外的负载

2.  存在文件服务器或数据库：大量的IO

3.  将session复制到所有服务器：数据冗余

4.  **将session存储到缓存数据库：读取速度快（存储在内存，直接读取，不需要IO）、数据结构简单**

    ![image-20210803214342905](markdown/Redis6.assets/image-20210803214342905.png)

##### 1.1.2.2 解决IO压力

读写分离、分库分表：通过破坏一定的业务逻辑来换取性能

**缓存数据库：减少IO操作，例如将热点数据存储缓存数据库（数据存储在内存，避免IO）**

![image-20210803214547858](markdown/Redis6.assets/image-20210803214547858.png)

### 1.2 NoSQL数据库

打破了传统关系型数据库以业务逻辑为依据的存储模式，而针对不同数据结构类型改为以性能为依据的存储方式。

#### 1.2.1 NoSQL概述

NoSQL = <font color = red>Not Only SQL</font>，泛指所有非关系数据。NoSQL不依赖业务逻辑存储，而以简单的<font color = red>key-value</font>模式存储，因此大大的增加的数据库的**拓展能力**。

>   理解：假设某个已上线的系统现有3亿用户，如果使用关系型数据库存储用户信息，此时想要往用户表加一个字段，这是非常恐怖的！如果是NoSQL数据库，情况就会好很多。

-   不遵循SQL标准
-   不支持ACID（不支持ACID不意味着不支持事务）
-   性能远超于SQL

#### 1.2.2 适用、不适用场景

适合：

-   高并发读写（电商秒杀）
-   海量数据读写
-   数据高可拓展性

不适合：

-   需求事务支持
-   需要结构化查询

#### 1.2.3 常见NoSQL数据库

1.  Memcache
    -   开源
    -   很早出现的NoSQL数据库
    -   支持的value类型比较单一
    -   数据存储在内存，==不支持持久化==
    -   一般作为缓存数据库辅助持久化数据库
    -   多线程 + 锁
2.  Redis
    -   开源
    
    -   几乎涵盖了Memcache的绝大部分功能
    
    -   支持多种数据结构的value，例如string、list、hash、set、zset
    
    -   数据存储在内存，==支持持久化==（周期性的把更新的数据写入磁盘，或者把修改操作写入记录文件，主要用于备份恢复），在此基础上实现了==Redis主从复制==
    
    -   一般作为缓存数据库辅助持久化数据库
    
    -   单线程 + 多路IO复用
    
        ![image-20210808164238093](markdown/Redis6.assets/image-20210808164238093.png)
3.  MongoDB
    -   文档型数据库

## 2. Redis 概述、安装

### 2.1 应用场景

-   高速缓存
    -   配合关系型数据库，存储高频热点数据，降低数据库IO
    -   配合分布式结构，实现内存数据共享，如分布式Session
-   数据结构
    -   最新N个数据：通过List实现按照自然时间排序的数据
    -   排行榜：利用zSet（有序集合）
    -   失效性数据，如短信验证码：Expire过期
    -   计数器、秒杀：原子性
    -   去重：利用Set
    -   发布订阅消息系统：pub/sub模式

### 2.2 Redis安装

#### 2.2.1 安装版本

[官网下载地址](https://redis.io/)

![image-20210808142233370](markdown/Redis6.assets/image-20210808142233370.png)

#### 2.2.2 安装步骤

1.  准备工作：下载安装最新版本的gcc编译器

    ```shell
    # 查看当前系统是否安装、安装了什么版本的gcc编译器
    gcc --version
    ```

    ```shell
    # 安装
    yum install centos-release-scl scl-utils-build
    yum install -y devtoolset-8-toolchain
    scl enable devtoolset-8 bash
    ```

2.  上传redis文件至服务器

    ![image-20210808143317088](markdown/Redis6.assets/image-20210808143317088.png)

3.  解压文件

    ```shell
    tar -zxvf redis-6.2.5.tar.gz

4.  进入目录

    ```shell
    cd redis-6.2.5
    ```

    ![image-20210808143914571](markdown/Redis6.assets/image-20210808143914571.png)

5.  编译

    ```shell
    make
    ```

    ![image-20210808144130316](markdown/Redis6.assets/image-20210808144130316.png)

6.  安装

    ```shell
    # 跳过 make test 直接进行安装
    make install
    ```

    ![image-20210808144239570](markdown/Redis6.assets/image-20210808144239570.png)

#### 2.2.3 安装目录

/usr/local/bin

![image-20210808144408052](markdown/Redis6.assets/image-20210808144408052.png)

-   redis-benchmark：性能测试工具
-   redis-check-aof：用于修复有问题的aof文件
-   redis-check-rdb：用于修复有问题的rdb文件
-   redis-sentinel：redis集群使用
-   ==redic-cli==：客户端，操作入口
-   ==redis-server==：redis服务器启动命令

#### 2.2.4 前台启动（不推荐）

以前台方式启动，命令行窗口关闭则redis服务随着关闭。

```shell
redis-server
```

![image-20210808145249184](markdown/Redis6.assets/image-20210808145249184.png)

#### 2.2.5 后台启动（推荐）

1.  进入redis解压目录，复制redis配置文件

    ```shell
    cd /opt/redis-6.2.5/
    cp redis.conf /usr/local/bin/my-config.conf
    ```

2.  修改配置文件

    修改配置文件中的daemonize配置项目，将原来的no改为yes

    ![image-20210808151301421](markdown/Redis6.assets/image-20210808151301421.png)

3.  启动Redis

    ```shell
    # 启动redis
    redis-server my-config.conf
    # 查看启动状态
    ps -ef | grep redis
    ```

    ![image-20210808151512920](markdown/Redis6.assets/image-20210808151512920.png)

4.  测试验证

    ```shell
    # 客户端连接redis
    redis-cli
    ```

    ![image-20210808151811533](markdown/Redis6.assets/image-20210808151811533.png)

5.  关闭Redis

    方案1：通过客户端，使用shutdown命令

    ```
    # 单实例关闭
    redis-cli shutdown
    
    # 多实例关闭
    redis-cli -p 6379 shutdown
    ```

    方案2：找到redis进程号，使用kill -9 关闭

#### 2.2.6 Redis相关知识

-   默认端口6379

-   默认16个数据库，下标从0开始。默认使用0号数据库，使用 select <dbid> 切换数据库。

    ![image-20210808162920157](markdown/Redis6.assets/image-20210808162920157.png)

## 3. 五大常用数据类型

Redis官网对Redis命令、Redis数据类型有详细的介绍：

-   命令
    -   [Redis命令中心（英文）](https://redis.io/commands)
    -   [Redis命令中心（中文）](http://www.redis.cn/commands.html)
-   数据类型
    -   [数据类型（英文）](https://redis.io/topics/data-types-intro)
    -   [数据类型（中文）](http://www.redis.cn/topics/data-types.html)

### 3.1 键（key）

[常见命令](http://www.redis.cn/commands.html#generic)

### 3.2 字符串（String）

#### 3.2.1 简介



#### 3.2.2 常用命令

#### 3.2.3 数据结构



### 3.3 列表（List）

#### 3.3.1 简介

#### 3.3.2 常用命令

#### 3.3.3 数据结构



### 3.4 集合（Set）

#### 3.4.1 简介

#### 3.4.2 常用命令

#### 3.4.3 数据结构



### 3.5 哈希（Hash）

#### 3.5.1 简介

#### 3.5.2 常用命令

#### 3.5.3 数据结构



### 3.6 有序集合（ZSet）

#### 3.6.1 简介

#### 3.6.2 常用命令

#### 3.6.3 数据结构
