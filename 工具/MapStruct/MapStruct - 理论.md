# MapStruct 理论

[toc]



##### MapStruct 是什么？能干嘛？

在一些层级划分复杂的系统中，各个层级间通过 PO、DAO、DTO、VO、BO、QO 等`数据对象`进行通讯，不可避免的就需要进行数据对象间的映射操作，由此诞生了非常多的`对象映射组件`，如Apache BeanUtils、Spring BeanUtils、CGLIB BeanCopier、Dozer、Orika、ModelMapper、JMapper、Selma。MapStruct 也是一种对象映射组件。



##### MapStruct 有什么特点？什么时候选择 MapStruct？

```java
@Data
public class Source {
    private Integer aaa;
    private String bbb;
    private Long ccc;
}

@Data
public class Target {
    private Integer aaa;
    private String bbb;
    private Long ccc;
}
```

Spring BeanUtils 大致的工作原理：程序运行阶段，基于Java的反射机制，获取并遍历Source对象各个域的属性（类型、名称），“提取”域的值，然后“注入”Target对象对应的域。

MapStruct 大致的工作原理：基于注解以及注解处理器，在代码编译阶段为数据对象生成数值拷贝的代码，程序运行阶段直接通过这些生成的代码进行数值的拷贝。

对比 Spring BeanUtils，MapStruct 无需在程序的运行期间进行反射、生成字节码等操作。从效果上看，相当于调用了手工编写的数值转换代码，只不过 MapStruct 将编写代码的过程自动化了，但是为工程师节省了编写这种无聊代码的时间，同时避免了低级错误的发生。<u>如果追求极致性能，可以考虑使用 MapStruct。</u>



参考文章：

-   https://www.jianshu.com/p/913e791ecbcc