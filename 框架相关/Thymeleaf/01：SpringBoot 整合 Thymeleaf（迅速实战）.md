# SpringBoot 整合 Thymeleaf（迅速实战）



>   参考文章：https://www.jianshu.com/p/908b48b10702/
>
>   Note：Thymeleaf 最为显著的特征是`属性增强`，任何属性都可以通过 `ht:xx` 来完成交互，例如，th:value 最终会被处理并替换为 value



[TOC]



## 一、基本语法



### 1. 变量表达式：`${}`

用途：获取 Model 对象的属性值

示例：

>   ```java
>   // User.class
>   
>   public class User {
>       private String id;
>       private String username;
>       private String password;
>       private Date createTime;
>       ...
>   }
>   ```
>
>   ```java
>   // UserController.class
>   
>   @Controller
>   @RequestMapper("/")
>   public class UserControlelr {
>   
>       @GetMapping("/{id}")
>       public String getUser(@PathVariable String id, Model model){
>           User user = ...
>           model.setAttribute("user",user);
>           return "user";
>       }
>   }
>   ```
>
>   ```html
>   <!--user.html-->
>   
>   ...
>   <div>
>       账号：<input type="text" th:value="${user.username}"/>
>       密码：<input type="password" th:value="${user.password}"/>
>       时间：<input type="text" th:value="${user.createTime}"/>
>   </div>
>   ...
>   ```
>
>   ```yaml
>   # application.yaml
>   
>   spring:
>     thymeleaf:
>       cache: false # 关闭缓存
>       prefix: classpath:/views/ # 页面路径
>   ```



### 2. 选择变量表达式：`*{}`

用途：获取 Model 对象的属性值

示例：

>   ```html
>   <!--user.html-->
>   
>   ...
>   <div th:object="${user}">
>       账号：<input type="text" th:value="*{username}"/>
>       密码：<input type="password" th:value="*{password}"/>
>       时间：<input type="text" th:value="*{createTime}"/>
>   </div>
>   ...
>   ```

首先通过 `${}` 获取到对象，然后用过 `*{}` 获取对象的属性。对照前文可以看出 `*{}`的简写风格更加清爽。



### 3. 链接表达式：`@{}`

用途：拿到应用路径，然后拼接静态资源路径

示例：

>   ```html
>   <script th:src="@{/webjars/jquery/jquery.js}"></script>
>   <link th:href="@{/webjars/bootstrap/css/bootstrap.css}" rel="stylesheet" type="text/css">
>   ```



### 4. 片段表达式：`~{}`

片段表达式是 Thymeleaf 的特色之一，粒度可以达到标签级别，这是JSP无法做到的。

片段表达式拥有三种语法：

-   `~{viewName}`：引入完整页面
-   `~{viewName::selector}`：引入指定页面的片段。其中 selector 可为片段名、jquery 选择器等
-   `~{::selector}` ：引用当前页面的片段

示例：首先通过 `th:fragment` 定义片段，然后通过 `th:replace` 引用片段

>   ```html
>    <!--/view/commom/head.html-->
>   ...
>   <head th:fragment="static">
>           <script th:src="@{/webjars/jquery/3.3.1/jquery.js}"></script>
>   </head>
>   ...
>   ```
>
>   ```html
>   <!--/views/user.html-->
>   ...
>   <div th:replace="~{common/head::static}"></div>
>   ...
>   ```
>
>   在实际使用中，往往使用更简洁的表达，去掉表达式外壳直接填写片段名。例如：
>
>   ```html
>   <!--/views/user.html-->
>   ...
>   <div th:replace="common/head::static"></div>
>   ...
>   ```

值得注意的是，替换路径`th:replace`后的值默认会被拼接上 `${spring.thymeleaf.prefix}`



### 5. 消息表达式：`#{}`

即通常的国际化属性：`#{msg}`  用于获取国际化语言翻译值。例如：

```html
<title th:text="#{user.title}"></title>
```



### 6. 其他表达式

在基础语法中，默认支持字符串连接、数学运算、布尔逻辑和三目运算等。例如：

```html
<input name="name" th:value="${'I am '+(user.name!=null?user.name:'NoBody')}"/>
```



## 二、内置对象

