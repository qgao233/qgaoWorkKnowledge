# ArrayList在for循环中使用remove方法移除元素

## 1 list.remove

参考：[ArrayList在for循环中使用remove方法移除元素](https://blog.csdn.net/u012316120/article/details/51509066)

### 1.1 正着遍历漏值

先访问后删除。

```
//边循环边删除
for (int i = 0; i < list.size(); i++) {
	System.out.println("i的值:" + i + " 对应的数字:" + list.get(i));
	if(i == 3) list.remove(i);//删除list的第四项
}
```

删除前：

size|1|2|3|4|5|6
:-:|:-:|:-:|:-:|:-:|:-:|:-:
值|0|1|2|3|4|5
索引`i`|0|1|2|3|4|5

删除中：

`i = 3`，打印的值为3：

size|1|2|3|4|5|6
:-:|:-:|:-:|:-:|:-:|:-:|:-:
值|0|1|2|~~3~~ 4|5
索引`i`|0|1|2|**·3·**|4|5

size缩小为5，`i = 4`，打印的值为5：

size|1|2|3|4|5|-
:-:|:-:|:-:|:-:|:-:|:-:|:-:
值|0|1|2|~~3~~ 4|5
索引`i`|0|1|2|3|**·4·**

**可见`值=4`被漏掉了**，因为4跑到了索引为3的头上。

### 1.2 反着遍历ok

先访问后删除。

```
//边循环边删除
for (int i = list.size() -1 ; i >= 0; i--) {
	System.out.println("i的值  " + i + " 对应的数字 " + list.get(i));
	if(i == 3) list.remove(i);
}
```

删除前：

size|1|2|3|4|5|6
:-:|:-:|:-:|:-:|:-:|:-:|:-:
值|5|4|3|2|1|0
索引`i`|5|4|3|2|1|0

删除中：

`i = 3`，打印的值为3：

size|1|2|3|4|5|6
:-:|:-:|:-:|:-:|:-:|:-:|:-:
值||5|4 ~~3~~|2|1|0
索引`i`|5|4|**·3·**|2|1|0

size缩小为5，`i = 2`，打印的值为2：

size|-|1|2|3|4|5
:-:|:-:|:-:|:-:|:-:|:-:|:-:
值||5|4 ~~3~~|2|1|0
索引`i`||4|3|**·2·**|1|0

**可见值全部打印了**。

## 2 iterator.remove

参考：[Iterator remove()详解](https://blog.51cto.com/tianxingzhe/1693218)

* iterator.remove：删除当前指针所指向的元素，一般和`next()`一起用，这时候的作用就是删除`next()`返回的元素。

### 2.1 iteraotr内部原理

Iterator 工作在一个独立的线程中，并且拥有一个 `mutex` 锁。 

Iterator 被创建之后会建立一个指向原来集合对象的**单链索引表**，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照 `fail-fast` 原则：
	
  * Iterator 会马上抛出 java.util.ConcurrentModificationException 异常。

因此 Iterator 在工作的时候**不允许**迭代的对象被改变。

但可以使用 Iterator 本身的方法 remove() 来删除对象， Iterator.remove() 方法会在删除当前迭代对象的**同时维护**索引的一致性。

  
1. **单线程**：不允许在Iterator迭代中更改容器(add, delete....)，但可以在迭代的时候采用iterator.remove()方法确保迭代器在查找next的时候，指针不会丢失。

```
while(iterator.hasNext() {
	Object item = iterator.next();
	// list.remove(i);              //不允许
	// list.add(new Object());      //不允许
	iterator.remove();              //避免ConcurrentModificationException
	......
}
```

2. **多线程**：如果当前有多个线程在对容器进行操作，例如一个线程正在向容器中写数据，而另一个线程在迭代此容器，这时候就必须考虑并发下的线程安全问题。ConcurrentModificationException官方文档第一句就指出：
   >This exception may be thrown by methods that have detected concurrent modification of an object when such modification is not permissible.

   这时候可以采用`java.util.concurrent`包下面的线程安全的容器解决此异常。