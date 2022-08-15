# junit的Assert

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

