# Spring和Ehcache整合详解

参考：[Spring和Ehcache整合详解](https://blog.csdn.net/feiyangtianyao/article/details/90692021)

Spring 提供了对缓存功能的抽象：本身不直接提供缓存功能的实现，但允许绑定不同的缓存解决方案（如Ehcache），spring支持注解方式使用缓存，非常方便。

* Ehcache是一个纯Java的进程内缓存框架，具有快速、精干等特点，使用的是本地的内存做缓存，
* redis则是部署在另一个server的缓存服务器，

>一般可以用redis + ehcache做两级缓存。

## 1 搭建环境

### 1.1 pom依赖

```
<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
		</dependency>
		<!-- ehcache 相关依赖 -->
		<dependency>
			<groupId>net.sf.ehcache</groupId>
			<artifactId>ehcache</artifactId>
		</dependency>
		<dependency>
			<groupId>cn.pomit</groupId>
			<artifactId>Mybatis</artifactId>
			<version>${project.version}</version>
		</dependency>
	</dependencies>
```

具体版本看自己。

### 1.2 xml配置

spring-ehcache.xml

```
<context:annotation-config />
    <context:component-scan base-package="cn.pomit.springwork">
</context:component-scan>

<!-- 声明了EhCacheManagerFactoryBean的bean，读取ehcache.xml的配置。
ehcache.xml是用来配置缓存的名字。只有配置的名字才可以在代码中被使用。 -->
<bean id="ehcache"
    class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
    <property name="configLocation" value="classpath:ehcache/ehcache.xml" />
</bean>

<bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager">
    <property name="cacheManager" ref="ehcache" />
</bean>

<cache:annotation-driven cache-manager="cacheManager" />

<bean id="cacheService" class="cn.pomit.springwork.ehcache.service.CacheService">
    <property name="userInfoService" ref="userInfoService" />
</bean>
```

ehcache.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">

  <!-- 磁盘缓存位置 -->
  <diskStore path="java.io.tmpdir/ehcache"/>

  <!-- 默认缓存 -->
  <defaultCache
          maxEntriesLocalHeap="10000"
          eternal="false"
          timeToIdleSeconds="120"
          timeToLiveSeconds="120"
          maxEntriesLocalDisk="10000000"
          diskExpiryThreadIntervalSeconds="120"
          memoryStoreEvictionPolicy="LRU"/>

  <!-- userCache缓存 -->
  <cache name="userCache"
         maxElementsInMemory="1000"
         eternal="false"
         timeToIdleSeconds="0"
         timeToLiveSeconds="1000"
         overflowToDisk="false"
         memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```

## 2 业务代码

CacheService.java

```
import org.springframework.cache.annotation.Cacheable;

import cn.pomit.springwork.mybatis.domain.UserInfo;
import cn.pomit.springwork.mybatis.service.UserInfoService;

public class CacheService {
	UserInfoService userInfoService;

	@Cacheable(value = "userCache", key = "#root.targetClass.simpleName+'-'+#root.methodName+'-'+#userName")
	public UserInfo getUserInfoByUserName(String userName) {
		return userInfoService.getUserInfoByUserName(userName);
	}

	public UserInfoService getUserInfoService() {
		return userInfoService;
	}

	public void setUserInfoService(UserInfoService userInfoService) {
		this.userInfoService = userInfoService;
	}
	
}
```

## 3 注意事项
CacheService在这里没有用@Service作注解，而是放在xml定义了一个bean，这是因为，@Service注解不能被动态代理，导致缓存无效，经过多次实验，放在xml里是可以被代理到，并使缓存生效。

这个原因是：

* Springmvc和Spring配置的扫描路径没有分离开来，重复扫描导致了Spring扫描过程使用了代理，但是Springmvc过程把这个bean从spring窗口中移去。

如果Springmvc使用context:include-filter配置了只扫描Controller和RestController注解就可以解决这个问题。

[《Spring和Spring Mvc 5整合详解》](https://www.pomit.cn/p/181499739820288)这篇已经修正了Springmvc的配置

## 4 测试

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import cn.pomit.springwork.ehcache.service.CacheService;
import cn.pomit.springwork.mybatis.domain.UserInfo;

@RestController
@RequestMapping("/ehcache")
public class EhcacheRest {

	@Autowired
	private CacheService cacheService;

	@RequestMapping(value = "/test/{name}", method = { RequestMethod.GET })
	public UserInfo test(@PathVariable("name") String name) {
		return cacheService.getUserInfoByUserName(name);
	}

}
```