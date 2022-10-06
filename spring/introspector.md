# 使用内省导致内存泄露

参考：[IntrospectorCleanupListener作用](https://www.cnblogs.com/panchanggui/p/10237086.html)

---

```
<!--web.xml-->
<listener>
        <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
</listener>
```

1. 此监听器主要用于解决java.beans.Introspector导致的内存泄漏的问题

2. 此监听器应该配置在web.xml中与Spring相关监听器中的第一个位置(也要在ContextLoaderListener的前面)

JDK中的`java.beans.Introspector`类的用途是发现Java类是否符合JavaBean规范，

如果有的框架或程序用到了Introspector类，那么就会启用一个**系统级别的缓存**，此缓存会存放一些曾加载并分析过的JavaBean的引用。

当Web服务器关闭时，由于此缓存中存放着这些JavaBean的引用，所以垃圾回收器**无法回收**Web容器中的JavaBean对象，最后导致内存变大。

而`org.springframework.web.util.IntrospectorCleanupListener`就是专门用来处理Introspector内存泄漏问题的辅助类。IntrospectorCleanupListener会在Web服务器**停止时清理**Introspector缓存，使那些Javabean能被垃圾回收器正确回收。

Spring自身**不会出现**这种问题，因为Spring在加载并分析完一个类之后会马上刷新JavaBeans Introspector缓存，这就保证Spring中不会出现这种内存泄漏的问题。

但有些程序和框架在使用了JavaBeans Introspector之后，**没有进行清理**工作(如 Quartz，Struts)，最后导致内存泄漏。