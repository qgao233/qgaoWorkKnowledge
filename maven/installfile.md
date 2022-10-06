# 安装jar包到本地仓库

参考：[使用maven安装jar包到本地仓库时遇到The goal you specified requires a project to execute but there is no POM in thi...](https://blog.csdn.net/weixin_30682415/article/details/101948894)

将相关参数加上引号,如下:

```
mvn install:install-file "-Dfile=D:\setup\fastdfs-client-java-1.27-RELEASE.jar" "-DgroupId=org.csource" "-DartifactId=fastdfs-client-java"  "-Dversion=1.27-SNAPSHOT"  "-Dpackaging=jar"
```