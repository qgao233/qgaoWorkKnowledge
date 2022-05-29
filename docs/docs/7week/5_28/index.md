# 第7周-5.28

## 1 注解继承(扩展限定)

[@Qualifier扩展限定](https://www.jianshu.com/p/73ef9519cbd7)

**当注解A用在注解B上称之为：注解B继承了注解A。**

又可以说成：注解B的**扩展注解B限定描述符注解**为注解A。

接下来看个有关`@Qualifer`扩展的例子：

```
@Target( {ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE} )
@Retention(RetentionPolicy.RUNTIME)
@Qualifier                          //注解A
public @interface MyQualifier{}     //注解B

//使用
public class Main3 (
    @Autowired
    public List<User> userList;

    @Autowired
    @Qualifier
    public List<User> qualifierList;

    @Autowired
    @MyQualifier
    public List<User> myQualifierList;

    @Bean
    @Qualifier
    public User get1() {
        return new User(1,"1"); 
    }

    @Bean
    @Qualifier
    public User get2() {
        return new User(2,"2"); 
    }

    @Bean
    @MyQualifier
    public User get3(){
        return new User(3,"3"); 
    }

    @Bean
    @MyQualifier
    public User get4(){
        return new User(4,"4"); 
    }

    @Bean
    public User get5(){
        return new User(5,"5"); 
    }
}
```

client测试：

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.register (Main3.class);
    context.refresh();

    Main3 bean = context.getBean(Main3.class);

    System.out.println(bean.userList);
    System.out.println(bean.qualifierList);
    System.out.println(bean.myQualifierList);

    context.close();
}
```

上例中`@Qualifier`用在`@MyQualifier`上，那么`@MyQualifier`就是`@Qualifier`的子注解。

>因为`@Qualifier`注解可以用在注解上,`当@Qualifier`用在其他注解上,相当于其他注解也拥有了**区分该注入哪个Bean**的能力。

`myQualifierList`被`@MyQualifier`修饰，那么其自动注入时只会注入被`@MyQualifier`修饰的Bean`{User.id=3,User.id=4}`的Bean。

`qualifierList`被`@Qualifier`修饰,因为`@Qualifier`是`@MyQualifier`的父注解 那么`qualifierList`自动注入时会注入被`@MyQualifier`和`@Qualifier`修饰的Bean`{User.id=3,User.id=4,User.id=1,User.id=2}`的Bean。

## 2 java提供有关依赖注入的注解

[依赖注入 javax.inject中@Inject、@Named、@Qualifier和@Provider用法](https://blog.csdn.net/qq_34120041/article/details/53672199)

这个是 Java EE 6 规范 JSR 330 -- Dependency Injection for Java 中的东西，也就是 **Java EE 的依赖注入**。

spring ==|== JSR-330
-|-
@Autowired的缺省情况|@Inject
@Component|@Named无值时的缺省情况
**扩展@Qualifier限定描述符注解**情况|@Qualifier