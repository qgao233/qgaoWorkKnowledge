# faces-config.xml

如果未在web.xml或web-fragment.xml中显式映射此配置文件,并且满足以下一个或多个条件,则必须自动映射此配置文件.

* 在WEB-INF中可以找到`faces-config.xml`文件.

* `faces-config.xml`文件位于应用程序类路径中jar的META-INF目录中.

* 以`.faces-config.xml`结尾的文件名位于应用程序类路径中jar的META-INF目录中.

* javax.faces.CONFIG_FILES上下文参数在web.xml或web-fragment.xml中声明.

* 传递给ServletContainerInitializer实现的onStartup()方法的类集不为空.

您可以将配置文件放在webapp中的其他位置,但需要记住以下事实:

您不希望将文件公开为可访问,因此您希望它最终位于WEB-INF/ META-INFfolders下;
您需要将以下条目添加到web.xml:

```
<context-param>
    <param-name>javax.faces.CONFIG_FILES</param-name>
    <param-value>
        WEB-INF/path/to/`faces-config.xml`
   </param-value>
</context-param>
```

否则,如果在某些预定义位置找到`faces-config.xml`,则会自动加载.