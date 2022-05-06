# 第2周-4.19

## 1 junit的Assert

2022-4-19 11:50:4

>assert: 断言

[junit断言总结](https://blog.csdn.net/weixin_34342992/article/details/94621347)

平时编写自己的测试类，如果没有断言，那么就没写测试的必要了。

JUnit框架用一组assert方法封装了最常见的测试任务。

junit中的assert方法全部放在Assert类中，都是静态方法。（因此可以静态引用）

方法|解释
:-|:-
assertTrue/False([String message,]boolean condition);|判断一个条件是true还是false
fail([String message,]);|失败，可以有消息，也可以没有消息。
assertEquals([String message,]Object expected,Object actual);|判断是否相等，可以指定输出错误信息。第一个参数是期望值，第二个参数是实际的值。
assertNotNull/Null([String message,]Object obj);|判读一个对象是否非空(非空)。
assertSame/NotSame([String message,]Object expected,Object actual);|判断两个对象是否指向同一个对象。看内存地址。
failNotSame/failNotEquals(String message, Object expected, Object actual)|当不指向同一个内存地址或者不相等的时候，输出错误信息。

>说白了，就是先提前断言（判断）想要得到的结果，如果与预想的不一致，Assert类就会抛出一个AssertionFailedError异常，Junit测试框架将这种错误归入Fails并且加以记录。

## 2 Java 开发的模拟测试框架: Mockito

2022-4-19 11:50:11

https://baijiahao.baidu.com/s?id=1727997644914260506&wfr=spider&for=pc
https://blog.csdn.net/xiao__jia__jia/article/details/115252780

解决：

* 如何测试一个rest接口；
* 如何测试一个包含客户端调用服务端的复杂方法；
* 如何测试一个包含从数据库读取数据的复杂方法;
* ...

>假设现在要测试method A, method A里面又依赖Method B、Method C、Method D，而依赖的这3个method又不好去构建（如ObsClient需要真实AK SK，HttpClient需要构建客户端与服务器，Database相对好构建，但是假设Method C只是从table1、table2联合查询，你还得分别往table1、table2 insert数据，很繁琐），这时候可以考虑Mockito进行优雅测试，当然如果你想去构建真实的测试场景，未免有点舍本逐末了

说白了，就是构造一些虚拟数据，来测试方法，避免大动干戈。

使用（可以静态引用）：

### 2.1 验证行为

一旦mock对象被创建了，mock对象会记住所有的交互。然后你就可以选择性的验证你感兴趣的交互，也就是说，你对这个Mock对象做的操作，都被记录下来了，验证的时候会是合法的。

```
@Test
void testBehavior() {
 
    //构建moock数据
    List<String> list = mock(List.class);
    list.add("1");
    list.add("2");
 
    System.out.println(list.get(0)); // 会得到null ，前面只是在记录行为而已，没有往list中添加数据
 
    verify(list).add("1"); // 正确，因为该行为被记住了
    verify(list).add("3");//报错，因为前面没有记录这个行为
}
```

### 2.2 Stub（存根/打桩）

* 存根：用于预先说明当执行了什么操作的时候，产生一个什么响应，


存根特点：默认情况下，所有的函数都有返回值。

mock函数默认返回的是：
* null，
* 一个空的集合或者
* 一个被对象类型包装的内置类型，例如0、false对应的对象类型为Integer、Boolean；


>测试桩函数可以被覆写 : 例如常见的测试桩函数可以用于初始化夹具，但是测试函数能够覆写它。请注意，覆写测试桩函数是一种可能存在潜在问题的做法；一旦测试桩函数被调用，该函数将会一致返回固定的值；


示例：**常规存根**，主要用到when 、thenReturn、then、thenThrow、thenAnswer这几个函数。

```
@Test
void testStub() {
    List<Integer> l = mock(ArrayList.class);
 
    when(l.get(0)).thenReturn(10);
    when(l.get(1)).thenReturn(20);
    when(l.get(2)).thenThrow(new RuntimeException("no such element"));
 
    assertEquals(l.get(0), 10);
    assertEquals(l.get(1), 20);
    assertNull(l.get(4));
    assertThrows(RuntimeException.class, () -> {
        int x = l.get(2);
    });
}
```

**void函数存根**

主要用到doThrow(), doAnswer(), doNothing(), doReturn() and doCallRealMethod()等函数，当然，这些函数也可以用于有返回值的函数的存根；
>需要注意的是，doCallRealMethod不能用于Mock对象，只能用于监察对象。

```
@Test
void testVoidStub(){
    List<Integer> l = mock(ArrayList.class);
    doReturn(10).when(l).get(1);
    doThrow(new RuntimeException("you cant clear this List")).when(l).clear();
 
    assertEquals(l.get(1),10);
    assertThrows(RuntimeException.class,()->l.clear());
}
```

### 2.3 参数匹配器

在上面的stub测试中，我们需要对我们关系的数据一个一个的进行stub，如果数据过多的时候，会比较麻烦，

这个时候，我们可以利用参数匹配器来完成相应的操作，参数既可以用与stub，也可以用于验证的时候。
>所有的参数匹配器都存放在org.mockito.ArgumentMatchers中。

```
@Test
void testMatchers() {
    List<Integer> l = mock(ArrayList.class);
    when(l.get(anyInt())).thenReturn(100);
 
    assertEquals(l.get(999),100);
}
```

### 2.4 调用次数

校验某个操作执行的次数

```
@Test
void testTimes() {
    List<Integer> l = mock(ArrayList.class);
    when(l.get(anyInt())).thenReturn(100);
    System.out.println(l.get(0));
    System.out.println(l.get(1));
    System.out.println(l.get(1));
    System.out.println(l.get(2));
    System.out.println(l.get(2));
    System.out.println(l.get(2));
    System.out.println(l.get(2));
 
    verify(l, times(7)).get(anyInt());
    verify(l, atLeastOnce()).get(0);
    verify(l, atLeast(3)).get(2);
    verify(l, atMost(6)).get(2);
    verify(l, atMostOnce()).get(0);
}
```

### 2.5 简化Mock创建方式(注解)

可以通过@Mock注解来简化Mock对象的创建过程，这样的话，我们就可以在多个测试中直接共用这些mock对象了，

>需要注意的是，我们需要在方法开始执行下面的操作：MockitoAnnotations.initMocks

```
@Mock
private MyService myService;  //会将MyService进行Mock
 
@InjectMocks
private MyController MyController; // 会向MyController注入Mock对象, 例如：myService
 
@Before
public void init() {
    MockitoAnnotations.initMocks(this);
}  
```

### 2.6 间接依赖bean

如果一个Bean A依赖于Bean B，而Bean B又依赖于Bean C，现在想对A的接口进行测试，又想把C Mock掉。

例如：MyFirstService 间接依赖了MyService, 下面演示如何将MyService进行隔离。

使用mock生成一个虚拟的对象，这样不需要在spring-test.xml文件单独配置这个myservice的Bean。

```
@Profile("development")
@Configuration
public class MyServiceTestConfiguration {
    @Bean(name = "myService")
    @Primary   // spring在寻找时，优先使用该bean
    public MyService myService() {
        return Mockito.mock(MyService.class);
    }
}
```

测试:

```
@ContextConfiguration("classpath:spring-test.xml")
@RunWith(SpringJUnit4ClassRunner.class)
@ActiveProfiles("development")
public class myServiceImplTest {
    @Autowired
    private myFirstService myFirstService;
 
}
```

### 2.7 @spy神器
spy类的原理是，如果不打桩默认都会执行真实的方法，如果打桩则返回桩实现。

对 spy 变量打桩时，如果使用 when 去设置模拟值时，他里面的代码逻辑依然会被执行，只是mock了返回结果；

使用 doReturn 设置模拟值的话，则不会出现这个问题！
```
A aSpy = spy(new A());
//aSpy.set...();

when(aSpy.service()).thenReturn("spy!");//service()逻辑依然执行
doReturn("spy!").when(aSpy).service();//service()不会执行
```

注意：spy后的实例，原有处理逻辑还在，但内部对象还得靠自己手动set进去

两个方法同个方法名，参数类型分别为Object 和 Collection， anyObject（）无法区分Object 和Collection，会导致程序走真实逻辑。
可以这样处理： anyCollection() 匹配Collection 和 any(MyClass.class)

### 2.8 api
（一）Mockito
org.mockito.Mockito是mockito提供的核心api，提供了大量的静态方法，用于帮助我们来mock对象，验证行为等等，然后需要注意的是，很多方法都被封装在了MockitoCore类里面，下面对一些常用的方法做一些介绍。

方法|解释
:-|:-
mock|构建一个我们需要的对象；可以mock具体的对象，也可以mock接口。
spy|构建监控对象
verify|验证某种行为
when|当执行什么操作的时候，一般配合thenXXX 一起使用。表示执行了一个操作之后产生什么效果。
doReturn|返回什么结果
doThrow|抛出一个指定异常
doAnswer|做一个什么相应，需要我们自定义Answer；
times|某个操作执行了多少次
atLeastOnce|某个操作至少执行一次
atLeast|某个操作至少执行指定次数
atMost|某个操作至多执行指定次数
atMostOnce|某个操作至多执行一次
doNothing|不做任何处理
doReturn|返回一个结果
doThrow|抛出一个指定异常
doAnswer|指定一个操作，传入Answer
doCallRealMethod|返回真实业务执行的结果，只能用于监控对象


（二）ArgumentMatchers
用于进行参数匹配，减少很多不必要的代码

方法|解释
:-|:-
anyInt|任何int类型的参数，类似的还有anyLong/anyByte等等。
eq|等于某个值的时候，如果是对象类型的，则看toString方法
isA|匹配某种类型
matches|使用正则表达式进行匹配


（三）OngoingStubbing
OngoingStubbing用于返回操作的结果。

方法|解释
:-|:-
thenReturn|指定一个返回的值
thenThrow|抛出一个指定异常
then|指定一个操作，需要传入自定义Answer；
thenCallRealMethod|返回真实业务执行的结果，只能用于监控对象。

## 3 nio的Path,File

2022-4-19 15:1:24

>java7提供

在很多方面，java.**nio**.file.Path接口和java.**io**.File有相似性，但也有一些细微的差别。在很多情况下，可以用Path来代替File类。

### 3.1 路径

**windows下绝对路径**
```
Path path = Paths.get("c:\\data\\myfile.txt"); //1
Path path = Paths.get("/home/jakobjenkov/myfile.txt");//2. 会被解析成相对路径：base为C盘，从而对应绝对路径是：C:/home/jakobjenkov/myfile.txt
```

**linux下绝对路径**
```
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

### 3.2 api

**Paths**

方法|解释
:-|:-
Path get(String,...)|获得Path对象

**Path**

方法|解释
:-|:-
String toString()|返回调用 Path 对象的字符串表示形式
boolean starts With(String path)|判断是否以 path 路径开始
boolean ends With(String path)|判断是否以 path 路径结朿
boolean isAbsolute()|判断是否是绝对路径
Path getParento|返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
Path getRoot()|返回调用 Path 对象的根路径
Path getFileName|返回与调用 Path 对象关联的文件名
int getNameCount()|返回Path 根目录后面元素的数量
Path getName (int idx)|返回指定索引位置idx 的路径名称
Path toAbsolute Path()|作为绝对路径返回调用 Path 对象
Path resolve(Path p)|合并两个路径，返回合并后的路径对应的Path对象
File toFile()|将Path转化为File类的对象

**Files**

用于操作文件或目录

方法|解释
:-|:-
Path copy(Path sc, Path dest, CopyOption ...how)|文件的复制
Path createDirectory(Path path, FileAttribute<?> ...attr)|创建一个自录
Path createFile(Path path, FileAttribute<?> ...arr)|创建一个文件
void delete (Path path)|删除一个文件/目录，如果不存在，执行报错
void deletelfExists (Path path)|Path对应的文件/目录如果存在，执行删除
Path move(Path src. Path dest. CopyOption...how)|将src移动到dest 位置
long size(Path path)|返回 path 指定文件的大小

用于判断

方法|解释
:-|:-
boolean exists(Path path, LinkOption opts)|判断文件是否存在
boolean isDirectory(Path path, LinkOption opts)|判断是否是目录
boolean isRegularFile(Path path. LinkOption...opts)|判断是否是文件
boolean isHidden(Path path)|判断是否是隐薇文件
boolean isReadable(Path path)|判断文件是否可读
boolean isWritable(Path path)|判断文件是否可写
boolean notExists(Path path. LinkOptionu opts)|判断文件是否不存在

用于操作内容

方法|解释
:-|:-
SeekableByteChannel newByte Channel(Path path OpenOption...how)|获取与指定文件的连接，how 指定打开方式。
DirectoryStream<Path> newDirectoryStream(Path path) |打开path指定的目录
InputStream newlnputStream(Path path. OpenOption ... how)|获取InputStream对象
OutputStream newOutputStream(Path path, OpenOption ... how) |获取OutputStream对象

### 3.3 NIO和IO遍历指定目录效果对比

利用IO的File递归遍历目录：

```
public static void showPath(String dir){
    File files = new File(dir);
    if (files.isDirectory()) {
        for (File file: files.listFiles()
             ) {
            System.out.println(file.getAbsolutePath());
            showPath(file.getAbsolutePath().toString());
        }
    }else{
        return;
    }
}
```

采用NIO的Files.walk方法，借助Stream流 直接逐一把目录打印出来：
```
public static void printDir(String dir){
    Path path = Paths.get(dir);
    try {
        Files.walk(path).forEach(System.out::println);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
>汽车为你造好了，没必要再骑车上高速了。