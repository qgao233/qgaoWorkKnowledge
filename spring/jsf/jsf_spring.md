# 集成JSF与Spring

参考：[Spring JSF集成教程](https://blog.csdn.net/Aria_Miazzy/article/details/88382208)

---

集成两者很简单，首先要知道使用jsf的项目，从后端获取数据的方式都是在前台使用el表达式获取，只要将这个el表达式由servlet换成从spring容器获取即可。

关键点则在于，如何让jsf识别spring的bean，而不是servlet。

## 1 环境准备

### 1.1 pom依赖

```
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>4.1.4.RELEASE</version>
</dependency>
 
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-web</artifactId>
	<version>4.1.4.RELEASE</version>
</dependency>
 
<dependency>
	<groupId>javax.inject</groupId>
	<artifactId>javax.inject</artifactId>
	<version>1</version>
</dependency>
```

### 1.2 添加DelegatingVariableResolver

**重点**：在[faces-config.xml](./facesconfig.md)中添加`DelegatingVariableResolver`。

```
<application>
    <el-resolver>
        org.springframework.web.jsf.el.SpringBeanFacesELResolver
    </el-resolver>
</application>
```

从名字可以看出`el-resolver`标签是委托变量解析​​器，SpringBeanFacesELResolver将jsf中的el解析为spring中的bean，即将**JSF前端**连接到**Spring后端**服务层。

### 1.3 添加spring框架监听器

```
<listener>
	<listener-class>
		org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>
<listener>
	<listener-class>
		org.springframework.web.context.request.RequestContextListener
	</listener-class>
</listener>
```

## 2 测试例子

### 2.1 JSF页面car.xhtml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
	xmlns:h="http://java.sun.com/jsf/html"
	xmlns:ui="http://java.sun.com/jsf/facelets">
<h:head>
	<title>JSF Spring Integration</title>
</h:head>
<h:body>
 
	<h2>Car Names List</h2>
 
	<ul>
 
		<ui:repeat var="cars" value="#{carBean.fetchCarDetails()}">
			<li><h3>#{cars}</h3></li>
		</ui:repeat>
 
	</ul>
</h:body>
</html>
```

### 2.2 spring bean：CarBean.java

```
import java.util.List;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.SessionScoped;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
 
@Component
@ManagedBean
@SessionScoped
public class CarBean {
 
	@Autowired
	CarDao carDao;
 
	public void setCarDao(CarDao carDao) {
		this.carDao = carDao;
	}
 
	public List<String> fetchCarDetails() {
 
		return carDao.getCarDetails();
	}
 
}
```

### 2.3 配置包扫描

applicationContext.xml

```
<context:component-scan base-package="com.journaldev.jsfspring" />
```