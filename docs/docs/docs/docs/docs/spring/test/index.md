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