# 工具类 ResolvableType

[TOC]



## 一、Java 中的 Type

官方说明：“Type is the <u>common superinterface for all types</u> in the Java programming language. These include raw types, parameterized types, array types, type variables and primitive types.”

Type 的子接口以及实现类：

![image-20230308155525335](markdown/Spring实战 - 工具类 ResolvableType.assets/image-20230308155525335.png)

-   ParameterizedType

    ```java
    @Slf4j
    class AaaTest {
        private Map<String, Integer> map;
    
        public static void main(String[] args) throws NoSuchFieldException {
            Field field = AaaTest.class.getDeclaredField("map");
            Type genericType = field.getGenericType();
    
            /*
             * ParameterizedType：参数化类型，例如List<T>，Set<T>，Map<K,V>
             */
            if (genericType instanceof ParameterizedType) {
                ParameterizedType parameterizedType = (ParameterizedType) genericType;
    
                /*
                 * 获取外部类的Type信息：以 Map.Entry 为例，它的外部类就是 Map
                 */
                Type ownerType = parameterizedType.getOwnerType();
                log.info(String.valueOf(ownerType));
    
                /*
                 * 获取原始类型信息：
                 *     List<Integer> => List
                 *     Set<Integer> => Set
                 *     Map<String,Integer> => Map
                 */
                Type rawType = parameterizedType.getRawType();
                log.info(String.valueOf(rawType));
    
                /*
                 * 获取类型参数信息：
                 *     List<Integer> => [Integer]
                 *     Set<Integer> => [Integer]
                 *     Map<String,Integer> => [String,Integer]
                 */
                Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
                log.info(String.valueOf(Arrays.asList(actualTypeArguments)));
            }
        }
    }
    ```

-   TypeVariable

    ```java
    @Slf4j
    class BbbTest<K, V extends Number & Serializable> {
        private Map<K, V> map;
    
        public static void main(String[] args) throws NoSuchFieldException {
            Field field = BbbTest.class.getDeclaredField("map");
            Type genericType = field.getGenericType();
    
            /*
             * ParameterizedType：参数化类型，例如List<T>，Set<T>，Map<K,V>
             */
            if (genericType instanceof ParameterizedType) {
                ParameterizedType parameterizedType = (ParameterizedType) genericType;
    
                Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
                for (Type actualTypeArgument : actualTypeArguments) {
    
                    /*
                     * TypeVariable：类型变量，指的是List<T>，Map<K,V>中的T，K，V
                     */
                    if (actualTypeArgument instanceof TypeVariable) {
                        TypeVariable typeVariable = (TypeVariable) actualTypeArgument;
    
                        /*
                         * 获取类型变量的上届：也就是申明泛型的时候 extends 右边的东西，如果没有指定则是 Object
                         */
                        Type[] bounds = typeVariable.getBounds();
                        log.info(String.valueOf(Arrays.asList(bounds)));
    
                        /*
                         * 获取类型变量在源码中的名字：“T”，“K”，“V”
                         */
                        String name = typeVariable.getName();
                        log.info(name);
    
                        /*
                         * 获取类型变量的载体：
                         *     BbbTest<K, V extends Number & Serializable> => K,V 的载体就是 class com.xzy.java.bbb.BbbTest
                         */
                        GenericDeclaration genericDeclaration = typeVariable.getGenericDeclaration();
                        log.info(String.valueOf(genericDeclaration));
                    }
                }
            }
        }
    }
    ```

-   GenericArrayType

    ```java
    @Slf4j
    public class CccTest<T> {
        T[] array1;
        List<T>[] array2;
        T[][][] array3;
    
        public static void main(String[] args) throws NoSuchFieldException {
            Field field = CccTest.class.getDeclaredField("array1");
            Type genericType = field.getGenericType();
    
            /*
             * GenericArrayType：泛型数组，例如T[]
             */
            if (genericType instanceof GenericArrayType) {
                GenericArrayType genericArrayType = (GenericArrayType) genericType;
    
                /*
                 * 获取数组元素的类型信息：
                 *     T[] => T                是 TypeVariable
                 *     List<T>[] => List<T>    是 GenericArrayType
                 *     T[][][] => T[]][]       是 GenericArrayType
                 */
                Type genericComponentType = genericArrayType.getGenericComponentType();
                log.info(String.valueOf(genericComponentType));
                log.info(genericArrayType.getClass().toString());
            }
        }
    }
    ```

