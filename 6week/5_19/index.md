# 第6周-5.19

## 1 test测试常用注解

[Springboot Test 详解](https://www.jianshu.com/p/c59263b90986)

在spring boot常这样写test：

```
@RunWith(SpringRunner.class)
@SpringBootTest(**.class)
@ActiveProfiles("Test")
public class Test{}
```

### 1.1 @RunWith

junit包下有实现，里面的value是指在Junit run之前为test准备什么样的支持。

* spring支持：`@RunWith(SpringRunner.class)`
* Mock支持：`@RunWith(MockitoJUnitRunner.class)`

>springRunner是SpringJUnit4ClassRunner的子类。

### 1.2 @SpringBootTest

* 没有value，则会从当前类往上找到spring boot主类
* 有value，直接将该value对应的类认为是spring boot主类

主类即被`@SpringBootApplication`注解，，这样才能将ioc环境引入test类，test类才能使用@Autowired等将bean注入等操作。

### 1.3 @ActiveProfiles("Test")

说白了就是去找配置文件，诸如application-**.properties/yml的文件。

## 2 反射中get**和getDeclared**的区别

* `get**()`：获得某个类的所有的公共（public）的(字段/方法)，**包括父类**中的(字段/方法)。
* `getDeclared**()`：获得某个类的所有声明的(字段/方法)，即包括`public`、`private`和`proteced`，但是**不包括父类**的申明(字段/方法)。

## 3 反射常用api

### 3.1 Class常用api

（1） Field[] getFields()
返回这个类或其超类的公共字段的Field对象数组。如果Class对象描述的是基本类型或者数组类型，将返回一个长度为0的数组。

（2） Field[] getDeclaredFields()
返回这个类的全部字段对应的Field对象数组。如果Class对象描述的是基本类型或者数组类型，将返回一个长度为0的数组。

（3）Field getField(String name)
返回指定名称的Field。如果不存在对应名称的字段，则抛出异常 NoSuchFieldException 。

（4）Field getDeclaredField(String name)
返回指定名称的Field。如果不存在对应名称的字段，则抛出异常 NoSuchFieldException 。

（5） Method[] getMethods()
返回所有的公共方法，包括从超类继承来的公共方法。

（6） Method[] getDeclaredMethods()
返回这个类或者接口的所有方法，但是不包括从超类继承的方法。

（7） Constructor[] getConstructors()
返回这个类的公共构造方法。

（8） Constructor[] getDeclaredConstructors(Class<?>... parameterTypes)
返回这个类的全部构造方法。参数类型是一个类对象的数组，该数组标识构造函数的正式参数类型，即**声明的顺序**。

（9） String getPackageName
得到包含这个类型的包的包名。如果该类是一个数组类型，则返回元素类型所属的包。如果是基本数据类型，则返回 “java.lang”。