[TOC]

# 一、你知道几种定义Bean的方式？

>    JavaBean、SpringBean、对象
>
>   1.   无论是JavaBean还是SpringBean，首先，它们肯定都是对象
>   2.   JavaBean是满足特定规范的类的对象：①所有属性都是private ②通过getter和setter访问属性
>   3.   SpringBean是由Spring框架生成的对象

## 1、\<bean>

1.   创建XML文件，在XML文件中通过<bean>标签定义Bean

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
     
         <!--通过XML文件中的bean标签定义Bean-->
         <bean id="userServiceXml" class="com.xzy.a.UserServiceImpl"/>
     
     </beans>
     ```

2.   Spring容器读取XML文件，根据Bean定义信息创建Bean

     ```java
     public class Main {
         public static void main(String[] args) {
             ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
             UserService userServiceXml = applicationContext.getBean("userServiceXml", UserService.class);
             System.out.println(userServiceXml);
         }
     }
     ```

     

## 2、@Bean

1.   创建配置类，在配置类中通过@Bean定义Bean

     ```java
     public class StudentConfig {
     
         /*
          * 比较通过XML和配置类定义Bean：
          * 1、XML 《==》 配置类
          * 2、<bean> 《==》 @Bean
          */
     
         /*
          * 相比于<bean>，使用@Bean定义Bean更有优势：
          * 1、可以灵活的控制Bean的创建
          * 2、解析XML文件性能更差
          */
     
         /**
          * Bean的ID等于方法名
          */
         @Bean
         public StudentService studentService() {
             // do something to make bean
             return new StudentServiceImpl();
         }
     }
     ```

2.   Spring容器读取配置类，根据Bean定义信息创建Bean

     ```java
     public static void method1() {
         AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
         applicationContext.register(StudentConfig.class);
         applicationContext.refresh();
         
         StudentService studentService = applicationContext.getBean(StudentService.class);
         System.out.println(studentService);
     }
     
     public static void method2() {
         AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(StudentConfig.class);
         
         StudentService studentService = applicationContext.getBean(StudentService.class);
         System.out.println(studentService);
     }
     ```

     

## 3、@Component

1.   使用@Component注解标注目标类

     ```java
     @Component
     public class TeacherServiceImpl implements TeacherService {
     }
     ```

2.   通过XML或者配置类设置组件扫描

3.   Spring容器读取XML或配置类，根据设置进行组件扫描，获取Bean定义信息，创建Bean

     xml：

     ```java
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
     
         <!--组件扫描-->
         <context:component-scan base-package="com.xzy.c"/>
     </beans>
     ```

     配置类：

     ```java
     @ComponentScan(basePackages = "com.xzy.c")
     public class TeacherConfig {
     }
     ```

     Spring容器：

     ```java
     public static void method1() {
         ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("scan.xml");
         TeacherService teacherService = applicationContext.getBean(TeacherService.class);
         System.out.println(teacherService);
     }
     
     public static void method2() {
         AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(TeacherConfig.class);
         TeacherService teacherService = applicationContext.getBean(TeacherService.class);
         System.out.println(teacherService);
     }
     ```



## 4、BeanDefinition ⚠

>   大部分框架的使用方式可以分为2类：声明式、编程式。上面3种定义Bean的方式都属于声明式，本节要说明的BeanDefinitoin则属于编程式。另外，上面3种Bean定义方式的底层都是基于BeanDefinition实现的。
>
>   BeanDefinition包含了Bean的所有描述信息：
>
>   ![image-20211128222440767](markdown/Spring_周瑜.assets/image-20211128222440767.png)



```java
public static void method0() {
    // 1、通过BeanDefinition定义Bean
    AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition();
    beanDefinition.setBeanClass(HumanServiceImpl.class);

    // 2、将BeanDefinition注册到Spring容器
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.registerBeanDefinition("humanServiceImpl", beanDefinition);
    applicationContext.refresh();

    // 3、Spring容器依据BeanDefinition创建Bean
    HumanService bean = applicationContext.getBean(HumanService.class);
    System.out.println(bean);
}

