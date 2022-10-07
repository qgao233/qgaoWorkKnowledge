## 2 CsvToBeanBuilder

[更优雅的解析文档中的字段-CsvToBeanBuilder](https://blog.csdn.net/qq_31289187/article/details/86104522)

优雅地将**结构化文件**转成java类。

### 2.1 maven依赖

```
<!-- 省略java bean的setter/getter -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.4</version>
</dependency>
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>4.2</version>
</dependency>
```

### 2.2 创建文件：user.txt

内容如下（一般线上的文件，都是严格按照规范的）

```
tiger|M|25|shanghai
hanzi|W|23|beijing
xiaoming|M|18|
xiaohua||12|
```

可见xiaohua的性别和地址不详，字段值都会为空值，方便后期文件解析以及处理。

### 2.3 UserBean

构造和文件对应的java类，一行即一个对象，每一列对应类的一个字段。

```
import com.opencsv.bean.CsvBindByPosition;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
 
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class User {
    //position为文件列的索引，从0开始，0表示第1列
    @CsvBindByPosition(position = 0)
    private String name;
    @CsvBindByPosition(position = 1)
    private String gender;
    @CsvBindByPosition(position = 2)
    private int age;
    @CsvBindByPosition(position = 3)
    private String address;
}
```

### 2.4 使用CsvToBeanBuilder

```
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.util.List;

public class ParseFileToBeanUtils {
 
    /**
     * 解析文件并返回beanList
     * @param filePath 文件路径
     * @param beanClass 对应的beanClass
     * @param separator 文件中使用的分割符
     * @return  返回beanList
     * */
    public static List<User> parseFileToUserBean(String filePath,Class beanClass,char separator) throws FileNotFoundException {
         List<User> users = new CsvToBeanBuilder(new FileReader(filePath))
                .withType(beanClass)            //构造成哪个java bean
                 .withSeparator(separator)      //结构化文件中数据之间的分隔符
                 .build()                       //开始构造
                 .parse();                      //转换解析成列表
         return users;
    }
 
    public static void main(String[] args) throws Exception{
        String filePath = "F:/user.txt";
        List<User> users = parseFileToUserBean(filePath,User.class,'|');
 
        users.forEach(user ->{
            System.out.println(user.toString());
        });
    }
}
```

### 2.5 运行结果

```
User(name=tiger, gender=M, age=25, address=shanghai)
User(name=hanzi, gender=W, age=23, address=beijing)
User(name=xiaoming, gender=M, age=18, address=)
User(name=xiaohua, gender=, age=12, address=)
```

### 2.6 常用api

方法|解释
:-|:-
`CsvToBeanBuilder<T> withFilter(CsvToBeanFilter filter)`|构造filter，过滤掉不想要的行记录。