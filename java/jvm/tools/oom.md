# oom是如何出现的？

参考：[GC overhead limit exceeded问题](https://blog.csdn.net/qq_36908872/article/details/102685311)

---

oom：out of memory

## 1 先发生GC overhead limit exceeded

`java.lang.OutOfMemoryError: GC overhead limit exceeded`。

在连续多次 `GC` 花费的时间超过 98%, 并且GC回收的内存少于 2%, JVM就会抛出这个错误。

GC清理的这么点内存很快会再次填满, 迫使GC再次执行. 这样就形成恶性循环, CPU使用率一直是100%, 而GC却没有任何成果。

而系统用户就会看到系统卡死 - 以前只需要几毫秒的操作, 现在需要好几分钟才能完成。

这也是一个很好的 [快速失败原则](http://en.wikipedia.org/wiki/Fail-fast) 的案例。

## 2 最终发生Java heap space

`java.lang.OutOfMemoryError: Java heap space`。

堆内存完全用光，年轻代和老年代使用率全部99%。

## 3 解决办法

### 3.1 自慰式

>解决第1小节的问题

启动java程序时添加下面启动参数，可以不抛出 “java.lang.OutOfMemoryError: GC overhead limit exceeded” 错误信息：

```
-XX:-UseGCOverheadLimit
```

这不能真正地解决问题，只能推迟一点 `java.lang.OutOfMemoryError: Java heap space` 错误发生的时间。

### 3.2 分配给JVM的堆内存不足

增加堆内存大小即可。

### 3.3 内存泄漏导致

如果只增加堆内存：

* 推迟产生 `java.lang.OutOfMemoryError: Java heap space` 错误的时间。
* 增加 GC pauses 的时间, 从而影响程序的 吞吐量或延迟。

从根本上解决问题，思考：

* 哪类对象占用了最多内存？
* 这些对象是在哪部分代码中分配的。

操作流程：

1. 获得在生产服务器上执行堆转储(heap dump)的权限。
    >“转储”(Dump)是堆内存的快照, 可用于后续的内存分析. 这些快照中可能含有机密信息, 例如密码、信用卡账号等, 所以有时候, 由于企业的安全限制, 要获得生产环境的堆转储并不容易。

2. 在适当的时间执行堆转储。
    >一般来说,内存分析需要比对多个堆转储文件, 假如获取的时机不对, 那就可能是一个“废”的快照. 另外, 每执行一次堆转储, 就会对JVM进行一次“冻结”, 所以生产环境中,不能执行太多的Dump操作,否则系统缓慢或者卡死,你的麻烦就大了。

3. 用另一台机器来加载Dump文件。
    >如果出问题的JVM内存是8GB, 那么分析 Heap Dump 的机器内存一般需要大于 8GB. 然后打开转储分析软件(我们推荐Eclipse MAT , 当然你也可以使用其他工具)。

4. 检测快照中占用内存最大的 GC roots。
    >详情请参考: [Solving OutOfMemoryError (part 6) – Dump is not a waste](https://plumbr.eu/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste)。 这对新手来说可能有点困难, 但这也会加深你对堆内存结构以及 navigation 机制的理解。

5. 接下来, 找出可能会分配大量对象的代码。
    >如果对整个系统非常熟悉, 可能很快就能定位问题。

## 4 工具

### 4.1 plumbr

[Plumbr](http://plumbr.eu/) 能捕获所有的 java.lang.OutOfMemoryError , 并找出其他的性能问题, 例如最消耗内存的数据结构等等。

### 4.2 dump分析工具

参考：[Java中生成dump文件命令及分析工具下载](http://qclog.cn/1348)

#### 1 Eclipse MAT

http://www.eclipse.org/mat/

#### 2 Memory Analyzer（mat）——免费

下载地址：https://www.eclipse.org/mat/downloads.php
如果最新版本提示jdk版本比较低，而你本机还是jdk8的话，可以下载1.8.1的版本进行使用
历史版本下载地址：https://www.eclipse.org/mat/previousReleases.php

#### 3 JPROFILER——收费，可以试用10天

下载地址：https://www.ej-technologies.com/download/jprofiler/files