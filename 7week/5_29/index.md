# 第7周-5.29

## 1 scala的Seq与java的List相互转换

[scala的Seq与java的List相互转换](https://blog.csdn.net/qq_41958123/article/details/103386013)

```
在使用sparksql的dataset时有些地方需要将seq和list进行互相转化使用，scala集合提供相关的转化操作：

//Seq 转 List
List<Column> list = scala.collection.JavaConversions.seqAsJavaList(seq);
//List 转 Seq
List<Column> list = new ArrayList<>();
list.add(new Column("columnA"));
Seq<Column> seq = JavaConverters.asScalaIteratorConverter(list.iterator()).asScala().toSeq();
```