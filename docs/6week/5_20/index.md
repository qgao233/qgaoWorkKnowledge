# 第6周-5.20

## 1 oracle处理表的字段

[Oracle 中对一个表中多个列的增加和删除的sql语句](https://blog.csdn.net/bestcxx/article/details/49514079)
[oracle怎么修改表字段类型](https://m.php.cn/oracle/486482.html)

### 1.1 增加一列/多列

```
alter table 表名 add(列名1 VARCHAR2(20)< ,列名2 VARCHAR2(64 char)>);
```

### 1.2 删除一列/多列

```
alter table 表名 drop(列名1< ,列名2>);
```

### 1.3 修改表字段类型

#### 1.3.1 字段数据为空

```
alter table tb modify (name nvarchar2(20));
```

#### 1.3.2 字段有数据

* 改为`nvarchar2(20)`可以直接执行：

```
alter table tb modify (name nvarchar2(20));
```

* 改为`varchar2(40)`执行：

```
alter table tb modify (name varchar2(40));
```

>会弹出“ORA-01439:要更改数据类型,则要修改的列必须为空”，

**得用这个方法：**

```
/*修改原字段名name为name_tmp*/
alter table tb rename column name to name_tmp;
/*增加一个和原字段名同名的字段name*/
alter table tb add name varchar2(40);
/*将原字段name_tmp数据更新到增加的字段name*/
update tb set name=trim(name_tmp);
/*更新完，删除原字段name_tmp*/
alter table tb drop column name_tmp;
```