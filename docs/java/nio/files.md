## 4 Files.lines使用注意事项

[Files.lines()方法使用相关问题](https://blog.csdn.net/JewaveOxford/article/details/108404857)

### 4.1 使用try-with-resources

从path对应的文件中读取所有内容,并**按行分割**,返回一个Stream. 

因为是流，使用完后必须得关闭流，否则会报`/proc/stat: Too many open files`

使用try-with-resources语句来保证stream的close方法被调用,从而关闭打开的文件:

```
try(Stream<String> stream = Files.lines(Paths.get(file))){
    return stream.skip(start).limit(limit).collect(Collectors.toList());
} catch (IOException e){
    logger.error("get content from{} error,{}",file, e.getMessage());
}

//等价于：

Stream<String> stream = Files.lines(Paths.get(file));
try {
    return stream.skip(start).limit(limit).collect(Collectors.toList());
} catch (IOException e){
    logger.error("get content from{} error,{}",file, e.getMessage());
} finally {
    stream.close();
}
```