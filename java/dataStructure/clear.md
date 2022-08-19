## 2 StringBuffer清空操作

[StringBuffer清空操作delete和setLength的效率对比分析](https://blog.csdn.net/qq_35559358/article/details/77448582)

`setLength()`方法用时较短，因此在StringBuffer 清空操作中，使用setLength(int newLength)方法效率较高。