public static void method1() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    // 查看registerBean方法的源码可以发现，最终还是通过BeanDefinition定义Bean
    applicationContext.registerBean(HumanServiceImpl.class);
    applicationContext.refresh();

    HumanService bean = applicationContext.getBean(HumanService.class);
    System.out.println(bean);
}
```



## 5、FactoryBean ⚠

1.   创建实现FactoryBean接口的类

     ```java
     public class PersonFactoryBean implements FactoryBean<PersonService> {
         /**
          * Return an instance (possibly shared or independent) of the object
          * managed by this factory.
          * <p>As with a {@link BeanFactory}, this allows support for both the
          * Singleton and Prototype design pattern.
          * <p>If this FactoryBean is not fully initialized yet at the time of
          * the call (for example because it is involved in a circular reference),
          * throw a corresponding {@link FactoryBeanNotInitializedException}.
          * <p>As of Spring 2.0, FactoryBeans are allowed to return {@code null}
          * objects. The factory will consider this as normal value to be used; it
          * will not throw a FactoryBeanNotInitializedException in this case anymore.
          * FactoryBean implementations are encouraged to throw
          * FactoryBeanNotInitializedException themselves now, as appropriate.
          *
          * @return an instance of the bean (can be {@code null})
          * @throws Exception in case of creation errors
          * @see FactoryBeanNotInitializedException
          */
         @Override
         public PersonService getObject() throws Exception {
             // do something ...
             return new PersonServiceImpl();
         }
     
         /**
          * Return the type of object that this FactoryBean creates,
          * or {@code null} if not known in advance.
          * <p>This allows one to check for specific types of beans without
          * instantiating objects, for example on autowiring.
          * <p>In the case of implementations that are creating a singleton object,
          * this method should try to avoid singleton creation as far as possible;
          * it should rather estimate the type in advance.
          * For prototypes, returning a meaningful type here is advisable too.
          * <p>This method can be called <i>before</i> this FactoryBean has
          * been fully initialized. It must not rely on state created during
          * initialization; of course, it can still use such state if available.
          * <p><b>NOTE:</b> Autowiring will simply ignore FactoryBeans that return
          * {@code null} here. Therefore it is highly recommended to implement
          * this method properly, using the current state of the FactoryBean.
          *
          * @return the type of object that this FactoryBean creates,
          * or {@code null} if not known at the time of the call
          * @see ListableBeanFactory#getBeansOfType
          */
         @Override
         public Class<?> getObjectType() {
             return PersonServiceImpl.class;
         }
     }
     ```

2.   向Spring容器传入FactoryBean的定义

     ```java
     public static void main(String[] args) {
         // 1、通过BeanDefinition定义FactoryBean
         AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition();
         beanDefinition.setBeanClass(PersonFactoryBean.class);
     
         // 2、向Spring容器注册BeanDefinition
         AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
         applicationContext.registerBeanDefinition("person", beanDefinition);
         applicationContext.refresh();
     
         // 3、从Spring容器获取Bean
         PersonService personService = (PersonService) applicationContext.getBean("person");
         PersonFactoryBean personFactoryBean = (PersonFactoryBean) applicationContext.getBean("&person");
     
     
         /*
          * FactoryBean是非常特殊的Bean：Spring容器会根据FactoryBean的BeanDefinition创建两个Bean，
          * 一个是FactoryBean本身，一个是FactoryBean中getObject()方法返回的Bean。
          */
         System.out.println(personService);
         System.out.println(personFactoryBean);
     
         System.out.println(applicationContext.getBean(PersonService.class));
         System.out.println(applicationContext.getBean(PersonFactoryBean.class));
     }
     ```

     FactoryBean非常特殊：表明看传入Spring容器的只有FactoryBean的定义，但是，Spring容器最终会创建两个Bean——FactoryBean本身 + FactoryBean内getObject()方法返回的Bean（挂羊头卖狗肉、明修栈道暗度陈仓、借壳上市）

     >   需要注意：
     >
     >   1.   向Spring容器传入Bean定义的时候，如果没有指定beanName，默认采用首字母小写后的类名
     >   2.   如果没有指定FactoryBean的beanName，需要注意通过FactoryBean首字母小写后的类名获取到的Bean的类型！

     

## 6、Suppiler

```java
public static void method0() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.registerBean(Cat.class);
    applicationContext.refresh();

    // Cat的属性都是空的
    Cat cat = applicationContext.getBean(Cat.class);
    System.out.println(cat);
}

public static void method1() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.registerBean(Cat.class, new Supplier<Cat>() {
        @Override
        public Cat get() {
            // 自定义创建Cat的过程
            Cat cat = new Cat();
            cat.setName("Tom");
            return cat;
        }
    });
    applicationContext.refresh();

    // 由于自定义了填充Cat的过程，因此Cat的属性不是空的
    Cat cat = applicationContext.getBean(Cat.class);
    System.out.println(cat);
}
```



# 二、Spring容器到底是什么鬼？

## 1、单例池、BeanFactory、ApplicationContext

### 1.1 单例模式、单例Bean、单例池

什么是单例模式？

>   简单的说就是通过一些手段使得==某个类在全局只存在一个对象==



什么是单例 Bean ？

>   简单的说就是通过一些手段使得==某个类具有某给名称的对象在 Spring 容器内只有一个==
>
>   ⚠ 单例Bean ≠ 单例模式：简单的说，单例模式更加严格，因为它直接将类的构造方法 private 了，没法手动创建对象；单例Bean则宽松很多——除了由 Spring 容器创建并管理对象，还可以手动创建对象。



什么是单例池？

>   ==单例池存在的意义就是实现单例 Bean==。
>
>   Spring 中单例池的实现：
>
>   ```java
>   ConcurrentHashMap<String,Object> singletonObject; // beanName->singletonBean
>   ```
>
>   单例池添加单例 Bean 的时机：
>
>   -   懒加载单例 Bean —— 初次使用的时候创建并添加
>   -   非懒加载单例 Bean —— Spring 启动的时候创建并添加



### 1.2 BeanFactory

BeanFactory 相当于一个容器 —— 装有 Bean 定义信息（BeanDefiniton）、Bean（单例池）



### 1.3 ApplicationContext

ApplicationContext 间接继承了 BeanFactory，同时还继承了很多其他接口，因此，ApplicationContext 可以替换 BeanFactory，但 ApplicationContext 比 BeanFactory 强大

![image-20211130230632471](markdown/Spring_周瑜.assets/image-20211130230632471.png)



## 2、ApplicationContext 实现类的划分



![image-20211201220933652](markdown/Spring_周瑜.assets/image-20211201220933652.png)



### 划分角度：配置方式

```java
public static void method1() {
    // 基于classpath的相对路径
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("beanConfig.xml");
    System.out.println(applicationContext.getBean(UserService.class));
}

