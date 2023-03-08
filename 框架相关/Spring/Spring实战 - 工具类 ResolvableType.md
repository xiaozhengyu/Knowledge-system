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