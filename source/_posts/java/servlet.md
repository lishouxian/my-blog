---
title: 'JavaWeb之Servlet'
date: 2020-08-22 23:00:00
tags: [javaweb]
categories: javaweb
published: true
hideInList: true
feature: 
isTop: false
---


## Servle技术

###  什么是Servlet

1. Servlet是javaee规范之一。规范就是接口
2. Servlet就是javaweb三大组件之一。三大组件分别是：servlet程序，filter过滤器，listener监听器。
3. Servlet是运行在服务器上的一个java小程序，他可以接受客户端发送过来的请求，并相应数据给客户端。

<!-- more -->

###  手动实现Servlet程序

1. 编写一个类来实现Servlet 接口

```java
public class HelloServlet implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
    }
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("hello");
    }
    @Override
    public String getServletInfo() {
        return null;
    }
    @Override
    public void destroy() {
    }
}
```

2. 配置Servlet到`web.xml`

```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <servlet>
<!--servlet-name标签 给servlet程序起别名-->
    <servlet-name>hello</servlet-name>
    <servlet-class>com.lishx.demo01.HelloServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>hello</servlet-name>
<!--配置访问地址-->
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

3. 启动Tomcat

**常见的错误1**：配置访问地址出错，未加`/`

`java.lang.IllegalArgumentException: servlet映射中的<url pattern>[hello]无效`

**常见的错误2**：`servlet-name`配置出错

`java.lang.IllegalArgumentException: Servlet映射指定未知的Servlet名称[hello1]`

**常见的错误3**：`servlet-class`配置出错

`java.lang.IllegalArgumentException: servlet映射中的<url pattern>[hello]无效`

###  url地址到Servlet程序的访问路径



TODO

###  Servlet的生命周期

1. 执行Servlet构造器方法
2. 执行init初始化方法
3. 执行service方法
4. 执行destroy销毁方法

**service方法每次访问都会调用，其他方法只在创建和销毁时才会调用**

###  GET和POST请求的分发处理

```java
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
	//强转类型
    HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
    //使用getMethod方法获取方法名
    String method = httpServletRequest.getMethod();
    if("GET".equals(method)){
        doGET();
    }else if("POST".equals(method)){
        doPOST();
    }
    System.out.println("hello");
}
//方法分别打包执行
private void doPOST() {
    System.out.println("doPOST");
}

private void doGET() {
    System.out.println("doGET");
}
```

###  通过继承HttpServlet实现Servlet程序

一般在项目开发中，都是使用`HttpServlet`类的方式去实现Servlet程序。

1. 编写一个类去继承`HttpServlet`
2. 根据业务需要重写doGet 或者doPost 方法

```java 
public class HelloServlet2 extends HttpServlet {
    // 在GET请求的时候调
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

     // 在POST请求的时候调
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

3. 到web.xml中配置Servlet程序的访问地址

```xml
<servlet>
  <servlet-name>hello2</servlet-name>
  <servlet-class>com.lishx.demo01.HelloServlet2</servlet-class>
</servlet>

<servlet-mapping>
  <servlet-name>hello2</servlet-name>
  <url-pattern>/hello2</url-pattern>
</servlet-mapping>
```

### 使用IDEA创建Servlet程序

![image-20200822152405790](https://blog-1251782526.cos.ap-shanghai.myqcloud.com/uPic/007S8ZIlly1ghzyida7vxj30u00wljww.jpg)



## ServletConfig类

ServletConfig类从类名上来看是Servlet的配置信息类

Servlet程序和ServletConfig对象都是有tomcat负责创建，我们负责使用。

Servlet程序默认是第一次访问的时候创建，ServletConfig是每个Servlet程序创建时，就创建一个对应的ServletConfig对象。



### ServletConfig类的三大作用

- 可以获取Servlet程序的别名 servlet-name的值
- 获取初始化参数init-param
- 获取ServletContext对象

获取方式

```java
@Override
public void init(ServletConfig servletConfig) throws ServletException{
    System.out.println("servlet-name = " + servletConfig.getInitParameter("username"));
	System.out.println("初始化参数 = " + servletConfig.getInitParameter("username"));
    System.out.println("获取ServletContext对象 = " + servletConfig.getServletContext());
}
```

配置内容

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.lishx.demo01.HelloServlet</servlet-class>
    <!--初始化参数-->
    <init-param>
        <param-name>username</param-name>
        <param-value>root</param-value>
    </init-param>
</servlet>
```

获取`ServletConfig`的其他方式

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    super.doGet(req, resp);
    ServletConfig servletConfig = getServletConfig();
}
```

**请注意 如果重写 init 方法的话， 必须要 `super.init(config)`调用父类的方法**

## ServletContext类

### 什么是ServletContext？

- servletContext是一个接口，他表示Servlet上下文对象
- 一个web工程，只有一个ServletContext对象实例
- ServletContext是一个域对象
- ServletContext是在web工程部署启动的时候创建，在web工程停止的时候销毁

什么是域对象？

域对象是可以向Map一样存取数据的对象

这里的域指的是存取数据的操作范围：整个web工程

|        |     存数据     |     取数据     |     删除数据      |
| :----: | :------------: | :------------: | :---------------: |
|  Map   |     put()      |     get()      |     remove()      |
| 域对象 | putAttribute() | getAttribute() | removeAttribute() |

### ServletContext的四个常见作用

- 获取web.xml中配置的上下文参数
- 获取当前的工程路径
- 获取工程部署后在服务器硬盘上的绝对路径
- 像map一样存取数据

```java 
//使用注解的方式来配置
@WebServlet(name = "ServletContext")
public class ServletContext extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//- 获取web.xml中配置的上下文参数
        javax.servlet.ServletContext servletContext = getServletConfig().getServletContext();
//        javax.servlet.ServletContext servletContext = getServletContext();
        String name = servletContext.getInitParameter("name");
        System.out.println(name);
//- 获取当前的工程路径
        String contextPath = servletContext.getContextPath();
//- 获取工程部署后在服务器硬盘上的绝对路径
        String realPath = servletContext.getRealPath("/");
    }
```











































































































































