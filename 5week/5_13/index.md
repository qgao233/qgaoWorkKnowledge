# 第5周-5.13

## 1 spring通过配置与注解给字段赋值

### 2.1 @Value使用

需要无参构造方法。

#### 1 普通属性

```
@Value("${flag}")
public int flag;
```

不用写setter方法。

#### 2 静态属性

```
public static int flag;
```

想给上面的静态属性使用@Value赋值，必须得将@Value写在它的setter方法上：

```
@Value("${flag}")
public void setFlag(int flag){
    this.flag = flag
}
```

### 2.2 @ConfigurationProperties

需要无参构造方法和setter缺一不可。


## 2 子类与父类的bean管理

测试环境：

* `spring-boot-starter-parent 2.6.7`
* `java 8`

如果只在子类上注解了bean，或者说只在xml配置文件中配置了子类。

父类被初始化的同时，**也会被放入spring容器管理**。

### 2.1 测试

这里测试抽象父类，普通父类和其测试结果一样。

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

    protected String fatherName = "normalFather";

    @Value("${fatherName}")
    protected String fatherYmlName;

    public String getFatherYmlName() {
        return fatherYmlName;
    }

    public String getFatherName() {
        return fatherName;
    }

    public abstract String print();
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
}
```

**test类**

```
@SpringBootTest(classes = {TestingWebApplication.class})
public class Test1 {

    @Autowired
    private Son son;

    @Autowired
    private AbstractFather father;

    @Test
    public void test(){
        System.out.println(son.getName());
        System.out.println(son.getFatherName());
        System.out.println(son.print());
        System.out.println(father.getFatherName());
        System.out.println(father.getFatherYmlName());
    }
}
```

### 2.2 测试结果

```
son
normalFather
son -> print
normalFather
father
```