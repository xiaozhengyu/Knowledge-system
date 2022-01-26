# Docker 实战 - 局域网络搭建

## 1、docker network 命令

```
PS C:\Users\xiaozy> docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks # 显示局域网详细信息
  ls          List networks                                        # 罗列所有局域网
  prune       Remove all unused networks
  rm          Remove one or more networks
```



## 2、创建局域网

创建局域网：

```
PS C:\Users\xiaozy37528> docker network create myNetwork
f26e19d32597aec0a762c2e7081f86de930aa885b90c983a78730af2a6a018e6
```

罗列所有局域网：

```
PS C:\Users\xiaozy37528> docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
cb5ee1a12afa   bridge            bridge    local
9b5cbe43e4fb   example_default   bridge    local
26e5ebb8f04c   host              host      local
f26e19d32597   myNetwork         bridge    local
ccc51ee1d166   none              null      local
```

查看局域网详细信息：

```
PS C:\Users\xiaozy37528> docker network inspect myNetwork
[
    {
        "Name": "myNetwork",
        "Id": "f26e19d32597aec0a762c2e7081f86de930aa885b90c983a78730af2a6a018e6",
        "Created": "2022-01-26T03:11:03.8063259Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```



## 3、容器连接到局域网

将容器连接到局域网有两种方式：

1.   启动容器时加入

     ```
     docker run -itd --name myRedis --network myNetwork --network-alias redis -p 6379:6379 redis
     ```

2.   启动容器后加入

     ```
     docker network connect myNetwork redis
     ```

     

     ```
     PS C:\Users\xiaozy37528> docker container ls
     CONTAINER ID   IMAGE                COMMAND                  CREATED      STATUS          PORTS                                       NAMES
     85f3be5e8ee0   nginx                "/docker-entrypoint.…"   2 days ago   Up 33 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp           MyNginx
     c824254e2a94   nacos/nacos-server   "bin/docker-startup.…"   6 days ago   Up 2 hours      0.0.0.0:8848->8848/tcp, :::8848->8848/tcp   nacos
     PS C:\Users\xiaozy37528> docker network connect myNetwork MyNginx
     PS C:\Users\xiaozy37528> docker network connect myNetwork nacos
     PS C:\Users\xiaozy37528> docker network inspect myNetwork
     [
         {
             "Name": "myNetwork",
             "Id": "f26e19d32597aec0a762c2e7081f86de930aa885b90c983a78730af2a6a018e6",
             "Created": "2022-01-26T03:11:03.8063259Z",
             "Scope": "local",
             "Driver": "bridge",
             "EnableIPv6": false,
             "IPAM": {
                 "Driver": "default",
                 "Options": {},
                 "Config": [
                     {
                         "Subnet": "172.19.0.0/16",
                         "Gateway": "172.19.0.1"
                     }
                 ]
             },
             "Internal": false,
             "Attachable": false,
             "Ingress": false,
             "ConfigFrom": {
                 "Network": ""
             },
             "ConfigOnly": false,
             "Containers": {
                 "85f3be5e8ee00b51ce5cf1cf88e105cdffe2ea312bf8a9a30d0f3343a582c231": {
                     "Name": "MyNginx",
                     "EndpointID": "d9a7a59c21221a96b626435ac77facd382de425a51513ef61ca7d8b8aa7c6843",
                     "MacAddress": "02:42:ac:13:00:02",
                     "IPv4Address": "172.19.0.2/16",
                     "IPv6Address": ""
                 },
                 "c824254e2a94bd32a5c216a0978e4ef7959cce6331a9ac220d75debd4593666c": {
                     "Name": "nacos",
                     "EndpointID": "a2119d6c6742bd886ec19e910c207c20011e532d25f9f8afb0b2f59533fda13d",
                     "MacAddress": "02:42:ac:13:00:03",
                     "IPv4Address": "172.19.0.3/16",
                     "IPv6Address": ""
                 }
             },
             "Options": {},
             "Labels": {}
         }
     ]
     ```

     

## 4、验证

Nacos 容器 ping Nginx 容器：

```
PS C:\Users\xiaozy37528> docker exec -it nacos /bin/bash
[root@c824254e2a94 nacos]# ping 172.19.0.2
PING 172.19.0.2 (172.19.0.2) 56(84) bytes of data.
64 bytes from 172.19.0.2: icmp_seq=1 ttl=64 time=0.142 ms
64 bytes from 172.19.0.2: icmp_seq=2 ttl=64 time=0.038 ms
clea64 bytes from 172.19.0.2: icmp_seq=3 ttl=64 time=0.050 ms
64 bytes from 172.19.0.2: icmp_seq=4 ttl=64 time=0.037 ms
64 bytes from 172.19.0.2: icmp_seq=5 ttl=64 time=0.053 ms
64 bytes from 172.19.0.2: icmp_seq=6 ttl=64 time=0.037 ms
64 bytes from 172.19.0.2: icmp_seq=7 ttl=64 time=0.039 ms
64 bytes from 172.19.0.2: icmp_seq=8 ttl=64 time=0.033 ms
64 bytes from 172.19.0.2: icmp_seq=9 ttl=64 time=0.040 ms
64 bytes from 172.19.0.2: icmp_seq=10 ttl=64 time=0.039 ms
64 bytes from 172.19.0.2: icmp_seq=11 ttl=64 time=0.050 ms
64 bytes from 172.19.0.2: icmp_seq=12 ttl=64 time=0.041 ms
64 bytes from 172.19.0.2: icmp_seq=13 ttl=64 time=0.052 ms
64 bytes from 172.19.0.2: icmp_seq=14 ttl=64 time=0.091 ms
64 bytes from 172.19.0.2: icmp_seq=15 ttl=64 time=0.049 ms
64 bytes from 172.19.0.2: icmp_seq=16 ttl=64 time=0.041 ms
64 bytes from 172.19.0.2: icmp_seq=17 ttl=64 time=0.055 ms
64 bytes from 172.19.0.2: icmp_seq=18 ttl=64 time=0.037 ms
--- 172.19.0.2 ping statistics ---
18 packets transmitted, 18 received, 0% packet loss, time 17692ms
rtt min/avg/max/mdev = 0.033/0.051/0.142/0.026 ms
[root@c824254e2a94 nacos]#
```

Nginx 容器 ping Nacos 容器：

