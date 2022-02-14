# ACL & 权限控制

Zookeeper 的 ACL（Access Control List，访问控制表）权限在生产环境是特别重要的，ACL 权限可以针对节点设置相关读写等权限，保障数据安全性。



## ACL 命令

-   **getAcl 命令**：获取某个节点的 ACL 权限信息。
-   **setAcl 命令**：设置某个节点的 ACL 权限信息。
-   **addauth 命令**：输入认证授权信息，注册时输入明文密码，加密形式保存。



## ACL 构成

Zookeeper 的 ACL 通过 `[scheme:id:permissions]` 来构成权限列表。

-   **scheme**：代表采用的权限机制，包括 world、auth、digest、ip、super 几种
-   **id**：代表允许访问的用户
-   **permissions**：权限组合字符串，由 cdrwa 组成，其中每个字母代表支持不同权限
    -   创建权限 create(c)
    -   删除权限 delete(d)
    -   读权限 read(r)
    -   写权限 write(w)
    -   管理权限admin(a)



## ACL 实例

https://www.runoob.com/w3cnote/zookeeper-acl.html