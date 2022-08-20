# API框架-Swagger

* 号称世界上最流行的API框架
* RestFul API文档在线生成工具--->>>API文档与API同步更新
* 可以直接运行,可以在线测试API接口
* 支持多种语言:(Java,PHP.....)

## 1 诞生原由

在web开发的场景下，分为两个大时代：

### 1.1 以前的时代

SSM + JSP模板引擎====>后端程序员

也就是整套程序从前台到后台的代码全是后端程序员开发。

### 1.2 现在的时代

可称为**前后端分离时代**。

>当然这套实现早在16、17年时就已经开始上线了。而且在22年的今天，主要实现技术为：SpringBoot + VUE..........等等

主要概念为：

* 后端:后端控制层 + 服务层 + 数据访问层
    >后端使用postman、curl等工具进行测试。
* 前端:前端控制层 + 视图层
    >前端可以单独测试，即伪造后端交互数据，不需要后端传入json数据了，前端工程已经可以运行。

**前后端交互：**

* 通过相关的API接口进行交互
* 前后端相对独立,松耦合
* 前后端可以分别部署在不同的服务器上

**问题：**

因为前后端是分别开发的，那么当前后端集成**联调**时，前端和后端开发人员**无法**做到**及时协商，尽早解决问题**，就会导致项目延期。

说白了就是后端没有及时提供更新后的API给前端程序。

## 2 使用Swagger

使用SpringBoot集成Swagger需要两个依赖：

* springfox-swagger2
* springfox-swagger-ui

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

配置swagger：

```
@Configuration // 标识配置类
@EnableSwagger2 // 开启Swagger
public class SwaggerConfig {
​
    /**
    * 配置Swagger信息
    */
    private ApiInfo apiInfo() {
        // 配置作者信息
        Contact contact = new Contact("qgao",
                "https://www.qgaoIdea.com",
                "qgao233@163.com");
        // 配置API文档标题
        return new ApiInfo("框架师Api",
                // API文档描述
                "Api Documentation",
                // API版本号
                "1.0",
                // 配置URL(公司官网/blog地址)
                "https://www.qgaoIdea.com",
                // 作者信息
                contact,
                // 以下内容默认即可
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }
​
    /**
   * 配置了Swagger的Docket的Bean实例
   */
    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                // 配置扫描接口
                .select()
                // 配置Swagger是否启动,默认:true
                .enable(false)
                /*
                *RequestHandlerSelectors,配置要扫描接口的方式
                * 参数说明:
                * basePackage:基于包扫描
                * class:基于类扫描
                * any():扫描全部
                * none():全部都不扫描
                * withMethodAnnotation:通过方法的注解扫描
                * // withMethodAnnotation(GetMapping.class))
                * withClassAnnotation:通过类的注解扫描
                */
                .apis(RequestHandlerSelectors.basePackage("com.mobai.swagger.controller"))
                // .paths()过滤,不扫描哪些接口
                .paths(PathSelectors.any())
                .build();
    }
    
}
```

测试运行
唯一地址：http://localhost:8080/swagger-ui.html

### 2.1 常用注解

* `@ApiModel("注释")`：声明在**实体类**上，给类添加注释 
* `@ApiModelProperty("注释")`：声明在**成员变量**上，给成员变量添加注释
* `@Api("")`：声明在**接口类**上，给类添加注释
* `@ApiOperation("注释")`：声明在**方法**上，给接口(一般是Controller)的方法添加注释
* `@ApiParam("")`：声明在方法的**入参**上，给方法的参数添加注释

## 3 使用案例

### 3.1 案例1：开发启用，生产关闭

Ⅰ、添加：

* 默认配置：application.properties
    >spring.profiles.active=dev
* 测试环境：application-dev.properties
    >server.port=8081
* 生产环境：application-prod.properties
    >server.port=8082

Ⅱ、SwaggerConfig配置类判断当前环境：

```
@Bean
public Docket docket(Environment environment) {
    // 设置要显示的Swagger环境
    Profiles profiles = Profiles.of("dev", "prod");
    // 通过environment.acceptsProfiles();判断自己是否在自己设定换的环境当中
    boolean flag = environment.acceptsProfiles(profiles);
​
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            // 监听自己设置的环境
            .enable(flag)
            // 配置扫描接口
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.qgao.swagger.controller"))
            .paths(PathSelectors.any())
            .build();
}
```

### 3.2 案例2：配置API文档的分组

Ⅰ、如何配置组：

```
在后面加上
// 配置分组
.groupName("qgao")
```

Ⅱ、配置多个组：

```
@Bean
public Docket docket1() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("A");
}
​
@Bean
public Docket docket2() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("B");
}
​
@Bean
public Docket docket3() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("C");
}
```