public static void method2() {
    // 基于工程的相对路径
    FileSystemXmlApplicationContext applicationContext1 = new FileSystemXmlApplicationContext("demo2\\src\\main\\resources\\beanConfig.xml");
    System.out.println(applicationContext1.getBean(UserService.class));


    // 文件系统绝对路径
    FileSystemXmlApplicationContext applicationContext2 = new FileSystemXmlApplicationContext("E:\\Programing\\spring-learning\\spring-zhouyu\\demo2\\src\\main\\resources\\beanConfig.xml");
    System.out.println(applicationContext2.getBean(UserService.class));
}

public static void method3() {
    // 基于Java配置类
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanConfig.class);
    System.out.println(applicationContext.getBean(UserService.class));
}
```

>   什么是classpath？
>
>   ![image-20211201221511404](markdown/Spring_周瑜.assets/image-20211201221511404.png)
>
>   
>
>   什么是工程路径？什么是工程相对路径？
>
>   ![image-20211201221714218](markdown/Spring_周瑜.assets/image-20211201221714218.png)



### 划分角度：是否支持刷新（refresh）

```java
/**
 * 支持刷新与不支持刷新的Spring容器
 */
private static void method1() {
    // 支持刷新
    ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("beanConfig.xml");
    classPathXmlApplicationContext.refresh();

    // 不支持刷新
    AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(BeanConfig.class);
    annotationConfigApplicationContext.refresh();

    /*
     * 比较两个类的继承体系：
     *     ClassPathXmlApplicationContext        -> (...) -> AbstractRefreshableApplicationContext -> (...) -> AbstractApplicationContext
     *     AnnotationConfigApplicationContext    -> GenericApplication                                      -> AbstractApplicationContext
     *
     * 查看源码可以发现：
     *     1. refresh() 定义于 AbstractApplicationContext
     *     2. refresh() 调用 refreshBeanFactory() 来刷新 Spring 容器
     *     3. refreshBeanFactory() 是一个抽象方法，交由子类实现
     *     4. GenericApplication 在 refreshBeanFactory() 中禁用了刷新功能；AbstractRefreshableApplicationContext 在 refreshBeanFactory() 中支持了刷新功能
     */
}

/**
 * Spring容器支持刷新会有什么用处？
 */
private static void method2() {
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("beanConfig.xml");

    // 可以观察到刷新后从Spring容器中获取到的单例Bean变了（底层原理：单例池被重新创建）
    System.out.println(applicationContext.getBean(UserService.class));
    System.out.println(applicationContext.getBean(UserService.class));
    System.out.println(applicationContext.getBean(UserService.class));
    applicationContext.refresh();
    System.out.println(applicationContext.getBean(UserService.class));
    System.out.println(applicationContext.getBean(UserService.class));
    System.out.println(applicationContext.getBean(UserService.class));

    /*
     * 更高级一些的功能：在程序的运行过程中，修改XML文件中的Bean定义，然后刷新Spring容器...
     */
}
```

AnnotationConfigApplicationContext 与 ClassPathXmlApplicationContext 的继承体系：

![image-20211201222917644](markdown/Spring_周瑜.assets/image-20211201222917644.png)

从源码层面解释 AnnotationConfigApplicationContext 为什么不支持刷新：

![image-20211201222642531](markdown/Spring_周瑜.assets/image-20211201222642531.png)

# 三、Spring的生命周期

![image-20211207223842358](markdown/Spring_周瑜.assets/image-20211207223842358.png)

# 四、Spring有几种依赖注入的方式？

## 1、 手动注入

### A. \<property/> + Setter





### B. \<constructor-arg/> + Constructor





## 2、 自动注入

### A. \<bean autowire = “xxx”>

### B. @Autowired
