# nio的Path,File

2022-4-19 15:1:24

>java7提供

在很多方面，java.**nio**.file.Path接口和java.**io**.File有相似性，但也有一些细微的差别。在很多情况下，可以用Path来代替File类。

### 3.1 路径

**windows下绝对路径**
```
Path path = Paths.get("c:\\data\\myfile.txt"); //1
Path path = Paths.get("/home/jakobjenkov/myfile.txt");//2. 会被解析成相对路径：base为C盘，从而对应绝对路径是：C:/home/jakobjenkov/myfile.txt
```

**linux下绝对路径**
```
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

### 3.2 api

#### 3.2.1 Paths

方法|解释
:-|:-
Path get(String,...)|获得Path对象

#### 3.2.2 Path

方法|解释
:-|:-
String toString()|返回调用 Path 对象的字符串表示形式
boolean starts With(String path)|判断是否以 path 路径开始
boolean ends With(String path)|判断是否以 path 路径结朿
boolean isAbsolute()|判断是否是绝对路径
Path getParento|返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
Path getRoot()|返回调用 Path 对象的根路径
Path getFileName|返回与调用 Path 对象关联的文件名
int getNameCount()|返回Path 根目录后面元素的数量
Path getName (int idx)|返回指定索引位置idx 的路径名称
Path toAbsolute Path()|作为绝对路径返回调用 Path 对象
Path resolve(Path p)|合并两个路径，返回合并后的路径对应的Path对象
File toFile()|将Path转化为File类的对象

#### 3.2.3 Files

**1 用于操作文件或目录**

方法|解释
:-|:-
Path copy(Path sc, Path dest, CopyOption ...how)|文件的复制
Path createDirectory(Path path, FileAttribute<?> ...attr)|创建一个自录
Path createFile(Path path, FileAttribute<?> ...arr)|创建一个文件
void delete (Path path)|删除一个文件/目录，如果不存在，执行报错
void deletelfExists (Path path)|Path对应的文件/目录如果存在，执行删除
Path move(Path src. Path dest. CopyOption...how)|将src移动到dest 位置
long size(Path path)|返回 path 指定文件的大小

**2 用于判断**

方法|解释
:-|:-
boolean exists(Path path, LinkOption opts)|判断文件是否存在
boolean isDirectory(Path path, LinkOption opts)|判断是否是目录
boolean isRegularFile(Path path. LinkOption...opts)|判断是否是文件
boolean isHidden(Path path)|判断是否是隐薇文件
boolean isReadable(Path path)|判断文件是否可读
boolean isWritable(Path path)|判断文件是否可写
boolean notExists(Path path. LinkOptionu opts)|判断文件是否不存在

**3 用于操作内容**

方法|解释
:-|:-
SeekableByteChannel newByte Channel(Path path OpenOption...how)|获取与指定文件的连接，how 指定打开方式。
DirectoryStream<Path> newDirectoryStream(Path path) |打开path指定的目录
InputStream newlnputStream(Path path. OpenOption ... how)|获取InputStream对象
OutputStream newOutputStream(Path path, OpenOption ... how) |获取OutputStream对象

**4 其它**

`Files.lines()`

以Stream流的形式读取文件的所有行

```
/**
 * 以流的形式读取文件的所有行
 * 读取的的字节是以UTF-8解码的字符集
 * 
 * @param path 文件的路径
 * @return Stream<String> 文件中的行组成的流
 * @throws IOException 出现IO错误时抛出该异常
 * @throws SecurityException 如果是默认提供程序,则安全管理器是已安装,检查读取方法来检查对文件的读取访问
 */
public static Stream<String> lines(Path path) throws IOException {}

/**
 * 以流的形式读取文件的所有行
 * 该方法和readAllLines不同,不会将所有行读取到一个List中,而是以流的形式进行惰性加载
 * 以指定的解码的字符集读取字节,支持readAllLines的行终止符
 * 当该方法返回时,后续读取文件发生的IOException将会在读取Stream流的方法处抛出一个包装的UncheckedIOException.
 * 如果关闭文件发生IOException也会包装成为一个UncheckedIOException
 * 返回的流封装了一个读取器,如果需要周期性的读取文件，需要使用try-with-resources语句来保证stream的close方法被调用,从而关闭打开的文件 
 * 
 * @param path 文件的路径
 * @param cs 指定的解码格式
 * @return Stream<String> 文件中的行组成的流
 * @throws IOException 出现IO错误时抛出该异常
 * @throws SecurityException 如果是默认提供程序,则安全管理器是已安装,检查读取方法来检查对文件的读取访问
 */
public static Stream<String> lines(Path path, Charset cs) throws IOException {}
```



### 3.3 NIO和IO遍历指定目录效果对比

利用IO的File递归遍历目录：

```
public static void showPath(String dir){
    File files = new File(dir);
    if (files.isDirectory()) {
        for (File file: files.listFiles()
             ) {
            System.out.println(file.getAbsolutePath());
            showPath(file.getAbsolutePath().toString());
        }
    }else{
        return;
    }
}
```

采用NIO的Files.walk方法，借助Stream流 直接逐一把目录打印出来：
```
public static void printDir(String dir){
    Path path = Paths.get(dir);
    try {
        Files.walk(path).forEach(System.out::println);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
>汽车为你造好了，没必要再骑车上高速了。