## 二、Spring 中的 ResolvableType

ResolvableType 是 Spring 中解析泛型信息的一个工具类，它封装了 Type，使得泛型信息的解析变得更加简单。



### 2.1 对比 ResolvableType 与 Type

要解析的类：

```java
public class GenericClass {
   private HashMap<String, List<Integer>> field;
}
```

使用 Type 解析：

```java
private static void analysisByJDK() throws NoSuchFieldException {
    Field field = GenericClass.class.getDeclaredField("field");
    ParameterizedType fieldType = (ParameterizedType) field.getGenericType();
    Type[] argType = fieldType.getActualTypeArguments();
    log.info("解析HashMap<String, List<Integer>>中的HashMap：{}", fieldType.getRawType());
    log.info("解析HashMap<String, List<Integer>>中的HashMap的父类型：{}", ((Class) fieldType.getRawType()).getSuperclass());
    log.info("解析HashMap<String, List<Integer>>中的String：{}", ((Class) argType[0]));
    log.info("解析HashMap<String, List<Integer>>中的List<Integer>：{}", ((ParameterizedType) argType[1]));
    log.info("解析HashMap<String, List<Integer>>中的List：{}", ((ParameterizedType) argType[1]).getRawType());
    log.info("解析HashMap<String, List<Integer>>中的Integer：{}", ((Class) ((ParameterizedType) argType[1]).getActualTypeArguments()[0]));
}
```

使用 ResolvableType 解析：

```java
private static void analysisBySpring() throws NoSuchFieldException {
    Field field = GenericClass.class.getDeclaredField("field");
    ResolvableType fieldResolvableType = ResolvableType.forField(field);
    log.info("解析HashMap<String, List<Integer>>中的HashMap：{}", fieldResolvableType.getRawClass());
    log.info("解析HashMap<String, List<Integer>>中的HashMap的父类型：{}", fieldResolvableType.getSuperType().getRawClass());
    log.info("解析HashMap<String, List<Integer>>中的HashMap的父类型：{}", fieldResolvableType.getSuperType());
    log.info("解析HashMap<String, List<Integer>>中的String：{}", fieldResolvableType.getGeneric(0).resolve());
    log.info("解析HashMap<String, List<Integer>>中的List<Integer>：{}", fieldResolvableType.getGeneric(1));
    log.info("解析HashMap<String, List<Integer>>中的List：{}", fieldResolvableType.getGeneric(1).resolve());
    log.info("解析HashMap<String, List<Integer>>中的Integer：{}", fieldResolvableType.getGeneric(1, 0));
}
```

处理结果：

```
========== Type ==========
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的HashMap：class java.util.HashMap
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的HashMap的父类型：class java.util.AbstractMap
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的String：class java.lang.String
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的List<Integer>：java.util.List<java.lang.Integer>
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的List：interface java.util.List
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的Integer：class java.lang.Integer
========== ResolvableType ==========
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的HashMap：class java.util.HashMap
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的HashMap的父类型：class java.util.AbstractMap
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的HashMap的父类型：java.util.AbstractMap<java.lang.String, java.util.List<java.lang.Integer>>
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的String：class java.lang.String
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的List<Integer>：java.util.List<java.lang.Integer>
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的List：interface java.util.List
[main] INFO com.xzy.spring.aaa.GenericClass - 解析HashMap<String, List<Integer>>中的Integer：java.lang.Integer
```

Type 最麻烦的地方就是各种方法分散在子接口中，使用前需要先进行类型判断、强制类型转换，既不方便又容易出错。相比之下，ResolvableType 统一封装了各种方法，用起来方便很多。

![image-20230309140542678](markdown/Spring实战 - 工具类 ResolvableType.assets/image-20230309140542678.png)



### 2.2 创建 ResolvableType

