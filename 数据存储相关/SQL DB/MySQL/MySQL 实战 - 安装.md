# MySQL 安装

（Win10 + Docker 环境）

1.   检索 MySQL 镜像

     ![image-20220127211312278](markdown/MySQL 实战 - 安装.assets/image-20220127211312278.png)

2.   拉取 MySQL 镜像

     ![image-20220127211332747](markdown/MySQL 实战 - 安装.assets/image-20220127211332747.png)

2.   创建并启动 MySQL 容器

     ```docker
     docker run -itd -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name MyMySQL mysql
     ```

     ![image-20220127211631772](markdown/MySQL 实战 - 安装.assets/image-20220127211631772.png)

     -   -p 3306:3306：端口映射
     -   -e MYSQL_ROOT_PASSWORD=123456：设置 Mysql root 账号的密码

3.   查看容器是否启动成功

     ![image-20220127211820470](markdown/MySQL 实战 - 安装.assets/image-20220127211820470.png)

4.   查看能够连接 MySQL

     ![image-20220127211910632](markdown/MySQL 实战 - 安装.assets/image-20220127211910632.png)