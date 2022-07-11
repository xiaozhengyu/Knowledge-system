# Maven - 打包插件



当你使用 Maven 对项目进行打包时，你需要了解以下 3 个打包插件：

-   maven-jar-plugin：maven 默认的打包插件，用于构建 project jar
-   maven-shade-plugin：用于打可执行包，executable（fat）jar
-   maven-assembly-plugin：用于定制化打包方式



[toc]



---

## 一、Apache Maven Shade Plugin

>   官方说明：https://maven.apache.org/plugins/maven-shade-plugin/
>
>   -   Shade is bound to the `packing` phase and is used to create a **shaded** jar.

Shade 作用于 Maven 的 Packing 阶段，它能够将项目依赖的 jar 包解压并融合到项目自身的编译文件（.class）中。

例子1：

>   我们自己搞了一个项目，它的结构是这样的：
>
>   ```
>   com.xzy.demo
>       Main.java
>   ```
>
>   我们的项目依赖的某个 jar 包的结构是这样的：
>
>   ```
>   com.fake.text
>       A.class
>       B.class
>   ```
>
>   借助 Shade 插件，我们可以对上面两个项目的结构进行融合，并打进一个 jar 包：
>
>   ```
>   com.
>       xzy.demo
>           Main.class
>       fake.text
>           A.class
>           B.class
>   ```

例子2：

>   ```
>   // com.fake.test.StringUtils（v1.0）
>   
>   aaa(); // MARK
>   bbb();
>   ```
>
>   ```
>   // com.fake.test.StringUtils（v2.0）
>   
>   bbb();
>   ccc(); // MARK
>   ```
>
>   ```
>   // com.xzy.demo.Main
>   
>   import com.fake.test.StringUtils;
>   class Main {
>   
>       public static void main(args[]) {
>       
>           ...
>           StringUtils.aaa();
>           StringUtils.ccc();
>           ...
>       }
>   
>   }
>   ```
>
>   假设，在某个 jar 的 1.0 版本中 StringUtils 包含了 aaa 方法，在 2.0 版本中 aaa 方法被删除并新增了 ccc 方法。一般情况下，如果我们的项目是没法同时使用 aaa 和 ccc 方法的——包的名称会冲突，但是借助 Shade 我们可以解决这个问题：
>
>   -   This plugin provides the capability to package the artifact in an uber-jar, including its dependencies and to *shade* - i.e. **rename - the packages of some of the dependencies.**
>
>   ```
>   com.
>       xzy.demo
>           Main
>       a.fake.test
>           StringUtils
>       b.fake.test
>           StringUtils
>   ```
>
>   ```
>   // com.xzy.demo.Main
>   
>   import com.a.fake.test.StringUtils;
>   import com.a.fake.test.StringUtils;
>   class Main {
>   
>       public static void main(args[]) {
>       
>           ...
>           com.a.fake.test.StringUtils.aaa();
>           com.a.fake.test.StringUtils.ccc();
>           ...
>       }
>   
>   }
>   ```