# Spring 声明式事务的传播行为

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



## 什么是事务传播行为？

Spring 在 TransactionDefinition 接口中规定了 7 种类型的事务传播行为。<u>事务的传播行为是 Spring 框架独有的事务增强特性，它不属于事务实际提供方的数据库行为。</u><(￣︶￣)↗[GO!](https://segmentfault.com/a/1190000013341344)

事务传播行为用来描述：由某一个事务传播行为修饰的方法，被嵌套进另一个方法时，事务如何传播。

用伪代码说明：

```java
 public void methodA(){
    methodB();
    //doSomething
 }
 
 @Transaction(Propagation=XXX)
 public void methodB(){
    //doSomething
 }
```

代码中 methodA() 嵌套调用了 methodB()，method() 通过 @Transaction(Propagation=XXX) 配置了事务传播行为。这里需要注意的是，methodA() 并没有开启事务，某一个事务传播行为修饰的方法，并不是必须在开启事务的外围方法中调用。



## Spring 的七种事务传播行为



-   ==REQUIRED==

    当前方法【必须】在事务中执行。外部有事务，则加入事务；外部无事务，则自建事务

    

-   ==REQUIRES_NEW==

    当前方法【必须】新建事务执行。外部有事务，则挂起外部事务

    

-   ==SUPPORTS==

    当前方法【可以】在事务中执行。外部有事务，则加入事务；外部无事务，则照常执行

    

-   ==NOT_SUPPORTS==

    当前方法【不能】在事务中执行。外部有事务，则将事务挂起以无事务状态执行

    

-   ==MANDATORY==

    当前方法【必须】在事务中执行。外部有事务，则加入事务；外部无事务，则抛出异常

    

-   ==NEVER==

    当前方法【不能】在事务中执行。外部有事务，则抛出异常

    

-   ==NESTED==

    外部有事务，则作为这个事务的内嵌事务执行；外部无事务，则新建事务执行



## 代码验证



建表：

```mysql
CREATE TABLE `user1` (
  `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL DEFAULT '',
  PRIMARY KEY(`id`)
)
```

```mysql
CREATE TABLE `user2` (
  `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL DEFAULT '',
  PRIMARY KEY(`id`)
)
```



### 1. REQUIRED

User1ServiceImpl：

```java
@Service
public class User1ServiceImpl implements User1Service {
    //省略其他...
    
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequired(User1 user){
        user1Mapper.insert(user);
    }
    
}
```

User2ServiceImpl：

```java
@Service
public class User2ServiceImpl implements User2Service {
    //省略其他...
    
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequired(User2 user){
        user2Mapper.insert(user);
    }
    
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequiredException(User2 user){
        user2Mapper.insert(user);
        throw new RuntimeException(); // 抛出异常
    }
    
}
```



#### 场景一：外部无事务

验证方法1：

```java
    @Override // 外围方法没有事务
    public void notransaction_exception_required_required(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入成功
        user2Service.addRequired(user2);
        
        throw new RuntimeException(); // 外围方法抛出异常
    }

// 外围方法没有事务，插入张三和插入李四的方法各自创建事务执行，事务不受外围方法影响
```

验证方法2：

```java
    @Override // 外围方法没有事务
    public void notransaction_required_required_exception(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        user2Service.addRequiredException(user2); // 抛出异常，回滚
    }

// 外围方法没有事务，插入张三和插入李四的方法各自创建事务执行，事务间相互独立，互补影响
```



#### 场景二：外部有事务

验证方法1：

```java
    @Override // 外围方法开启事务
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_exception_required_required(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        user2Service.addRequired(user2);
        
        throw new RuntimeException(); // 外围方法抛出异常
    }

// 外围方法开启事务，内部方法加入该事务一起执行，一起提交，一起回滚
```

验证方法2：

```java
    @Override // 外围方法开启事务
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_required_required_exception(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        user2Service.addRequiredException(user2); // 内部方法抛出异常
    }

// 外围方法开启事务，内部方法加入该事务一起执行，一起提交，一起回滚。addRequiredException()内部抛出异常并被外围方法感知，
// 导致外围方法的事务回滚。
```

验证方法3：

```java
    @Transactional
    @Override // 外围方法开启事务
    public void transaction_required_required_exception_try(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        try {
            user2Service.addRequiredException(user2); // 外围方法捕获异常
        } catch (Exception e) {
            System.out.println("方法回滚");
        }
    }

// 外围方法开启事务，内部方法加入该事务一起执行，一起提交，一起回滚。addRequiredException()内部抛出异常，即使不被外围方法感知
// 整个事务依然回滚。
```



### 2. REQUIRES_NEW

User1ServiceImpl：

```java
@Service
public class User1ServiceImpl implements User1Service {
    //省略其他...
    
    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiresNew(User1 user){
        user1Mapper.insert(user);
    }
    
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequired(User1 user){
        user1Mapper.insert(user);
    }
}
```

User2ServiceImpl：

```java
@Service
public class User2ServiceImpl implements User2Service {
    //省略其他...
    
    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiresNew(User2 user){
        user2Mapper.insert(user);
    }
    
    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiresNewException(User2 user){
        user2Mapper.insert(user);
        throw new RuntimeException();
    }
}
```



#### 场景一：外部无事务

验证方法1：

```java
    @Override
    public void notransaction_exception_requiresNew_requiresNew(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addRequiresNew(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入成功
        user2Service.addRequiresNew(user2);
        throw new RuntimeException(); // 抛出异常
    }

// 插入张三和李四的方法各自创建事务独立执行，不受外围方法影响
```

验证方法2：

```java
    @Override
    public void notransaction_exception_requiresNew_requiresNew(){
        User1 user1=new User1();
        user1.setName("张三");// 插入成功
        user1Service.addRequiresNew(user1);
        
        User2 user2=new User2();
        user2.setName("李四");// 插入失败
        user2Service.addRequiresNewException(user2);
        throw new RuntimeException();
    }
// 插入张三和李四的方法各自创建事务独立执行，互不影响
```



#### 场景二：外部有事务

**验证方法1：**

```java
    @Override
    @Transactional(propagation = Propagation.REQUIRED) // 外围开启事务
    public void transaction_exception_required_requiresNew_requiresNew(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入成功
        user2Service.addRequiresNew(user2);
        
        User2 user3=new User2();
        user3.setName("王五"); // 插入成功
        user2Service.addRequiresNew(user3);

        throw new RuntimeException(); // 抛出异常，导致外围事务回滚
    }

// 插入张三的方法加入外围方法的事务，跟着外围方法一起回滚
// 插入李四、王五的方法各自创建事务独立执行，不受外围方法影响
```

**验证方法2：**

```java
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_required_requiresNew_requiresNew_exception(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入成功
        user2Service.addRequiresNew(user2);
        
        User2 user3=new User2();
        user3.setName("王五"); // 插入失败
        user2Service.addRequiresNewException(user3);
    }

// 插入张三、王五的方法加入外围方法的事务，由于插入王五的方法抛出异常，导致外围事务回滚
// 插入李四的方法自己创建事务执行，不受外围方法影响
```

**验证方法3：**

```java
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_required_requiresNew_requiresNew_exception_try(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addRequired(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入成功
        user2Service.addRequiresNew(user2);
        
        User2 user3=new User2();
        user3.setName("王五"); // 插入失败
        try {
            user2Service.addRequiresNewException(user3);
        } catch (Exception e) {
            System.out.println("回滚");
        }
    }

// 插入张三、李四的方法加入外围方法的事务，插入王五的方法自己创建独立的事务
// 插入王五的事务发生回滚不会影响外围事务
```



### 3. SUPPORTS

#### 场景一：外部无事务

#### 场景二：外部有事务

### 4. NOT_SUPPORTS

#### 场景一：外部无事务

#### 场景二：外部有事务

### 5. MANDATORY

#### 场景一：外部无事务

#### 场景二：外部有事务

### 6. NEVER

#### 场景一：外部无事务

#### 场景二：外部有事务

### 7. NESTED

User1ServiceImpl：

```java
@Service
public class User1ServiceImpl implements User1Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addNested(User1 user){
        user1Mapper.insert(user);
    }
}
```

User2ServiceImpl：

```java
@Service
public class User2ServiceImpl implements User2Service {
    //省略其他...
    
    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addNested(User2 user){
        user2Mapper.insert(user);
    }
    
    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addNestedException(User2 user){
        user2Mapper.insert(user);
        throw new RuntimeException();
    }
}
```



#### 场景一：外部无事务

验证方法1：

```java
    @Override // 外围方法无事务
    public void notransaction_exception_nested_nested(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addNested(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入成功
        user2Service.addNested(user2);
        
        throw new RuntimeException();
    }

// 外围方法没有事务，插入张三、李四的方法在自己创建的事务中执行，不受外围方法异常的影响
```

验证方法2：

```java
    @Override // 外围方法无事务
    public void notransaction_nested_nested_exception(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addNested(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        user2Service.addNestedException(user2);
    }

// 外围方法没有事务，插入张三、李四的方法在自己创建的事务中执行，相互独立，互不影响
```



#### 场景二：外部有事务

验证方法1：

```java
    @Transactional
    @Override
    public void transaction_exception_nested_nested(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addNested(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        user2Service.addNested(user2);
        
        throw new RuntimeException();
    }

// 插入张三、李四的方法创建外围事务的子事务，外围事务回滚，子事务也要回滚
```

验证方法2：

```java
    @Transactional
    @Override
    public void transaction_nested_nested_exception(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入失败
        user1Service.addNested(user1);
        
        User2 user2=new User2();
        user2.setName("李四"); // 插入失败
        user2Service.addNestedException(user2);
    }

// 插入张三、李四的方法创建外围事务的子事务，外围事务回滚，子事务也要回滚
```

验证方法3：

```java
    @Transactional
    @Override
    public void transaction_nested_nested_exception_try(){
        User1 user1=new User1();
        user1.setName("张三"); // 插入成功
        user1Service.addNested(user1);
        
        User2 user2=new User2();
        user2.setName("李四");
        try {
            user2Service.addNestedException(user2); // 插入失败
        } catch (Exception e) {
            System.out.println("方法回滚");
        }
    }
// 插入张三、李四的方法创建外围事务的子事务，插入李四的子事务单独回滚，外围事务以及插入张三的子事务不受影响
```