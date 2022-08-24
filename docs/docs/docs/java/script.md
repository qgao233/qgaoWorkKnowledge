## 1 java里执行第三方脚本

[When Runtime.exec() won't](https://links.jianshu.com/go?to=%255Bhttps%3A%2F%2Fwww.javaworld.com%2Farticle%2F2071275%2Fwhen-runtime-exec---won-t.html%255D%28https%3A%2F%2Fwww.javaworld.com%2Farticle%2F2071275%2Fwhen-runtime-exec---won-t.html%29)

[简析 Runtime.exec(..)](https://www.jianshu.com/p/65503dfc7488)

java 程序中，如果我们想执行一些 Shell 命令或其他外部应用程序，通常都是使用java.lang.Runtime.exec(..)方法来执行的。

当 Java 内置的 Runtime.exec(..) 方法在执行外部命令时，可能存在一些不易察觉的坑，往往会导致程序运行失败。

### 1.1 Process.exitValue()

IllegalThreadStateException异常：Runtime.exe(..)可能出现IllegalThreadStateException异常。我们通常使用exec(..)方法执行 JVM 外部程序，如果想查看外部程序返回值，可以使用Process.exitValue()方法。

需要注意的是，**如果直接使用Process.exitValue()获取外部程序返回值，如果此时外部程序还未运行完成，则会抛出IllegalThreadStateException异常。**

举个栗子：比如在 Java 中执行 javac 程序，并获取其返回值。
代码如下：
```
public class BadExecJavac {
    public static void main(String[] args) throws IOException {
        Process pid = Runtime.getRuntime().exec("javac");
        int exitValue = pid.exitValue();
        System.out.println("Process exitValue: " + exitVal);
    }
}
```

>使用Process.waitFor()替换Process.exitValue()。

### 1.2 Process.waitFor()

**Process.waitFor()与Process.exitValue()同样会返回外部程序的执行结果，但是它会阻塞直到外部程序运行结束。**

代码如下：

```
public static void main(String[] args) throws IOException, InterruptedException {
    Process pid = Runtime.getRuntime().exec("javac");
    int exitValue = pid.waitFor();
    System.out.println("Process exitValue: " + exitValue);
}
```

>注：Process.exitValue()/Process.waitFor()获取外部程序的返回值为 0 表示执行成功，其余值表示外部程序执行出错。

综上：要解决IllegalThreadStateException异常，
* 要么就是手动捕获Process.exitValue()抛出的异常，
* 要么就使用Process.waitFor()（推荐）等待外部程序正常运行结束。
