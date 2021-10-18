# Docker + RabbitMQ + 延迟队列插件

---

## 1. 获取Docker容器

详见[RabbitMQ安装与使用.md](./RabbitMQ安装与使用.md)



## 2. 安装延迟队列插件

### 2.1 确认RabbitMQ版本

![image-20210929212106825](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929212106825.png)



### 2.2 下载插件

下载与安装的RabbitMQ匹配的插件：[rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases)

![image-20210929220114426](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929220114426.png)

### 2.3 安装插件

#### 2.3.1 将插件从宿主机拷贝至容器

```bash
docker cp 宿主机文件 容器名称或ID:容器目录
```

![image-20210929220700728](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929220700728.png)



#### 2.3.2 进入容器

```bash
 docker exec -it 容器名称或ID /bin/bash
```

![image-20210929220800109](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929220800109.png)

可以看到插件已经成功拷贝到容器的指定目录



#### 2.3.3 查看插件列表

```bash
rabbitmq-plugins list
```

![image-20210929221152955](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929221152955.png)



#### 2.3.4 启用插件

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

![image-20210929221410307](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929221410307.png)



#### 2.3.5 重启容器

```
docker restart 容器名称或ID
```

![image-20210929221540847](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929221540847.png)



#### 2.3.6 验证结果

![image-20210929221701345](markdown/Docker + RabbitMQ + 延迟队列插件.assets/image-20210929221701345.png)

