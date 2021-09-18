# synchronized锁升级、锁消除、锁粗化

参考文章：

1.   [java并发笔记四之synchronized 锁的膨胀过程（锁的升级过程）深入剖析](https://www.cnblogs.com/yuhangwang/p/11295940.html)
2.   [Java多线程锁的升级原理](https://blog.csdn.net/qq_41973594/article/details/109440441)

—

在jdk1.6之前，synchronized是一个重量级锁，效率很低。后来对synchronized进行了优化，有了一个锁升级的过程：无锁态（new）–> 偏向锁 –> 轻量级锁（自旋锁） –> 重量级锁



所谓的锁升级、锁消除、锁粗化，都是JVM优化synchronized的手段，JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现。



## 锁升级

