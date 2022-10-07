## 1 spring通过配置与注解给字段赋值

### 1.1 @Value使用

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

### 1.2 @ConfigurationProperties

需要无参构造方法和setter缺一不可。