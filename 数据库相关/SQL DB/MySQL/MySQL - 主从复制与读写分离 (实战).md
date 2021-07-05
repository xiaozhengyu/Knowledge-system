# MySQL - 主从复制与读写分离 (实战)

（尽量确保数据库版本一致）

#### 1. 关闭防火墙，确保Master和Slave所在的服务器能够相互ping通

>   Ubuntu查看防火墙状态、关闭防火墙、开启防火墙：sudo ufw status、sudo ufw disable、sudo ufw enable

![image-20210705093933856](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705093933856.png)

#### 2. 修改Master和Slave的配置信息，并重启数据库

>   配置文件：vi /etc/mysql/my.cnf
>
>   重启数据库：service mysql restart

Master配置：

![image-20210705141051732](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705141051732.png)

Slave配置：

![image-20210705141126697](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705141126697.png)

<mark>Master 和 Slave 的server-id 必须不同！</mark>

#### 3. 在Master创建Slave账号，并授予权限

```mysql
-- 创建Slave账号
CREATE USER 'lisi'@'%' IDENTIFIED BY '123456';

-- Slave账号授权
GRANT replication SLAVE ON *.* TO 'lisi'@'%';

-- 刷新系统权限
FLUSH PRIVILEGES;
```

#### 4. 在Slave测试刚才创建的账号

```shell
mysql -u账号 -p密码 -h地址
```

![image-20210705141701574](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705141701574.png)

>   实现这一步时出现了错误：ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.109.128' (111)
>
>   解决方法：修改Master配置信息，注释掉mysql的ip绑定，重启mysql
>
>   ![image-20210705142244518](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705142244518.png)

#### 5. 查看Master状态

>   注意：查询到的File和Position的值会随着数据库变动而发生该表，而后续启动Slave又需要这两个值，所以查到数据以后最好马上配置Slave，之后如果需要重新配置Slave也需要重新查询这两个值。

![image-20210705142750267](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705142750267.png)

#### 6. 启动Slave

```mysql
-- 关闭slave
STOP SLAVE;

-- 配置Master（一个Slave只能有一个Master，一个Master可以有多个Slave）
CHANGE MASTER TO MASTER_HOST = '192.168.109.128',
                 MASTER_USER = 'lisi',
                 MASTER_PASSWORD = '123456',
                 MASTER_LOG_FILE = 'mysql-bin.000003',
				 MASTER_LOG_POS = 700
-- 启动Slave
START SLAVE;

-- 查看Slave启动状态
SHOW SLAVE STATUS;
```

![image-20210705143436336](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705143436336.png)

只有Slave_IO_Running和Slave_SQL_Running两项的值都为Yes时，才能表明Slave已经成功开始工作。如果Slave没有正常启动，多关注一下Last_IO_Error项显示的错误信息。

#### 7. 测试

在Master进行编辑操作，验证Slave是否同步了相关操作。

![image-20210705145423028](markdown/MySQL - 主从复制与读写分离 (实战).assets/image-20210705145423028.png)