使用 ResolvableType 解析泛型信息前需要先获取 ResolvableType 实例。泛型可能存在于类、成员变量、方法参数、返回返回值…，ResolvableType 提供了一系列静态方法同于将这些位置的泛型信息转换成 ResolvableType 实例，常见方法如下：

```java
public class ResolvableType implements Serializable {

	// 根据原始类型 Class 创建 ResolvableType
	public static ResolvableType forClass(@Nullable Class<?> clazz);

	// 根据构造器参数创建 ResolvableType
	public static ResolvableType forConstructorParameter(Constructor<?> constructor, int parameterIndex);
	
	// 根据成员变量创建 ResolvableType
	public static ResolvableType forField(Field field);
	
	// 根据实例创建 ResolvableType
	public static ResolvableType forInstance(Object instance);

	// 根据方法参数创建 ResolvableType
	public static ResolvableType forMethodParameter(Method method, int parameterIndex);
	
	// 根据方法的返回值创建 ResolvableType
	public static ResolvableType forMethodReturnType(Method method);
	
	// 根据原始类型信息创建 ResolvableType
	public static ResolvableType forRawClass(@Nullable Class<?> clazz);

	// 根据某一种类型创建 ResolvableType
	public static ResolvableType forType(@Nullable Type type);
}

```

使用示例：

