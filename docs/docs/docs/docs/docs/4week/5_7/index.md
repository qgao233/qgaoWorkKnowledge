# 第4周-5.7

## 1 HttpURLConnection

[HttpURLConnection](https://www.apiref.com/android-zh/java/net/HttpURLConnection.html)

## 2 DataOutputStream和DataInputStream

[Java语言DataOutputStream和DataInputStream使用方法](https://blog.csdn.net/qq_43594119/article/details/109345771)

### 2.1 DataOutputStream

```
//创建数据写入流(写入到文件),第二个参数为true意思是追加写入，而不是覆盖写入
DataOutputStream dos = new DataOutputStream(new FileOutputStream("water.txt",true));.

//写入数据
dos.writeInt(int类型的数据); //写入int类型
dos.writeDouble(double类型的数据); //写入double类型
...
```

### 2.2 DataInputStream

>注意：这里的读取顺序要和写入顺序一样，且次数也要一样

```
//创建数据读取流(从文件读取)
DataInputStream dis = new DataInputStream(new FileInputStream("water.txt"));

//读取数据
dis.readInt() //获取int类型
dis.readDouble() //获取double类型
```

### 2.3 写入文件打开显示乱码

**乱码原因：**

和ObjectOutputStream.writeObject()乱码的原因差不多，DataOutputStream写入文件，直接打开就是乱码的，只有用DataInputStream读取才能还原，而且读取的顺序和次数必须和写入的顺序次数一样。

>ObjectOutputStream.writeObject()的作用是把一个实例的对象以文件的形式保存到磁盘上，
>这个过程就叫Java对象的持久化。而这个文件是以二进制的形式编写的，当你用文本编辑器将它打开，这些二进制代码与某个字符集映射之后，显示出来的东西就成了乱码。（即使输出的是一个String的对象，也是以该String对象的二进制编码的形式输出，而不是输出String对象的内容。）我们需要通过ObjectInputStream来进行读取。