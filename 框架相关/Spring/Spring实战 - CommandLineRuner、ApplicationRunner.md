# CommandLineRunner、ApplicationRunner

参考文章：[Spring Boot使用CommandLineRunner接口完成资源初始化](https://blog.csdn.net/lk142500/article/details/90270592)

[toc]

### 1、应用场景

需要在生成所有 Bean 之后进行某些操作，例如，加载数据、删除临时文件、清除缓存、读取配置文件、数据库连接（类似于开机自启动应用）。通过 CommandLineRunner、ApplicationRuner 接口能够实现这些需求：容器启动的最后阶段，处理所有 CommandLineRuner、ApplicationRunner 接口的实例（相当于设置了一个回调）

### 2、接口源码

CommandLineRuner 源码：

```java
/**
 * Interface used to indicate that a bean should <em>run</em> when it is contained within
 * a {@link SpringApplication}. Multiple {@link CommandLineRunner} beans can be defined
 * within the same application context and can be ordered using the {@link Ordered}
 * interface or {@link Order @Order} annotation.
 * <p>
 * If you need access to {@link ApplicationArguments} instead of the raw String array
 * consider using {@link ApplicationRunner}.
 *
 * @author Dave Syer
 * @since 1.0.0
 * @see ApplicationRunner
 */
@FunctionalInterface
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
```

ApplicationRunner 源码：

```java
/**
 * Interface used to indicate that a bean should <em>run</em> when it is contained within
 * a {@link SpringApplication}. Multiple {@link ApplicationRunner} beans can be defined
 * within the same application context and can be ordered using the {@link Ordered}
 * interface or {@link Order @Order} annotation.
 *
 * @author Phillip Webb
 * @since 1.3.0
 * @see CommandLineRunner
 */
@FunctionalInterface
public interface ApplicationRunner {

   /**
    * Callback used to run the bean.
    * @param args incoming application arguments
    * @throws Exception on error
    */
   void run(ApplicationArguments args) throws Exception;

}
```

两个接口的作用完全相同，唯一的区别是获取 run 方法入参的方式不用。

>   ```java
>   @SpringBootApplication
>   public class XxxApplication {
>       public static void main(String[] args) {
>           SpringApplication.run(DemoApplication.class, args);
>       }
>   }
>   ```

### 3、@Order 注解

如果存在多个 CommandLineRunner、ApplicationRunner 实例，可以通过 @Order 注解来设置它们的处理顺序。

>   Note：Lower values have higher priority