>   官方文档：[ 附录A： Thymeleaf 3.0 基础对象](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.thymeleaf.org%2Fdoc%2Ftutorials%2F3.0%2Fusingthymeleaf.html%23appendix-a-expression-basic-objects)
>
>   官方文档：[ 附录B： Thymeleaf 3.0 工具类 ](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.thymeleaf.org%2Fdoc%2Ftutorials%2F3.0%2Fusingthymeleaf.html%23appendix-b-expression-utility-objects)



### 七大基础对象：

-   `${#ctx}` 上下文对象，可用于获取其它内置对象。
-   `${#vars}`:    上下文变量。
-   `${#locale}`：上下文区域设置。
-   `${#request}`: HttpServletRequest对象。
-   `${#response}`: HttpServletResponse对象。
-   `${#session}`: HttpSession对象。
-   `${#servletContext}`:  ServletContext对象。



### 常用的工具类：

-   `#strings`：字符串工具类
-   `#lists`：List 工具类
-   `#arrays`：数组工具类
-   `#sets`：Set 工具类
-   `#maps`：常用Map方法。
-   `#objects`：一般对象类，通常用来判断非空
-   `#bools`：常用的布尔方法。
-   `#execInfo`：获取页面模板的处理信息。
-   `#messages`：在变量表达式中获取外部消息的方法，与使用＃{...}语法获取的方法相同。
-   `#uris`：转义部分URL / URI的方法。
-   `#conversions`：用于执行已配置的转换服务的方法。
-   `#dates`：时间操作和时间格式化等。
-   `#calendars`：用于更复杂时间的格式化。
-   `#numbers`：格式化数字对象的方法。
-   `#aggregates`：在数组或集合上创建聚合的方法。
-   `#ids`：处理可能重复的id属性的方法。



## 三、循环迭代

想要遍历`List`集合很简单，配合`th:each` 即可快速完成迭代。例如遍历用户列表：

```html
<div th:each="user:${userList}">
    账号：<input th:value="${user.username}"/>
    密码：<input th:value="${user.password}"/>
</div>
```

在集合的迭代过程还可以获取`状态变量`，<u>只需在变量后面指定状态变量名即可</u>，状态变量可用于获取集合的下标/序号、总数、是否为单数/偶数行、是否为第一个/最后一个。例如：

```html
<div th:each="user,stat:${userList}" th:class="${stat.even}?'even':'odd'">
    下标：<input th:value="${stat.index}"/>
    序号：<input th:value="${stat.count}"/>
    账号：<input th:value="${user.username}"/>
    密码：<input th:value="${user.password}"/>
</div>
```

>   如果缺省状态变量名，则迭代器会默认帮我们生成以变量名开头的状态变量 `xxStat`， 例如：
>
>   ```html
>   <div th:each="user:${userList}" th:class="${userStat.even}?'even':'odd'">
>       下标：<input th:value="${userStat.index}"/>
>       序号：<input th:value="${userStat.count}"/>
>       账号：<input th:value="${user.username}"/>
>       密码：<input th:value="${user.password}"/>
>   </div>
>   ```



## 四、条件判断

条件判断通常用于动态页面的初始化，例如：

```html
<div th:if="${userList}">
    <div>的确存在..</div>
</div>
```

如果想取反则使用unless 例如：

```html
<div th:unless="${userList}">
    <div>不存在..</div>
</div>
```



## 五、时间格式

使用默认的日期格式(toString方法) 并不是我们预期的格式：`Mon Dec 03 23:16:50 CST 2018`

```html
<input type="text" th:value="${user.createTime}"/>
```

此时可以通过时间工具类`#dates`来对日期进行格式化：`2018-12-03 23:16:50`

```html
<input type="text" th:value="${#dates.format(user.createTime,'yyyy-MM-dd HH:mm:ss')}"/>
```



## 六、内联写法

1）为什么需要内联写法？答：JS 无法获取 Model 对象

2）如何使用内联表达式？答：标准格式为：`[[${xx}]]` ，可以读取 Model 对象，也可以调用内置对象的方法。例如，获取用户变量和应用路径：

```html
<script th:inline="javascript">
    var user = [[${user}]];
    var APP_PATH = [[${#request.getContextPath()}]];
    var LANG_COUNTRY = [[${#locale.getLanguage()+'_'+#locale.getCountry()}]];
</script>
```

3）标签引入的 JS 里面能使用内联表达式吗？答：不能！内联表达式仅在页面生效，因为`Thymeleaf`只负责解析一级视图，不能识别外部标签JS里面的表达式。



## 七、实战案例