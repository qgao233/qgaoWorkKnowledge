# `__name__=='__main__'`

[Python `__name__=='__main__'`作用详解](http://c.biancheng.net/view/4643.html)

## 1 A模块

```
def test():
    print("A")

test()
```

输出一个A。

## 2 B模块

```
import A

A.test()
```

输出：

```
A
A
```

## 3 修改A，运行B

```
def test():
    print("A")

if __name__=='__main__':
    test()
```

输出一个A，此时是`B`引入`A`之后，在`B`中的输出。

## 4 在B中测试`__name__`

```
import A
print(__name__)     # __main__
print(A.__name__)   # A
```

可以看见`A`被引入`B`中之后，输出它本身模块的名字，因为`A`已经不是运行主模块了，只有真正运行的主模块的`__name__`才是`__main__`，其他被引入的`__name__`只能是本身模块的名字。