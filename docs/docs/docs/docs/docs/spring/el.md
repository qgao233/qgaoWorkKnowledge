## 1 spring EL表达式

起因是工作中碰到的一个bean设置property的问题，类似：

```
@Value("aa#{bb}")
private String name;
```

其中`#{bb}`的`#{}`是自定义的占位符，后期代码自己解析，结果和EL表达式的占位符冲突了，

因此需要写成下方的样子，将里面变成字符串，这样name依然是`aa#{bb}`：

```
@Value("#{'aa#{bb}'}")
private String name;
```

详细EL表达式使用见：

[Spring EL表达式](https://blog.csdn.net/weixin_39265427/article/details/120047221)
[spring的EL表达式](https://blog.csdn.net/u012045045/article/details/85338962)