# Spring 声明式事务

[官方文档：17.5.6 Using @Transactional](https://docs.spring.io/spring-framework/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#transaction-declarative-annotations)

## @Transactional 注解常用属性

-   value

    用于限定事务管理器

-   propagation

    事务的传播行为

-   isolation

    事务的隔离级别。只有 propagation 被设置为 REQUIRED 或 REQUIRES_NEW 时有效

-   timeout

    事务的超时时间。只有 propagation 被设置为 REQUIRED 或 REQUIRES_NEW 时有效

-   readOnly

    当前事务是否只包含读操作。只有 propagation 被设置为 REQUIRED 或 REQUIRES_NEW 时有效

-   rollbackFor

    抛出哪些异常时事务必须回滚

-   noRollbackFor

    抛出哪些异常时事务不要回滚



## 事务的传播行为

Spring 在 TransactionDefinition 接口中规定了 7 种类型的事务传播行为。事务的传播行为是 Spring 框架独有的事务增强特性，它不属于事务实际提供方的数据库行为。<(￣︶￣)↗[GO!](https://segmentfault.com/a/1190000013341344)



## 事务的隔离级别



