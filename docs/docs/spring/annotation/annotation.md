# aop 中使用@annotation寻找切入点

[spring aop 中@annotation()的使用](https://www.cnblogs.com/fanheyan/articles/10366682.html)

## 1 普通切面写法

```
//切入点签名
@Pointcut("execution(* com.lxk.spring.aop.annotation.PersonDaoImpl.*(..))")
private void aa() {
}

//然后在不同的 advice 里面使用
//后置通知
@AfterReturning(value = "aa()", returning = "val")
public void afterMethod(JoinPoint joinPoint, Object val) {}


//异常通知，joinPoint有需要可以加在入参里
@AfterThrowing(value = "aa()", throwing = "ex")
public void throwingMethod(Throwable ex) {}
```

## 2 使用@annotation

自定义注解：

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodLog {
    public String name() default "fieldName";
}
```

**第一种写法：**

```
//可以拿到自定义的注解，以及注解上的参数
@After(value = "aa() && @annotation(methodLog)", argNames = "joinPoint, methodLog")
public void methodAfter(JoinPoint joinPoint, MethodLog methodLog) throws Throwable {}
```

**第二种写法**，不用显示的声明切入点，直接将`@Pointcut`的表达式写在`advice`上：

```
//value里面的意思，就是复合那个切入的点的条件， 以&&连接，也就是说2个都符合
@Around(value = "(execution(* com.lxk.service..*(..))) && @annotation(methodLog)", argNames = "joinPoint, methodLog")
public Object methodAround(ProceedingJoinPoint joinPoint, MethodLog methodLog) throws Throwable {}
```

**第三种写法**，最终简化如下：

```
@Around(value = "@annotation(methodLog)")
public Object methodAround(ProceedingJoinPoint joinPoint, MethodLog methodLog) throws Throwable {}
```

### 2.1 总结

考虑到自定义注解命名的时候，可能取的名字很大众化，其他的jar包，也就是你项目引入的jar包，可能有重名的注解，如果要是不加包限制的话，那估计就会出现意想不到的效果。

因此写法都是千篇一律地使用 `&&` 符号：**限制包**，然后**限制**使用的是哪个注解。

即**第二种写法**是最常用的！！！