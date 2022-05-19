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

## 4 反射修改static final修饰的字段

[Java反射-修改字段值, 反射修改static final修饰的字段](http://www.javashuo.com/article/p-tkbmpjoq-cz.html)

### 4.1 修改private修饰StringBuilder类型的字段

```
public class Pojo {
    private StringBuilder name = new StringBuilder("default");

    public void printName() {
        System.out.println(name);
    }
}

//测试
Pojo p = new Pojo();
// 查看被修改以前的值
p.printName();
// 反射获取字段, name成员变量
Field nameField = p.getClass().getDeclaredField("name");
// 因为name成员变量是private, 因此须要进行访问权限设定
nameField.setAccessible(true);
// 使用反射进行赋值
nameField.set(p, new StringBuilder("111"));
// 打印查看被修改后的值
p.printName();
```

### 4.2 修改final修饰StringBuilder类型的字段

```
Pojo2 p = new Pojo2();
// 查看被修改以前的值
p.printName();
// 反射获取字段, name成员变量
Field nameField = p.getClass().getDeclaredField("name");
// 因为name成员变量是private, 因此须要进行访问权限设定
nameField.setAccessible(true);
// 使用反射进行赋值
nameField.set(p, new StringBuilder("111"));
// 打印查看被修改后的值
p.printName();
```

### 4.3 修改final修饰String类型的字段（失败）

```
Pojo3 p = new Pojo3();
p.printName();      //"default"
Field nameField = p.getClass().getDeclaredField("name");
nameField.setAccessible(true);
nameField.set(p, "111");
p.printName();      //"default"
```

**修改失败**

由于JVM在编译时期, 就把final类型的String进行了优化, 在编译时期就会把String处理成常量, 因此 Pojo3里的printName()方法被写死了：

```
public void printName() {
    System.out.println("default");
}
```

**而看似name修改失败，其实修改成功了**：

```
Pojo3 p = new Pojo3();
Field nameField = p.getClass().getDeclaredField("name");
nameField.setAccessible(true);
nameField.set(p, "111");
Object name = nameField.get(p);
System.out.println(name.toString());    //"111"
```

### 4.4 修改final修饰String类型的字段（成功）

阻止final修饰的String在JVM编译时就被处理为常量。

可以让final String类型的name的初始值**通过一次运行**才能获得, 那么就**不会**在编译时期就被处理为常量。

```
public class Pojo4 {
    // 防止JVM编译时就把"default4"做为常量处理
    private final String name = (null == null ? "default4" : "");
    //或者
    //private final String name = new StringBuilder("default5").toString();
    ...
}
//测试
Pojo4 p = new Pojo4();
p.printName();
Field nameField = p.getClass().getDeclaredField("name");
nameField.setAccessible(true);
nameField.set(p, "111");
p.printName();      //  "111"
```

### 4.5 修改static修饰StringBuilder类型的字段

成功

### 4.6 修改static final修饰StringBuilder类型的字段（失败）

```
Pojo7 p = new Pojo7();
p.printName();
Field nameField = p.getClass().getDeclaredField("name");
nameField.setAccessible(true);
nameField.set(p, new StringBuilder("111"));
p.printName();
```

上述代码会抛异常，反射没法修改同时被static final修饰的变量。

### 4.6 修改static final修饰StringBuilder类型的字段（成功）

```
Field nameField = p.getClass().getDeclaredField("name");
nameField.setAccessible(true);
//去掉final修饰符
Field modifiers = nameField.getClass().getDeclaredField("modifiers");
modifiers.setAccessible(true);
modifiers.setInt(nameField, nameField.getModifiers() & ~Modifier.FINAL);
//再修改就正常了
nameField.set(p, new StringBuilder("111"));
//最后加回final修饰符
modifiers.setInt(nameField, nameField.getModifiers() & ~Modifier.FINAL);
```