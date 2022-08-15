## 5 spring xml bean的property的简写——p命令空间

对“setter方法注入”进行简化，替换，不再用`<property>`，

而是在`bean`标签里`<bean p:属性名="普通值" p:属性名-ref="引用值">`

p命名空间使用前提，必须添加命名空间:

![](media/1.png)