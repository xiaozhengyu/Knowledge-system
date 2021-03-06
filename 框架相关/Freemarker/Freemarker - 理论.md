# Freemarker

<(￣︶￣)↗[Freemarker 在线手册（中文）](http://freemarker.foofun.cn/index.html)

---

Freemarker 是一款模板引擎：即一种基于模板和数据模型来生成文本（HTML网页、电子邮件、配置文件、源代码等）的工具



![Figure](markdown/Freemarker - 理论.assets/overview.png)



>   Note：
>
>   模板其实就是一个文本，但它还只是半成品，需要进一步加工处理。
>
>   -   由谁来负责处理？答：模板引擎
>   -   文本能够被模板引擎处理，必须满足什么条件？答：文本必须按照一定的规则编写，方便模板引擎找出需要处理的地方
>   -   大致的处理流程？答：扫描文本，找到需要处理的地方 -> 执行相应的操作获取填充数据 -> 将填充数据 -> 输出最终文本

>   Note：
>
>   模板 = 空白的暑假作业
>
>   数据模型 = 参考答案
>
>   模板引擎 = 你
>
>   “你打开空白的作业，找到对应的参考答案，把答案抄上去，得到写好的作业 😎”

