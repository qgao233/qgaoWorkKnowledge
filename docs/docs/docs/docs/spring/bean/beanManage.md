## 2 子类与父类的bean管理

测试环境：

* `spring-boot-starter-parent 2.6.7`
* `java 8`

如果只在子类上注解了bean，或者说只在xml配置文件中配置了子类。

父类被初始化的同时，**只会把子类的实例放入spring容器管理**。

### 2.1 单子类测试

这里测试抽象父类，普通父类和其测试结果一样。

#### 1 测试代码

**依赖**

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.7</version>
    <relativePath></relativePath>
</parent>
```

**application.yml**

```
sonName: son
fatherName: father
```

**AbstractFather类**

```
public abstract class AbstractFather {

    private String fatherName = "normalFather";

    @Value("${fatherName}")
    private String fatherYmlName;

    public String getFatherYmlName() {
        return fatherYmlName;
    }

    public abstract String print();


    public String getFatherName() {
        return fatherName;
    }

    public void setFatherName(String fatherName) {
        this.fatherName = fatherName;
    }
}
```

**Son类**

```
@Component
public class Son extends AbstractFather{
    @Value("${sonName}")
    private String name;

    public String getName() {
        return name;
    }

    @Override
    public String print() {
        return "son -> print";
    }

    public void init(){
        setFatherName("son's father");
    }
}
```

**son2类**

```
@Override
public String print() {
    return "son2 -> print";
}

public void init(){
    setFatherName("son2's father");
}
```

**test类**

```
@SpringBootTest(classes = {TestingWebApplication.class})
public class Test1 {

    @Autowired
    private Son son;

    @Autowired
    private Son2 son2;

    @Autowired
    @Qualifier("son")      //bean的id，默认id为类名的驼峰法
    private AbstractFather father1;

    @Autowired
    @Qualifier("son2")
    private AbstractFather father2;

    @Test
    public void test(){
        System.out.println(son.getName());
        System.out.println(son.getFatherName());
        System.out.println(son.print());
        System.out.println(father.getFatherName());
        System.out.println(father.getFatherYmlName());
        System.out.println(father.print());
    }
}
```

#### 2 测试结果

```
son
normalFather
son -> print
normalFather
father
son2 -> print
```

#### 3 结果总结

可见即使父类没有被放入ioc容器，但因为子类在ioc容器中，父类的@Value依然可以正常工作。

### 2.2 多子类测试

为了模拟实际环境，采用多线程。

#### 1 测试代码

**testThreads**

```
@Test
public void testThreads() throws InterruptedException {
    new Thread(()->{
        try {
            son.init();
            System.out.println(Thread.currentThread().getName()+":"+son.getFatherName());
            Thread.sleep(100);
            System.out.println(Thread.currentThread().getName()+":"+son.getFatherName());

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    },"thread-1").start();
    Thread thread2 = new Thread(()->{
        try {
            son2.init();
            System.out.println(Thread.currentThread().getName()+":"+son2.getFatherName());

            Thread.sleep(500);
            System.out.println(Thread.currentThread().getName()+":"+son2.getFatherName());

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    },"thread-2");
    thread2.start();
    thread2.join();

    System.out.println(son);
    System.out.println(son2);
    System.out.println(father1);
    System.out.println(father2);
}
```

#### 2 测试结果

```
thread-1:son's father
thread-2:son2's father
thread-1:son's father
thread-2:son2's father
com.qgao.test.Son@1a5b8489
com.qgao.test.Son2@6f8f8a80
com.qgao.test.Son@1a5b8489
com.qgao.test.Son2@6f8f8a80
```

#### 3 结果总结

可见在spring环境下，虽然两个不同的子类（单例模式）都继承了相同的父类，但是父类的对象并没有纳入spring的ioc容器中。

### 2.3 总结两次测试

当子类被放入ioc容器，父类未被放入ioc容器，但父类中有@Value，

实际上是子类将父类中的被@Value注解的字段当成自己的了。