```java
package com.xzy.spring.bbb;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.ResolvableType;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * 如何获取ResolvableType实例
 *
 * @author xzy.xiao
 * @date 2023/3/8  20:01
 */
class GenericClass {

    private static final Logger log = LoggerFactory.getLogger(GenericClass.class);
    private Map<String, List<Integer>> map;
    private List<Integer> list;
    private Set<Integer> set;
    private Integer integer;

    public GenericClass() {
    }

    public GenericClass(Map<String, List<Integer>> map, List<Integer> list, Set<Integer> set, Integer integer) {
        this.map = map;
        this.list = list;
        this.set = set;
        this.integer = integer;
    }

    public void setMap(Map<String, List<Integer>> map) {
        this.map = map;
    }

    public Map<String, List<Integer>> getMap() {
        return map;
    }

    public static void main(String[] args) throws NoSuchMethodException {
        System.out.println("\n========== 根据原始类型 Class 创建 ResolvableType ==========\n");
        test1();

        System.out.println("\n========== 根据构造器参数创建 ResolvableType ==========\n");
        test2();

        System.out.println("\n========== 根据成员变量创建 ResolvableType ==========\n");
        test3();

        System.out.println("\n========== 根据实例创建 ResolvableType ==========\n");
        test4();

        System.out.println("\n========== 根据方法参数创建 ResolvableType ==========\n");
        test5();

        System.out.println("\n========== 根据方法的返回值创建 ResolvableType ==========\n");
        test6();

        System.out.println("\n========== 根据原始类型信息创建 ResolvableType ==========\n");
        test7();
    }

    /**
     * 根据原始类型信息创建 ResolvableType
     */
    private static void test7() {
        ResolvableType resolvableType0 = ResolvableType.forRawClass(List.class);
        log.info("原始类型 List.class => {}", resolvableType0);

        ResolvableType resolvableType1 = ResolvableType.forRawClass(Set.class);
        log.info("原始类型 Set.class => {}", resolvableType1);

        ResolvableType resolvableType2 = ResolvableType.forRawClass(Map.class);
        log.info("原始类型 Map.class => {}", resolvableType2);

        ResolvableType resolvableType3 = ResolvableType.forRawClass(String.class);
        log.info("原始类型 String.class => {}", resolvableType3);

        ResolvableType resolvableType4 = ResolvableType.forRawClass(Integer.class);
        log.info("原始类型 Integer.class => {}", resolvableType4);
    }

    /**
     * 根据方法的返回值创建 ResolvableType
     *
     * @throws NoSuchMethodException -
     */
    private static void test6() throws NoSuchMethodException {
        Method method = GenericClass.class.getMethod("getMap");
        ResolvableType resolvableType = ResolvableType.forMethodReturnType(method);
        log.info("方法 Map<String, List<Integer>> getMap() 的返回值 => {}", resolvableType);
    }

    /**
     * 根据方法参数创建 ResolvableType
     *
     * @throws NoSuchMethodException -
     */
    private static void test5() throws NoSuchMethodException {
        Method method = GenericClass.class.getMethod("setMap", Map.class);
        ResolvableType resolvableType = ResolvableType.forMethodParameter(method, 0);
        log.info("方法 void setMap(Map<String, List<Integer>> map) 的第一个参数 => {}", resolvableType);
    }

    /**
     * 根据实例创建 ResolvableType
     */
    private static void test4() {
        GenericClass genericClass = new GenericClass();
        ResolvableType resolvableType = ResolvableType.forInstance(genericClass);
        log.info("实例类型 GenericClass => {}", resolvableType);
    }

    /**
     * 根据成员变量创建 ResolvableType
     */
    private static void test3() {
        Field[] fields = GenericClass.class.getDeclaredFields();

        ResolvableType resolvableType0 = ResolvableType.forField(fields[0]);
        log.info("成员变量 private static final Logger log => {}", resolvableType0);

        ResolvableType resolvableType1 = ResolvableType.forField(fields[1]);
        log.info("成员变量 private Map<String, List<Integer>> map => {}", resolvableType1);

        ResolvableType resolvableType2 = ResolvableType.forField(fields[2]);
        log.info("成员变量 private List<Integer> list => {}", resolvableType2);

        ResolvableType resolvableType3 = ResolvableType.forField(fields[3]);
        log.info("成员变量 private Set<Integer> set => {}", resolvableType3);

        ResolvableType resolvableType4 = ResolvableType.forField(fields[4]);
        log.info("成员变量 private Integer integer => {}", resolvableType4);
    }

    /**
     * 根据构造器参数创建 ResolvableType
     */
    private static void test2() {
        // 构造器：GenericClass(Map<String, List<Integer>> map, List<Integer> list, Set<Integer> set, Integer integer)
        Constructor<?> constructor = GenericClass.class.getDeclaredConstructors()[1];

        ResolvableType resolvableType0 = ResolvableType.forConstructorParameter(constructor, 0);
        log.info("构造器参数 Map<String, List<Integer>> => {}", resolvableType0);

        ResolvableType resolvableType1 = ResolvableType.forConstructorParameter(constructor, 1);
        log.info("构造器参数 List<Integer> list => {}", resolvableType1);

        ResolvableType resolvableType2 = ResolvableType.forConstructorParameter(constructor, 2);
        log.info("构造器参数 Set<Integer> => {}", resolvableType2);

        ResolvableType resolvableType3 = ResolvableType.forConstructorParameter(constructor, 3);
        log.info("构造器参数 Integer => {}", resolvableType3);

    }

    /**
     * 根据原始类型 Class 创建 ResolvableType
     */
    private static void test1() {
        Class<GenericClass> clazz = GenericClass.class;
        ResolvableType resolvableType = ResolvableType.forClass(clazz);
        log.info("原始类型 GenericClass => {}", resolvableType);
    }
}
```



### 2.3 使用 ResolvableType

ResolvableType 中常用的获取泛型信息的方法如下：

```java
public class ResolvableType implements Serializable {
	
	// 获取泛型数组的元素类型
	public ResolvableType getComponentType();

	// 获取泛型的实际类型
	public ResolvableType getGeneric(@Nullable int... indexes);
	public ResolvableType[] getGenerics();

	// 获取类型实现的接口
	public ResolvableType[] getInterfaces();
	
	// 获取指定嵌套的类型
	public ResolvableType getNested(int nestingLevel);
	public ResolvableType getNested(int nestingLevel, @Nullable Map<Integer, Integer> typeIndexesPerLevel);

	// 获取原始类型
	public Class<?> getRawClass();

	// 获取父类型
	public ResolvableType getSuperType();

	// 获取当前实例表示的Type
	public Type getType();

	// 判断当前实例是否包含泛型参数
	public boolean hasGenerics();

	// 判断当前实例是否为数组
	public boolean isArray();

	// 判断指定类型是否可以转换为当前类型
	public boolean isAssignableFrom(Class<?> other);
	public boolean isAssignableFrom(ResolvableType other);

	// 获取当前实例解析出的Class
	public Class<?> resolve();
	public Class<?> resolveGeneric(int... indexes)
}

```

