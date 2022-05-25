# 第5周-5.11

## 1 Runtime.availableProcessors()

[Runtime.availableProcessors() 分析](https://blog.csdn.net/wanghao112956/article/details/100878026)

### 1.1 功能

* 返回jvm虚拟机可用核心数：返回机器可用的CPU数，
* 这个值有可能在虚拟机的特定调用期间更改。

### 1.2 返回可用核心数

在一个多核CPU服务器上，可能安装了多个应用，JVM只是其中的一个部分，有些cpu被其他应用使用了。

因此只能返回未被使用的cpu个数。

### 1.3 jdk源码实现

#### 1.3.1 windows

不仅需要判断CPU是否可用，还需要依据CPU亲和性去判断是否该线程可用该CPU。里面通过一个while循环去解析CPU亲和性掩码，因此这是一个**CPU密集型**的操作。

#### 1.3.2 linux

linux 实现比较懒，直接通过`sysconf`函数读取系统参数`_SC_NPROCESSORS_ONLN`。

### 1.4 性能比较

分**正常工作**和**cpu满负荷工作**进行比较。

#### 1.4.1 测试代码

```
public class RuntimeDemo {

    private static final int EXEC_TIMES = 100_0000;
    private static final int TEST_TIME = 10;

    public static void main(String[] args) throws Exception{
        int[] arr = new int[TEST_TIME];
        for(int i = 0; i < TEST_TIME; i++){
            long start = System.currentTimeMillis();
            for(int j = 0; j < EXEC_TIMES; j++){
                Runtime.getRuntime().availableProcessors();
            }
            long end = System.currentTimeMillis();
            arr[i] = (int)(end-start);
        }

        double avg = Arrays.stream(arr).average().orElse(0);
        System.out.println("avg spend time:" + avg + "ms");

    }
}
```

#### 1.4.2 cpu满负荷代码

```
public class CpuIntesive {

    private static final int THREAD_COUNT = 16;

    public static void main(String[] args) {
        for(int i = 0; i < THREAD_COUNT; i++){
            new Thread(()->{
                long count = 1000_0000_0000L;
                long index=0;
                long sum = 0;
                while(index < count){
                    sum = sum + index;
                    index++;
                }
            }).start();
        }
    }
}
```

#### 1.4.3 开始测试

* 正常工作：直接跑`测试代码`；
* 满负荷工作：先把`cpu满负荷代码`跑起来，再跑`测试代码`。

#### 1.4.4 测试结果

系统|配置|测试方法|测试结果
:-|:-|:-|:-
Windows|2核8G|正常|1425.2ms
Windows|2核8G|满负荷|6113.1ms
MacOS|4核8G|正常|69.4ms
MacOS|4核8G|满负荷|322.8ms

### 1.5 总结

可见该函数性能不错，实际使用时，可不考虑该函数性能损失。