# JSF参数传递

参考：[JSF参数传递方式](https://blog.csdn.net/iteye_8029/article/details/82490167)

---

## 1 f:param标签

### 1.1 前后端的参数传递

#### 1.1.1 请求页面

```
<h:form>
    <h:commandLink value="Test2" action="#{paramBean.test}">
        <f:param name="name" value="zhang"></f:param>
        <f:param name="id" value="123456"></f:param>
    </h:commandLink>
</h:form>
```

>注意:这里只能使用h:commandLink标签，而不能使用h:commandButton标签!

#### 1.1.2 后台取参数

##### 1 通过Request对象取值

```
HttpServletRequest request = (HttpServletRequest)FacesContext.getCurrentInstance().getExternalContext().getRequest();
request.getParameter("name");
```

##### 2 通过RequestParameterMap取值

```
Map varMap = FacesContext.getCurrentInstance().getExternalContext().getRequestParameterMap();
varMap.get("id");
```

##### 3 通过配置文件进行Bean的属性值注入，在Bean的方法中直接使用属性

```
<managed-bean>
    <managed-bean-name>paramBean</managed-bean-name>
    <managed-bean-class>com.spg.bean.ParamBean</managed-bean-class>
    <managed-bean-scope>request</managed-bean-scope>
    <managed-property>
        <property-name>id</property-name>
        <property-class>java.lang.String</property-class>
        <value>#{param.id} </value>
    </managed-property>
</managed-bean>
```

### 1.2 页面到页面的参数传递

#### 1.2.1 请求页面

* 第1种

```
<h:outputLink value="param2.jsf">
    <h:outputText value="Test4"></h:outputText>
    <f:param name="name" value="chen"></f:param>
    <f:param name="id" value="123456"></f:param>
</h:outputLink>
```

* 第2种

```
<h:outputLink value="param2.jsf?name=chen&id=123456">
    <h:outputText value="Test4"></h:outputText>
</h:outputLink>
```

>注意：以上两种方法，不能同时使用!

#### 1.2.2 页面中取参数

##### 1 使用JSF的值表达式

```
<h:outputText value="#{param.name}"></h:outputText>
<h:outputText value="#{param.id}"></h:outputText>
```

##### 2 使用JSP的表达式

```
<%=request.getParameter("name")%>
<%=request.getParameter("id")%>
```

## 2 Backing Bean 与 h:inputHidden标签

### Backing Bean

```
import javax.faces.component.UIInput;
import javax.faces.component.UIOutput;

public class BackingBean {

    private UIOutput idComponent;

    public UIOutput getIdComponent() {
        return idComponent;
    }

    public void setIdComponent(UIOutput idComponent) {
        this.idComponent = idComponent;
    }
}
```

### 2.1 前后端的参数传递

#### 2.1.1 请求页面

```
<h:form>
    <h:inputHidden value="123456" binding="#{backingBean.idComponent}"></h:inputHidden>
    <h:commandButton value="登录" action="#{paramBean.login}"></h:commandButton>
</h:form>
```

#### 2.1.2 后台取参数

```
FacesContext context = FacesContext.getCurrentInstance();
BackingBean backBean =(BackingBean)context.getApplication().getVariableResolver().resolveVariable(context,"backingBean");//该方法已经过时
BackingBean bean =(BackingBean)context.getApplication().getELResolver().getValue(context.getELContext(), null, "backingBean");
backBean.getIdComponent().getValue();
bean.getIdComponent().getValue();

FacesContext context = FacesContext.getCurrentInstance();
```

### 2.2 页面到页面的参数传递

#### 2.2.1 请求页面

```
<h:form>
    <h:inputHidden value="123456" binding="#{backingBean.idComponent}"></h:inputHidden>
    <h:commandButton value="Test5" action="param"></h:commandButton>
    <h:commandLink value="Test6" action="param"></h:commandLink>
</h:form>
```

>注意：h:outputLink 标签不能使用该方式传递参数！

#### 2.2.2 页面中取参数

```
<h:outputText value="#{backingBean.idComponent.value}"></h:outputText>
```

## 3 通过session(application)对象传递

### 3.1 前后端的参数传递

#### 3.1.1 请求页面

```
<h:form>
    <%session.setAttribute("name","hujilie"); %>
    <%application.setAttribute("id","123456"); %>
    <h:commandButton value="Test8" action="#{paramBean.test2}"></h:commandButton>
    <h:commandLink value="Test8" action="#{paramBean.test2}"></h:commandLink>
</h:form>
```

#### 3.1.2 后台取参数

```
FacesContext context = FacesContext.getCurrentInstance();
Map sessionMap =context.getExternalContext().getSessionMap();
Map applicationMap = context.getExternalContext().getApplicationMap();
HttpSession session =(HttpSession) context.getExternalContext().getSession(true);
ServletContext application = (ServletContext)context.getExternalContext().getContext();
sessionMap.get("name");
applicationMap.get("id");
session.getAttribute("name");
application.getAttribute("id");
```

### 3.2 页面到页面的参数传递

#### 3.2.1 请求页面

```
<h:form>
    <%session.setAttribute("name","hujilie"); %>
    <%application.setAttribute("id","123456"); %>
    <h:outputLink value="param2.jsf">Test10</h:outputLink>
</h:form>
```

#### 3.2.2 页面中取参数

```
<h:outputText value="#{sessionScope.name}"></h:outputText><br>
<h:outputText value="#{applicationScope.id}"></h:outputText>
```

## 4 f:attribute标签传递

### 4.1 前后端的参数传递

#### 4.1.1 请求页面

```
<h:form>
    <h:commandButton action="#{paramBean.test3}" value="Test11" actionListener="#{paramBean.changeName}">
        <f:attribute name="name" value="hujilie"/>
    </h:commandButton>
    <h:commandLink action="#{paramBean.test3}" value="Test12" actionListener="#{paramBean.changeName}">
        <f:attribute name="name" value="hujilie"/>
    </h:commandLink>
</h:form>
```

#### 4.1.2 后台取参数

```
public void changeName(ActionEvent e) {
    UIComponent component = e.getComponent();
    Map<String, Object> map = component.getAttributes();
    setName((String)map.get("name"));
}
```

## 5 f:setPropertyActionListener 标签传递

### 5.1 前后端的参数传递

#### 5.1.1 请求页面

```
<h:form>
    <h:commandButton action="#{paramBean.test3}" value="Test14">
        <f:setPropertyActionListener value="hujilie" target="#{paramBean.name}"/>
    </h:commandButton>
    <h:commandLink action="#{paramBean.test3}" value="Test15">
        <f:setPropertyActionListener value="hujilie" target="#{paramBean.name}"/>
    </h:commandLink>
</h:form>
```

#### 5.1.2 后台取参数

直接使用属性的值。