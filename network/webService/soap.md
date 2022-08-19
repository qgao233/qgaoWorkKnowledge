# SOAP与WSDL

## 1 什么是SOAP?

[WebService的两种方式SOAP和REST，之间的区别与优缺点](https://www.cnblogs.com/ypppt/p/13696463.html)

`SOAP` (Simple Object Access Protocol) 顾名思义，是一个严格定义的信息交换协议，用于在`Web Service`中把远程调用和返回封装成机器可读的格式化数据。

事实上SOAP数据使用XML数据格式，定义了一整套复杂的标签，以描述调用的远程过程、参数、返回值和出错信息等等。

然而，

* 随着需要的增长，又不得增加协议以支持安全性，这使SOAP变得异常庞大，背离了简单的初衷。
* 另一方面，各个服务器都可以基于这个协议推出自己的API，即使它们提供的服务及其相似，但定义的API不尽相同，

这两点导致了WSDL的诞生。

`WSDL` (Web Service Description Language) 也遵循XML格式，用来描述哪个服务器提供什么服务，怎样找到它，以及该服务使用怎样的接口规范，简言之，即**服务发现**。

现在，使用Web Service的过程变成：

1. 获得该服务的WSDL描述，
2. 根据WSDL构造一条格式化的SOAP请求发送给服务器，
3. 然后接收一条同样SOAP格式的应答，
4. 最后根据先前的WSDL解码数据。

绝大多数情况下，请求和应答使用HTTP协议传输，那么发送请求就使用HTTP的POST方法。

## 2 什么是WSDL?

[WSDL详解](https://blog.csdn.net/yhahaha_/article/details/93716263)

### 2.1 遇见WSDL

有人在WebService开发的时候，特别是和第三方有接口的时候，走的是SOAP协议，然后用户（或后台）给你一个WSDL文件（或网址），说按照上面的进行适配。

### 2.2 WSDL简介

WSDL (Web Services Description Language,Web服务描述语言)是遵循WSDL-XML模式的XML文档，他将Web服务描述定义为一组**服务访问点**，客户端可以通过这些服务访问点对包含**面向文档信息**或**面向过程调用**的服务进行**访问**(类似远程过程调用)。

WSDL记录信息：

1. 抽象描述**访问的操作**和访问时使用的**请求/响应消息**
2. 然后将其**绑定**到具体的**传输协议和消息格式**上以最终**定义**具体部署的**服务访问点**。
3. **组合**相关的具体部署的**服务访问点**成为抽象的**Web服务**。

总结，WSDL 文档将Web服务定义为服务访问点或端口的集合。

### 2.3 WSDL的基本概念

在 WSDL 中，由于服务访问点或消息的抽象定义，已从具体的服务部署或数据格式绑定中分离出来，因此可以对抽象定义进行再次使用。

* 消息：指对交换数据的抽象描述；
* 端口类型：指操作的抽象集合。用于特定端口类型的具体协议和数据格式规范构成了可以再次使用的绑定。
* 端口：将Web访问地址与可再次使用的绑定相关联。
* 服务：端口的集合。

### 2.4 WSDL文档元素

一个WSDL文档通常包含8个重要的元素，即：

* definitions：WSDL文档的根元素，其他元素被包含在definitions元素中。
* **types**：消息类型，数据类型定义的容器，它使用某种类型系统（如 XSD）。
* import。
* **message**：消息，通信数据的抽象类型化定义，它由一个或者多个 part 组成。
    * **part**：消息参数。
* **portType**：端口类型，特定端口类型的具体协议和数据格式规范。它由一个或者多个 Operation组成。 
    * **operation**：操作：对服务所支持的操作进行抽象描述，WSDL定义了四种操作： 
        >1. 单向(one-way):端点接受信息;
        >2. 请求-响应(request-response):端点接受消息,然后发送相关消息;
        >3. 要求-响应(solicit-response):端点发送消息,然后接受相关消息;
        >4. 通知(notification):端点发送消息。
* **binding**：特定端口类型的具体协议和数据格式规范。
* **service**：相关端口的集合，包括其关联的接口、操作、消息等。 
    * **port**：定义为绑定和网络地址组合的单个端点。

### 2.5 WSDL文档详细解构

下面通过一份wsdl文档，来详细解读WSDL结构：

```
 <?xml version="1.0" encoding="UTF-8" ?>
<wsdl:definitions
    targetNamespace="http://com.liuxiang.xfireDemo/HelloService"
    xmlns:tns="http://com.liuxiang.xfireDemo/HelloService"
    xmlns:wsdlsoap="http://schemas.xmlsoap.org/wsdl/soap/"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soapenc11="http://schemas.xmlsoap.org/soap/encoding/"
    xmlns:soapenc12="http://www.w3.org/2003/05/soap-encoding"
    xmlns:soap11="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
    <wsdl:types>
        <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
            attributeFormDefault="qualified" elementFormDefault="qualified"
            targetNamespace="http://com.liuxiang.xfireDemo/HelloService">
            <xsd:element name="sayHello">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element maxOccurs="1" minOccurs="1"
                            name="name" nillable="true" type="xsd:string" />
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
            <xsd:element name="sayHelloResponse">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element maxOccurs="1" minOccurs="0"
                            name="return" nillable="true" type="xsd:string" />
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
        </xsd:schema>
    </wsdl:types>
    <wsdl:message name="sayHelloResponse">
        <wsdl:part name="parameters" element="tns:sayHelloResponse" />
    </wsdl:message>
    <wsdl:message name="sayHelloRequest">
        <wsdl:part name="parameters" element="tns:sayHello" />
    </wsdl:message>
    <wsdl:portType name="HelloServicePortType">
        <wsdl:operation name="sayHello">
            <wsdl:input name="sayHelloRequest"
                message="tns:sayHelloRequest" />
            <wsdl:output name="sayHelloResponse"
                message="tns:sayHelloResponse" />
        </wsdl:operation>
    </wsdl:portType>
    <wsdl:binding name="HelloServiceHttpBinding"
        type="tns:HelloServicePortType">
        <wsdlsoap:binding style="document"
            transport="http://schemas.xmlsoap.org/soap/http" />
        <wsdl:operation name="sayHello">
            <wsdlsoap:operation soapAction="" />
            <wsdl:input name="sayHelloRequest">
                <wsdlsoap:body use="literal" />
            </wsdl:input>
            <wsdl:output name="sayHelloResponse">
                <wsdlsoap:body use="literal" />
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>
    <wsdl:service name="HelloService">
        <wsdl:port name="HelloServiceHttpPort"
            binding="tns:HelloServiceHttpBinding">
            <wsdlsoap:address
                location="http://localhost:8080/xfire/services/HelloService" />
        </wsdl:port>
    </wsdl:service>
</wsdl:definitions>
```

#### 2.5.1 definitions

* 该元素封装了整个文档。
* 通过其name提供了一个WSDL文档。
* 通过targetNamespace提供了的命名空间。

没有其他作用了。

#### 2.5.2 types

WSDL采用了W3C XML模式内置类型作为其基本类型系统。types元素用作一个容器，用于定义XML模式内置类型中没有描述的各种数据类型（估计意思是在本文档中新定义的元素不属于内置类型当中的）。

当声明消息部分的有效时，消息定义使用了在types元素中定义的数据类型和元素。

```
<wsdl:types>
    <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        attributeFormDefault="qualified" elementFormDefault="qualified"
        targetNamespace="http://com.liuxiang.xfireDemo/HelloService">
        <xsd:element name="sayHello">
            <xsd:complexType>
                <xsd:sequence>
                    <xsd:element maxOccurs="1" minOccurs="1"
                        name="nickname" nillable="true" type="xsd:string" />
                </xsd:sequence>
            </xsd:complexType>
        </xsd:element>
        <xsd:element name="sayHelloResponse">
            <xsd:complexType>
                <xsd:sequence>
                        <xsd:element maxOccurs="1" minOccurs="0"
                        name="return" nillable="true" type="xsd:string" />
                </xsd:sequence>
            </xsd:complexType>
        </xsd:element>
    </xsd:schema>
</wsdl:types>
```

上面是数据定义部分，该部分定义了两个元素：

* sayHello：定义了一个复杂类型，仅仅包含一个简单的字符串，将来用来描述操作的参入传入部分； 
* sayHelloResponse：定义了一个复杂类型，仅仅包含一个简单的字符串，将来用来描述操作的返回值； 

这里sayHelloResponse是和sayHello相关的，

* sayHello相当于一个方法，里面的： `name="nickname" type="xsd:string"`，是确定传入name的参数是String类型的，即`sayHello(String nickname)`，
* 而sayHelloResponse中的 `name="return" type="xsd:string"` 是确定方法`sayHello(String nickname)`返回的类型是String类型的，即`String sayHello(String nickname)`

#### 2.5.3 import

 import元素使得可以在当前的WSDL文档中使用其他WSDL文档中指定的命名空间中的定义元素。
 
 本例子中没有使用import元素。通常在用户希望模块化WSDL文档的时候，该功能是非常有效果的。 

 ```
 <wsdl:import namespace="http://xxx.xxx.xxx/xxx/xxx" location="http://xxx.xxx.xxx/xxx/xxx.wsdl"/>
 ```

必须有namespace属性和location属性： 

1. namespace属性：值必须与正导入的WSDL文档中声明的targetNamespace相匹配； 
2. location属性：必须指向一个实际的WSDL文档，并且该文档不能为空。

#### 2.5.4 message

message元素描述了：

* Web服务使用消息的有效负载。
* 输出或者接受消息的有效负载。
* SOAP文件头和错误detail元素的内容。

定义message元素的方式取决于**使用RPC样式**还是**文档样式**的消息传递。

在本文中的message元素的定义使用了采用文档样式的消息传递：

```
<wsdl:message name="sayHelloRequest">
    <wsdl:part name="parameters" element="tns:sayHello" />
</wsdl:message>
<wsdl:message name="sayHelloResponse">
    <wsdl:part name="parameters" element="tns:sayHelloResponse" />
</wsdl:message>
```

该部分是消息格式的抽象定义：定义了两个消息sayHelloResponse和sayHelloRequest：

**Ⅰ· sayHelloRequest：**

sayHello操作的请求消息格式，由一个消息片段组成，名字为parameters,元素是我们前面定义的types中的元素；

**Ⅱ· sayHelloResponse：**

sayHello操作的响应消息格式，由一个消息片段组成，名字为parameters,元素是我们前面定义的types中的元素；

如果采用RPC样式的消息传递，只需要将文档中的element元素修改为type即可。

#### 2.5.5 portType

portType元素定义了Web服务的抽象接口。

该接口有点类似Java的接口，都是定义了一个抽象类型和方法，没有定义实现。

在WSDL中，portType元素是由binding和service元素来实现的，这两个元素用来说明Web服务实现使用的Internet协议、编码方案以及Internet地址。 

一个portType中可以定义多个operation，一个operation可以看作是一个**方法**，本文中WSDL文档的定义：

```
<wsdl:portType name="HelloServicePortType">
    <wsdl:operation name="sayHello">
        <wsdl:input name="sayHelloRequest"
            message="tns:sayHelloRequest" />
        <wsdl:output name="sayHelloResponse"
            message="tns:sayHelloResponse" />
    </wsdl:operation>
</wsdl:portType>
```

portType定义了服务的调用模式的类型，这里包含一个操作sayHello方法，同时包含input和output表明该操作是一个请求／响应模式，

* 请求消息是前面定义的sayHelloRequest，
* 响应消息是前面定义的sayHelloResponse。
* input表示传递到Web服务的有效负载，
* output消息表示传递给客户的有效负载。 

这里相当于抽象类中定义了一个抽象方法sayHello，而方法参数的定义和返回值的定义都是在types中设置的，方法名又是在message中定义有的。

#### 2.5.6 binding

binding元素将一个抽象portType映射到一组具体协议(SOAP和HTTP)、消息传递样式、编码样式。

通常binding元素与协议专有的元素和在一起使用，本文中的例子：

```
<wsdl:binding name="HelloServiceHttpBinding" type="tns:HelloServicePortType">
    <wsdlsoap:binding style="document"
        transport="http://schemas.xmlsoap.org/soap/http" />
    <wsdl:operation name="sayHello">
        <wsdlsoap:operation soapAction="" />
        <wsdl:input name="sayHelloRequest">
            <wsdlsoap:body use="literal" />
        </wsdl:input>
        <wsdl:output name="sayHelloResponse">
            <wsdlsoap:body use="literal" />
        </wsdl:output>
    </wsdl:operation>
</wsdl:binding>
```

这部分将服务访问点的抽象定义与SOAP、HTTP绑定，描述如何通过SOAP/HTTP来访问按照前面描述的访问入口点类型部署的访问入口。 

其中规定了在具体SOAP调用时，应当使用的soapAction是”xxx”，这个Action在WebService代码调用中是很重要的。具体的使用需要参考特定协议定义的元素。

#### 2.5.7 service和port

service元素包含一个或者多个port元素，其中每个port元素表示一个不同的Web服务。

port元素将URL赋给一个特定的binding，甚至可以使两个或者多个port元素将不同的URL赋值给相同的binding。

文档中的例子：

```
<wsdl:service name="HelloService">
    <wsdl:port name="HelloServiceHttpPort"
        binding="tns:HelloServiceHttpBinding">
        <wsdlsoap:address
            location="http://localhost:8080/xfire/services/HelloService" />
    </wsdl:port>
</wsdl:service>
```

>原作者原话：对于这个WSDL文档的学习，第一次看是感觉非常陌生的，而且里面元素又多，学习的话先是要了解外层结构代表的意义和作用，然后理解里面的元素的意义和作用，有些元素作用不大，有些元素又是很关联的，有些元素是比较重要的。 