# Python 类型提示

python是弱类型语言，而Python 3.6+ 这一特点就是显式地将变量的类型展示出来。

```
def get_full_name(first_name: str, last_name: str):
    full_name = first_name.title() + " " + last_name.title()
    return full_name


print(get_full_name("john", "doe"))
```

这和声明默认值是不同的，例如：

```
first_name="john", last_name="doe"
```

有了类型提示后，在调用变量后，通过`.`可以拥有代码补全（即显示出该对象有哪些方法可以调用）。