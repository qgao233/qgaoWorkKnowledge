# 集成seam与Spring

参考：[在seam中使用spring](https://blog.csdn.net/iteye_2905/article/details/81601343)

---

seam对spring提供了很好的兼容性。

## 1 POJO代码的重用

正如spring管理的bean是POJO一样，seam管理的组件，也是POJO的，而且连ejb3也是POJO的，所以直接的重用以前用在spring 里的代码的方法，就是直接重用POJO了。

以前我们就applicationContext.xml来配置bean，现在只需要换一个语法就能把 spring的POJO bean变成seam的component。

例如，对于spring的bean:

```
<bean id="testBean" class="org.test.TestBean">
    <property name="stringProperty" value="this is a string" />
    <property name="intProperty" value="3" />
    <property name="anotherBean" ref="anotherBean" />
</bean>
```

对应到seam里就是：

```
<component name="testBean" class="org.test.TestBean">
    <property name="stringProperty">this is a string</property>
    <property name="intProperty">3</property>
    <property name="anotherBean">#{anotherBean}</property>
</component>
```

大部分的POJO可能这就可以搞定了。但是要注意：

假如这个POJO依赖于spring容器，比如它实现了spring的bean生命周期回调方法，类似于 afterPropertiesSet() , 那么你也得让seam去模仿spring容器，在适应的时候，去调用这些回调方法。比如：

```
public class TestBeanAdapter extends TestBean {
    @Create
    public void afterPropertiesSet() {
        super.afterPropertiesSet();
    }
}
```

## 2 通过JSF EL Resolver来桥接

只要实现了EL Resolver, 并在[faces-config.xml](./facesconfig.md)里注册它，那么在JSF里使用EL表达式时，就会自动地调用EL Resolver来寻找这个EL表达式对应的对象等。

seam当然实现了它自己的Resolver，而spring也实了这个Resolver，如果把它们都注册上，那么JSF EL 将变成seam与spring的一座桥梁。用下面代码注册spring EL Resolver:

```
<faces-config>
    <application>
        <el-resolver>
            org.springframework.web.jsf.el.SpringBeanFacesELResolver
        </el-resolver>
    </application>
</faces-config>
```

如果你定义了spring bean testBean, 那么在seam组件里，你就可以通过 @In("#{testBean}") 来注入spring的bean。

但是要记住，这种方法获得的对象，完全是由spring负责组装的地地道道的spring的bean, 所以它不具有seam component具有功能，比如双向注入，和seam的上下文完美一体，延长的持久化上下文(extended persistence context)。

>spring的bean只能注入不能导出，seam的bean是两者都可以。

## 3 seam里的spring

最完美的方案，我想你也想到了，就是让**seam来管理spring容器**，并加入seam的增强功能。这是可以做得到的，并且已经做到了，感谢spring的灵活性，和seam开发者们的努力。

像下面那样在**seam**的`conponents.xml`里加入spring名称空间，并且使用`<spring:context-loader />`：

```
<components xmlns="http://jboss.com/products/seam/components"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:spring="http://jboss.com/products/seam/spring"
    xsi:schemaLocation="
        http://jboss.com/products/seam/components
        http://jboss.com/products/seam/components-2.0.xsd
        http://jboss.com/products/seam/spring
        http://jboss.com/products/seam/spring-2.0.xsd">
    
    <!-- component declarations -->
    <spring:context-loader>
        <spring:config-locations>
            <value>classpath:spring-beans-persistence.xml</value>
            <value>classpath:spring-beans-service.xml</value>
        </spring:config-locations>
    </spring:context-loader>
</components>
```

现在当你启动seam时，seam便会到你指定的位置去寻找spring的配置，并起动一个spring容器。为了能让seam知道要如何增强 spring里的bean，我们需要在spring那边做一些手脚。

在**spring**的配置文件里加入seam的名称空间如下：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:seam="http://jboss.com/products/seam/spring-seam"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
        http://jboss.com/products/seam/spring-seam
        http://jboss.com/products/seam/spring-seam-2.0.xsd">

    <!-- bean definitions -->

    <bean id="aBean" class="org.test.TestBean" scope="prototype">
        <property name="aString" value="a string" />
        <seam:component name="testBean" scope="CONVERSATION" auto-create="true" />
    </bean>

    <bean id="anotherBean" class="org.test.AnotherBean" scope="prototype">
        <property name="aBean">
            <seam:instance name="testBean" />
        </property>
    </bean>

    <seam:configure-scopes default-auto-create="true"/>

</bean>
```

1. `<seam:conponent />` 把一个spring bean变成seam component
2. `<seam:instance>` 向一个spring bean里注入seam component
3. `<seam:configure-scopes>` 定义一些默认的配置

spring的 aBean在seam里被叫做 testBean, 并且它在 conversation scope里，spring的anotherBean在创建它时，会把testBean注入进去。

但是，在使用过程中，要注意，seam不能对spring通过jdk 动态代理增强过的类进行再增强，所以如果要让一个spring的bean参予到seam里来，就不要对它使用动态代理的方式来实现AOP，你需要在spring的配置文件里做如下配置：

```
<aop:config proxy-target-class="true">
......
</aop:config>
```