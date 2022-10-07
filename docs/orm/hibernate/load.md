# hibernate -- load/get

参考：[getHibernateTemplate.load()一直无法找到数据库中存在的数据](https://blog.csdn.net/lopper/article/details/4801997)

---

HibernateTemplate的

* load方法可以充分利用hibernate的一级缓存和二级缓存中的现有数据，
* 而get方法仅仅在一级缓存中进行数据查找，如果没有发现数据則将越过二级缓存，直接调用SQL查询数据库。

## 1 问题

如果中途别人把数据库中的数据修改了，

* load如果在二级缓存中找到了数据，则不会再访问数据库，
* 而get在一级缓存中没找到就直接查数据库，因此会返回最新数据。