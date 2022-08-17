# 使用 ConfigFilter 实现数据库连接信息加密

>   官方文档：https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter



使用明文的数据库连接信息：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/hello_world?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: xzy
    password: 123456
```

数据库密码直接写在配置中，对运维安全来说，是一个很大的挑战。Druid 为此提供一种数据库密码加密的手段 ConfigFilter。

ConfigFilter 的作用包括：

1.   从配置文件读取配置
2.   从远程 http 文件读取配置
3.   **为数据库密码提供加解密功能**



### 1、加密数据库密码

在命令行中执行下列命令：

```shell
java -cp druid-1.0.16.jar com.alibaba.druid.filter.config.ConfigTools YOUR_PASSWORD
```

输出：私钥、公钥、加密后的数据库密码

```
privateKey:MIIBVgIBADANBgkqhkiG9w0BAQEFAASCAUAwggE8AgEAAkEA6+4avFnQKP+O7bu5YnxWoOZjv3no4aFV558HTPDoXs6EGD0HP7RzzhGPOKmpLQ1BbA5viSht+aDdaxXp6SvtMQIDAQABAkAeQt4fBo4SlCTrDUcMANLDtIlax/I87oqsONOg5M2JS0jNSbZuAXDv7/YEGEtMKuIESBZh7pvVG8FV531/fyOZAiEA+POkE+QwVbUfGyeugR6IGvnt4yeOwkC3bUoATScsN98CIQDynBXC8YngDNwZ62QPX+ONpqCel6g8NO9VKC+ETaS87wIhAKRouxZL38PqfqV/WlZ5ZGd0YS9gA360IK8zbOmHEkO/AiEAsES3iuvzQNYXFL3x9Tm2GzT1fkSx9wx+12BbJcVD7AECIQCD3Tv9S+AgRhQoNcuaSDNluVrL/B/wOmJRLqaOVJLQGg==
publicKey:MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAOvuGrxZ0Cj/ju27uWJ8VqDmY7956OGhVeefB0zw6F7OhBg9Bz+0c84RjzipqS0NQWwOb4kobfmg3WsV6ekr7TECAwEAAQ==
password:PNak4Yui0+2Ft6JSoKBsgNPl+A033rdLhFw+L0np1o+HDRrCo9VkCuiiXviEMYwUgpHZUFxb2FpE0YmSguuRww==
```



### 2、配置 ConfigFilter

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
     <property name="filters" value="config" />
     <property name="url" value="jdbc:mysql://127.0.0.1:3306/hello_world?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8&useSSL=false" />
     <property name="username" value="xzy" />
     <property name="password" value="PNak4Yui0+2Ft6JSoKBsgNPl+A033rdLhFw+L0np1o+HDRrCo9VkCuiiXviEMYwUgpHZUFxb2FpE0YmSguuRww==" /> <!--加密后的密码-->
     <property name="connectionProperties" value="config.decrypt=true;config.decrypt.key=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAOvuGrxZ0Cj/ju27uWJ8VqDmY7956OGhVeefB0zw6F7OhBg9Bz+0c84RjzipqS0NQWwOb4kobfmg3WsV6ekr7TECAwEAAQ==" /> <!--公钥-->
</bean>
```

>   关于 ConfigFilter 配置：
>
>   1.   从本地文件系统获取配置文件
>
>        ```xml
>         <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
>             <property name="filters" value="config" />
>             <property name="connectionProperties" value="config.file=file:///home/admin/druid-pool.properties" />
>         </bean>
>        ```
>
>        
>
>   2.   从远程 HTTP 服务器获取配置文件
>
>        ```xml
>         <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
>             <property name="filters" value="config" />
>             <property name="connectionProperties" value="config.file=http://127.0.0.1/druid-pool.properties" />
>         </bean>
>        ```
>
>        这种配置方式，使得一个应用集群中，多个实例可以从同一个地方读取配置，集中配置，集中修改，部署更简单。
>
>        
>
>   3.   通过 JVM 启动参数配置 filter
>
>        ```
>        java -Ddruid.filters=config ....
>        ```



### 3、开启解密功能

有三种方式配置：

1) 在配置文件 xxx.properties 中指定 config.decrypt=true
2) 在 DruidDataSource 的 ConnectionProperties 中指定 config.decrypt=true <br/>
3) 在 JVM 启动参数中指定 -Ddruid.config.decrypt=true