## 2 java 10 新特性 var

Java 10 可以让编译器帮我们推断其类型：

```
var 变量名 = 初始值;
```

* var只能在方法里定义，不允许定义类的成员变量。
* var每次只能定义一个变量，不能符合声明变量。（var a,b;是不允许的）

什么时候该用var定义变量：

如果你定义变量时，给变量赋给一个直观的值，这时就可以使用var定义变量，