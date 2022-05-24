# Docker 网络

官方文档：https://docs.docker.com/network/

---

[toc]

---

## Networking overview



### Network drivers

Docker 的网络子系统是可插拔的。Docker 提供了多种网络驱动（Network drivers）给我们选择，不同的网络驱动具有不同的特点：

-   `bridge`：bridge 是 Docker 默认的网络驱动。当应用运行在独立的容器并且需要相互通讯，通常会采用 bridge 网络。
-   `overlay`：
-   `host`：
-   `ipvlan`：
-   `macvlan`：
-   `none`：
-   `Network plugins`：



### Network driver summary

-   当需要运行在同一个 Docker 主机上的多个容器相互通讯，最好使用自定义的 bridge 网络

-   当需要运行在不同 Docker 主机上的容器相互通讯，或者当多个应用以 swarm 的方式一起工作时，最好使用 overlay 网络

    >   Swarm 在 Docker 1.12 版本之前属于一个独立的项目，在 Docker 1.12 版本发布之后，该项目合并到了 Docker 中，成为 Docker 的一个子命令。目前，Swarm 是 Docker 社区提供的唯一一个原生支持 Docker 集群管理的工具。它可以把多个 Docker 主机组成的系统转换为单一的虚拟 Docker 主机，使得容器可以组成跨主机的子网网络。



## Use bridge networks

## Use overlay networks

## Use host networks

## Use IPvlan networks

## Use Macvlan networks

## Disable networking for a containers

## Networking tutorials

