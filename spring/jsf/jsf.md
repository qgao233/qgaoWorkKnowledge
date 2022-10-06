# JSF、facelets

参考：[JSF、facelets](https://www.cnblogs.com/yxsh/p/9001296.html)

---

* jsf是一种用于构建java web应用程序的框架。它提供了一种**以组件为中心**的用户界面（UI）构建方法，从而简化了Java服务器端应用程序的开发。

>参考现在的vue、react前端框架

* Facelets是一种轻量级的页面声明语言，用于使用HTML样式构建JSF(JavaServer Faces)视图。

总结：`facelets -> jsf -> html页面`

## 1 JSF ( Java Sever Faces )

### 1.1 工作方式：　
　　　　jsf应用是事件驱动的，当一个事件发生时（比如用户单击一个按钮），事件通知通过HTTP发往服务器，服务器端使用叫做FacesServlet的特殊servlet处理该通知，web容器里每一个jsf应用都有它自己的FacesServlet;

在后台，每一个jsf请求都触发了3件事情：

1. FacesServlet创建FacesContext(该对象中包含Web容器传给FacesServlet的service方法的ServletContext,ServletRequest,ServletRespons对象，在处理过程中主要就是修改这个FacesContext)

2. FacesServlet把控制权交给Lifecycle

3. Lifecycle分6个阶段处理FacesContext(也即jsf生命周期过程)

### 1.2 生命周期

1. **重建视图**: 建立组件树,如果是首次渲染,则组件树被重置合适的状态;如果不是首次渲染，则组件树被创建跳到响应阶段(JSF的组件树结构和DOM是一样的，只不过后者是client前者是server)。

2. **应用请求值**: 树中的每个组件都能从请求参数中提取到新的值，并把值存储本地,之后处理所有与组件相关的事件进入队列，如果某个组件的immediate属性设置为true，那么验证，转换，以及与组件关联的事件在这个阶段被处理.

3. **处理验证**: 组件值转换成与之相对应的数据类型。如果转换失败，这一阶段将继续完成所有剩余的转换器，验证和运行所需的检查，但在完成后，跳转到生命周期的Render Response阶段。如果验证成功，则检查组件上的required 的属性。如果该属性是必须的并且组件中输入了值，那么与之相关的验证程序运行。如果required的属性是必须但又没有输入值，这一阶段完成（所有剩余验证程序还会继续执行），然后生命周期跳跃到Render Response阶段。如果required 属性标识为false,不管组件中有没有输入值，验证过程都不会运行。

4. **更新模型**: 验证组件的本地值移动到模型中，同时本地副本被丢弃。

5. **调用应用程序**: 执行应用级逻辑（如事件处理程序)。

6. **呈现响应**: 呈现树中的组件。后续请求和Restore View阶段保存状态信息。

## 2 facelets

包括以下功能：

* 它使用XHTML创建网页。
* 除了支持JavaServer Faces和JSTL标记库之外，它还支持Facelets标签库。
* 它支持表达语言(EL)。
* 它是使用组件和页面的模板。

优点：

* 它通过模板和复合组件支持代码可重用性。
* 它通过定制提供组件和其他服务器端对象的功能可扩展性。
* 编译时间更快
* 它在编译时验证表达式语言。
* 高性能渲染能力。

JSF(Java Server Faces)技术支持各种标签库，以将组件添加到网页。 而Facelets为了支持JSF标签库机制，使用XML命名空间声明。