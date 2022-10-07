## 1 maven

2022-4-20 9:57:7

### 1.1 生命周期

**1 compile**

`mvn compile`会在当前目录生成target目录。

**2 clean**

`mvn clean`删除当前目录中的target目录。

**3 package**

`mvn package`将本地工程打包成jar包。

**4 install**

`mvn install`将本地工程打包成jar包，放入到本地仓库中。

>之后，其它Maven管理的项目可以通过pom.xml配置依赖引入到其工程。

**其它**

`mvn -DskipTests`，不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下。

`mvn -Dmaven.test.skip=true`，不执行测试用例，也不编译测试用例类。

### 1.2 maven寻找依赖流程

1. Maven 将从本地资源库获得 Maven 的本地资源库依赖资源(默认用户目录下的.m2目录)，如果没有找到，
2. 它会从默认的 Maven 中央存储库 – http://repo1.maven.org/maven2/ 查找下载，如果还是没有找到，
3. 它会从[配置](./repository.md)的远程存储库（包括私服、JBOSS仓库和java.net仓库）查找下载。

### 1.3 idea里的maven wrapper

要使用maven wrapper有2种方式：

**1、 主动生成：**
```
mvn -N io.takari:maven:0.7.6:wrapper
```

会生成3个文件：

1. mvnw(linux命令)
2. mvnw.cmd(windows命令)
3. .mvn目录

**2、从别的项目中将这3个文件copy过来**

有了这3个文件后，

maven的具体版本只依赖于当前项目中(.mvn/wrapper/maven-wrapper.properties)所配置的maven版本。

在项目目录下，使用mvnw代替以前的mvn命令。

mvnw会在每次执行命令时检测${home}/.m2/wrapper/dists目录下是否有maven-wrapper.properties中指定Maven版本，如果没有就将maven环境自动下载到dists目录。

>maven的配置文件settings.xml必须在{user}/.m2目录下