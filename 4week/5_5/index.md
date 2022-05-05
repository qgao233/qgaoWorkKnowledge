# 第4周-5.5

## 1 spring中单例bean引用原型bean

[spring中单例bean引用原型bean](https://blog.csdn.net/qq_40866844/article/details/122012130)

### 1.1 场景：单例bean引用不变的原型对象

#### 1.1.1 xml文件内容

定义headMaster和student对原型bean，teacher为单例bean。其中teacher中注入headMaster和student属性。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="headMaster" class="com.spring.learn.common.staff.HeadMaster" scope="prototype"/>
    <bean id="student" class="com.spring.learn.common.staff.Student" scope="prototype"/>

    <bean id="teacher" class="com.spring.learn.common.staff.Teacher">
        <property name="headMaster" ref="headMaster"/>
        <property name="student" ref="student"/>
    </bean>

</beans>
```

#### 1.1.2 测试和结果

单例bean的teacher中，headMaster和student属性始终不变。从applicationContext中重新获取对应bean后，返回新建对象。

```
public static void main(String[] args) {

    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean/scope/scope.xml");

    Teacher teacher = ac.getBean("teacher", Teacher.class);
    HeadMaster headMaster = teacher.getHeadMaster();
    Student student = teacher.getStudent();

    HeadMaster headMasterTwo = teacher.getHeadMaster();
    Student studentTwo = teacher.getStudent();

    System.out.println(headMaster);
    System.out.println(headMasterTwo);
    System.out.println(ac.getBean("headMaster", HeadMaster.class));

    System.out.println(student);
    System.out.println(studentTwo);
    System.out.println(ac.getBean("student", Student.class));
}
/*
* com.spring.learn.common.staff.HeadMaster@569cfc36
* com.spring.learn.common.staff.HeadMaster@569cfc36
* com.spring.learn.common.staff.HeadMaster@33723e30
* com.spring.learn.common.staff.Student@64f6106c
* com.spring.learn.common.staff.Student@64f6106c
* com.spring.learn.common.staff.Student@553a3d88
*/
```

### 1.2 保证单例bean引用的原型始终是新建对象

引入 lookup-method或者 replace-method。

#### 1.2.1 更新xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="headMaster" class="com.spring.learn.common.staff.HeadMaster" scope="prototype"/>
    <bean id="student" class="com.spring.learn.common.staff.Student" scope="prototype"/>

    <bean id="teacher" class="com.spring.learn.common.staff.Teacher">
        <lookup-method name="getHeadMaster" bean="headMaster"/>
        <lookup-method name="getStudent" bean="student"/>
    </bean>

</beans>
```

#### 1.2.2 测试和结果

```
public static void main(String[] args) {

    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean/scope/scope2.xml");

    Teacher teacher = ac.getBean("teacher", Teacher.class);
    HeadMaster headMaster = teacher.getHeadMaster();
    Student student = teacher.getStudent();

    HeadMaster headMasterTwo = teacher.getHeadMaster();
    Student studentTwo = teacher.getStudent();

    System.out.println(headMaster);
    System.out.println(headMasterTwo);
    System.out.println(ac.getBean("headMaster", HeadMaster.class));

    System.out.println(student);
    System.out.println(studentTwo);
    System.out.println(ac.getBean("student", Student.class));
}

/*
* com.spring.learn.common.staff.HeadMaster@6b67034
* com.spring.learn.common.staff.HeadMaster@16267862
* com.spring.learn.common.staff.HeadMaster@71248c21
* com.spring.learn.common.staff.Student@442675e1
* com.spring.learn.common.staff.Student@6166e06f
* com.spring.learn.common.staff.Student@49e202ad
*/
```

### 1.3 源码原理

本质上就是对getStudent和getHeadMaster加一层cglib代理，每次执行方法前会经过拦截器，返回application.getBean方法返回的bean对象。

#### 1.3.1 初始化设置属性
初始化teacher时，

`AbstractAutowireCapableBeanFactory`类中的:
```
// lookup-method标签识别入口
522行：mbdToUse.prepareMethodOverrides();
```

`AbstractBeanDefinition`类中的:
```
// BeanDefinition类中的参数methodOverrides，中的参数overrides<类型：Set<MethodOverride>>。
// MethodOverride中的属性参数overloaded 设置为false
1134行：getMethodOverrides().getOverrides().forEach(this::prepareMethodOverride);
```

#### 1.3.2 执行getHeadMaster和getStudent后

`CglibSubclassingInstantiationStrategy`类中的`intercept`方法:

```
// 从当前bean的BeanDefinition信息中，获取重写方法列表中的lookup方法对象。
// 根据此对象信息完成指定对象的实例化
238行：LookupOverride lo = (LookupOverride) getBeanDefinition().getMethodOverrides().getOverride(method);

// 返回 根据xml文件中lookup-method指定bean值，从applicationContext中获取bean的对象。
242行：Object bean = (argsToUse != null ? this.owner.getBean(lo.getBeanName(), argsToUse) :
						this.owner.getBean(lo.getBeanName()));
```

## 2 StringBuffer清空操作

[StringBuffer清空操作delete和setLength的效率对比分析](https://blog.csdn.net/qq_35559358/article/details/77448582)

`setLength()`方法用时较短，因此在StringBuffer 清空操作中，使用setLength(int newLength)方法效率较高。