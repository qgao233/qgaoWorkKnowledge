# 安装jar包到本地仓库

## 1 手动安装

参考：[使用maven安装jar包到本地仓库时遇到The goal you specified requires a project to execute but there is no POM in thi...](https://blog.csdn.net/weixin_30682415/article/details/101948894)

将相关参数加上引号,如下:

```
mvn install:install-file "-Dfile=D:\setup\fastdfs-client-java-1.27-RELEASE.jar" "-DgroupId=org.csource" "-DartifactId=fastdfs-client-java"  "-Dversion=1.27-SNAPSHOT"  "-Dpackaging=jar"
```

## 2 自动安装

将需要安装的jar包配置在pom.xml中：

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-install-plugin</artifactId>
    <version>2.5</version>
    <executions>
        <execution>
            <id>install-JWT-SDK</id>
            <phase>initialize</phase>
            <goals>
                <goal>install-file</goal>
            </goals>
            <configuration>
                <groupId>com.idsmanager.dingdang</groupId>
                <artifactId>JWT-SDK</artifactId>
                <version>1.1.1_1.8</version>
                <packaging>jar</packaging>
                <file>${basedir}/lib/JWT-SDK-1.1.1_1.8.jar</file>
            </configuration>
        </execution>
        <execution>
            <id>install-mssql</id>
            <phase>initialize</phase>
            <goals>
                <goal>install-file</goal>
            </goals>
            <configuration>
                <groupId>com.microsoft.sqlserver</groupId>
                <artifactId>sqljdbc42</artifactId>
                <version>6.0.8112</version>
                <packaging>jar</packaging>
                <file>${basedir}/lib/sqljdbc42-6.0.8112.jar</file>
            </configuration>
        </execution>
    </executions>
</plugin>
```