使用示例：

```java
package com.xzy.spring.ccc;

import lombok.extern.slf4j.Slf4j;
import org.springframework.core.ResolvableType;

import java.io.Serializable;
import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * 如何根据ResolvableType解析泛型信息
 *
 * @author xzy.xiao
 * @date 2023/3/10  13:56
 */
@Slf4j
class GenericClass extends MySuperClass implements Comparable<Integer>, Serializable {
    public Integer[] integerArray;
    public List<Integer> integerList;
    public Map<String, List<Integer>> string2IntegerListMap;
    public Map<Map<Integer, String>, Map<String, List<Integer>>> badMap;

    @Override
    public int compareTo(Integer o) {
        return 0;
    }

    public static void main(String[] args) throws NoSuchFieldException {
        log.info("========== 获取泛型数组的元素类型 ==========");
        test0();

        log.info("========== 获取泛型的实际类型 ==========");
        test1();

        log.info("========== 获取类型实现的接口 ==========");
        test2();

        log.info("========== 获取指定嵌套的类型 ==========");
        test3();

        log.info("========== 获取原始类型 ==========");
        test4();

        log.info("========== 获取父类型 ==========");
        test5();

        log.info("========== 获取Type ==========");
        test6();

        log.info("========== 判断当前实例是否包含泛型参数 ==========");
        test7();

        log.info("========== 判断当前实例是否为数组 ==========");
        test8();

        log.info("========== 判断指定类型是否可以转换为当前类型 ==========");
        test9();

        log.info("========== 获取当前实例解析出的Class ==========");

        test10();
    }

    /**
     * 获取当前实例解析出的Class
     *
     * @throws NoSuchFieldException -
     */
    private static void test10() throws NoSuchFieldException {
        Class<?> clazz0 = ResolvableType.forClass(GenericClass.class).resolve();
        log.info("获取当前实例解析出的Class：class GenericClass => {}", clazz0);

        Class<?> clazz1 = ResolvableType.forField(GenericClass.class.getField("integerList")).resolveGeneric(0);
        log.info("获取当前实例解析出的Class：List<Integer> => {}", clazz1);

        Class<?> clazz2 = ResolvableType.forField(GenericClass.class.getField("string2IntegerListMap")).resolveGeneric(0);
        log.info("获取当前实例解析出的Class：Map<String, List<Integer>>中的String => {}", clazz2);

        Class<?> clazz3 = ResolvableType.forField(GenericClass.class.getField("string2IntegerListMap")).resolveGeneric(1);
        log.info("获取当前实例解析出的Class：Map<String, List<Integer>>中的List<Integer> => {}", clazz3);
    }

    /**
     * 判断指定类型是否可以转换为当前类型
     */
    private static void test9() {
        log.info("判断指定类型是否可以转换为当前类型：GenericClass is MySuperClass ? => {}", ResolvableType.forClass(MySuperClass.class).isAssignableFrom(GenericClass.class));
        log.info("判断指定类型是否可以转换为当前类型：GenericClass is Comparable ? => {}", ResolvableType.forClass(Comparable.class).isAssignableFrom(GenericClass.class));
        log.info("判断指定类型是否可以转换为当前类型：GenericClass is Serializable ? => {}", ResolvableType.forClass(Serializable.class).isAssignableFrom(GenericClass.class));
        log.info("判断指定类型是否可以转换为当前类型：GenericClass is String ? => {}", ResolvableType.forClass(String.class).isAssignableFrom(GenericClass.class));
    }

    /**
     * 判断当前实例是否为数组
     *
     * @throws NoSuchFieldException -
     */
    private static void test8() throws NoSuchFieldException {
        log.info("判断当前实例是否为数组：Integer[] => {}", ResolvableType.forField(GenericClass.class.getField("integerArray")).isArray());
        log.info("判断当前实例是否为数组：Map<String, List<Integer>> => {}", ResolvableType.forField(GenericClass.class.getField("string2IntegerListMap")).isArray());
    }

    /**
     * 判断当前实例是否包含泛型参数
     */
    private static void test7() {
        log.info("判断当前实例是否包含泛型参数：class String => {}", ResolvableType.forClass(String.class).hasGenerics());
        log.info("判断当前实例是否包含泛型参数：interface List<E> extends Collection<E> => {}", ResolvableType.forClass(List.class).hasGenerics());
    }

    /**
     * 获取Type
     *
     * @throws NoSuchFieldException -
     */
    private static void test6() throws NoSuchFieldException {
        log.info("获取Type：{}", ResolvableType.forClass(GenericClass.class).getType());
        log.info("获取Type：{}", ResolvableType.forField(GenericClass.class.getField("integerArray")).getType());
        log.info("获取Type：{}", ResolvableType.forField(GenericClass.class.getField("string2IntegerListMap")).getType());
    }

    /**
     * 获取父类型
     */
    private static void test5() {
        ResolvableType superType = ResolvableType.forClass(GenericClass.class).getSuperType();
        log.info("获取父类型：class GenericClass extends MySuperClass => {}", superType);
    }

    /**
     * 获取原始类型
     *
     * @throws NoSuchFieldException -
     */
    private static void test4() throws NoSuchFieldException {
        ResolvableType fieldType = ResolvableType.forField(GenericClass.class.getDeclaredField("string2IntegerListMap"));
        log.info("获取原始类型：Map<String, List<Integer>> => {}", fieldType.getRawClass());
    }

    /**
     * 获取指定嵌套的类型
     *
     * @throws NoSuchFieldException -
     */
    private static void test3() throws NoSuchFieldException {
        // 嵌套级别从 1 开始
        ResolvableType fieldType = ResolvableType.forField(GenericClass.class.getDeclaredField("badMap"));

        /*
         * Map<Map<Integer, String>, Map<String, List<Integer>>>：
         *
         * Map<Map,Map>                                           第一层
         *          |
         *          |---> Map<String, List>                       第二层
         *                            |
         *                            |---> List<Integer>         第三层
         */
        log.info("获取指定嵌套的类型：Map<Map<Integer, String>, Map<String, List<Integer>>>的第一层 => {}", fieldType.getNested(1));
        log.info("获取指定嵌套的类型：Map<Map<Integer, String>, Map<String, List<Integer>>>的第二层 => {}", fieldType.getNested(2));
        log.info("获取指定嵌套的类型：Map<Map<Integer, String>, Map<String, List<Integer>>>的第三层 => {}", fieldType.getNested(3));
    }

    /**
     * 获取类型实现的接口
     */
    private static void test2() {
        ResolvableType[] implInterfaces = ResolvableType.forClass(GenericClass.class).getInterfaces();
        log.info("获取类型实现的接口：class GenericClass implements Comparable<Integer>, Serializable => {}", Arrays.toString(implInterfaces));
    }

    /**
     * 获取泛型的实际类型
     *
     * @throws NoSuchFieldException -
     */
    private static void test1() throws NoSuchFieldException {
        ResolvableType fieldType = ResolvableType.forField(GenericClass.class.getDeclaredField("string2IntegerListMap"));

        // 索引位置从0开始

        ResolvableType generic0 = fieldType.getGeneric(0);
        log.info("获取泛型的实际类型：Map<String, List<Integer>>中的String => {}", generic0);

        ResolvableType generic1 = fieldType.getGeneric(1);
        log.info("获取泛型的实际类型：Map<String, List<Integer>>中的List<Integer> => {}", generic1);

        ResolvableType generic10 = fieldType.getGeneric(1, 0);
        log.info("获取泛型的实际类型：Map<String, List<Integer>>中的Integer => {}", generic10);
    }

    /**
     * 获取泛型数组的元素类型
     *
     * @throws NoSuchFieldException -
     */
    private static void test0() throws NoSuchFieldException {
        ResolvableType fieldType = ResolvableType.forField(GenericClass.class.getDeclaredField("integerArray"));
        ResolvableType componentType = fieldType.getComponentType();
        log.info("获取泛型数组的元素类型：Integer[] => {}", componentType);
    }
}
```