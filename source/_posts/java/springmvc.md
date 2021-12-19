---
title: 'SpringMVC'
date: 2020-09-02 19:00:00
tags: [javaweb框架]
categories: javaweb
published: true
hideInList: true
feature: 
isTop: false
---

##  SpringMVC的基本概念

### 关于三层架构和MVC

#### 三层架构

- 开发服务器端的程序，一般都是基于两种形式，分别叫做B/S即浏览器服务器，和C/S即客户端服务器。
- 使用Java语言基本上都是开发B/S架构的程序，B/S架构又分成了三层架构
- 三层架构：
  - 表现层：WEB层，用来和客户端进行交互的，表现层一般会使用MVC的设计模型
  - 业务层：处理公司具体的业务逻辑
  - 持久层：用来操作数据库
  

<!-- more -->

#### MVC模型

- MVC全名是Model View Controller模型视图控制器，每个部分各司其职。
- Model：数据模型，Java Bean的类，用来进行数据封装
- View： 指JSP、HTML等用来展示数据给用户
- Controller： 用来接受用户的请求，整个流程的控制器。用来进行数据校验等。

![三层架构](https://tva1.sinaimg.cn/large/007S8ZIlly1gic400ei2uj31dp0u0dku.jpg)

## SpringMVC的入门案例

### SpringMVC的概述

- SpringMVC的概述：
  - 是一种基于Java程序实现的MVC设计模型的请求驱动类型的轻量级WEB框架
  - 属于SpringFrameworks的后续产品，现在已经融合在Spring Web Flow里面。提供了构建Web应用程序的全功能MVC模块
  - 使用Sprint可插入MVC架构从而使使用Spring进行WEB开发时，可以选择使用Spring的SpringMVC框架或者集成其他MVC框架
- SpringMVC框架在三层架构中的位置：表现层框架
- SpringMVC框架的优势

### SpringMVC的入门程序

#### 创建WEB工程，引入JAR包

Pom.xml具体的坐标如下图所示：

```xml
  
<!-- 版本锁定 --> <properties>
    <spring.version>5.0.2.RELEASE</spring.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### 配置核心的控制器(配置DispatcherServlet)

 在web.xml配置文件中核心控制器DispatcherServlet

```xml
<!-- SpringMVC的核心控制器 --> <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-
class>
<!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 --> <init-param>
    <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
<!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
  
```

编写springmvc.xml的配置文件

```xml
 
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
<!-- 配置spring创建容器时要扫描的包 -->
<context:component-scan base-package="com.itheima"></context:component-scan>
<!-- 配置视图解析器 -->
    <bean id="viewResolver"
class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
<!-- 配置spring开启注解mvc的支持
    <mvc:annotation-driven></mvc:annotation-driven>-->
</beans>
```

编写index.jsp

```jsp
 
<body>
<h3>入门案例</h3>
<a href="${ pageContext.request.contextPath }/hello">入门案例</a>
</body>
```

HelloController控制器类

```java
 package cn.itcast.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
/**
* 控制器
* @author rt */
@Controller
public class HelloController {
/**
* 接收请求 * @return */
    @RequestMapping(path="/hello")
    public String sayHello() {
        System.out.println("Hello SpringMVC!!");
        return "success";
    }
}
```

在WEB-INF目录下创建pages文件夹，编写success.jsp的成功页面

```jsp
<body>
     <h3>入门成功!!</h3>
</body>
```

#### 入门案例的执行过程分析

1. 入门案例的执行流程

   1. 当启动tomcat服务器的时候因为配置了load-on-start标签，所以会创建dispatchServlet对象，就会加载Springmvc.xml配置文件
   2. 开启了注解扫描，那么hello controller就会被创建
   3. 从index.jsp发送请求，请求会首先到达DispatchServlet核心控制器，再根据配置@PequestMapping注解找到执行的具体方法
   4. 根据执行方法的返回值，再根据配置试图解析器，去指定的目录下寻找jsp文件
   5. Tomcat服务器响应

   ![MVC执行过程](https://tva1.sinaimg.cn/large/007S8ZIlly1gic4p6e2yhj32480u04mh.jpg)

2. 入门案例中的组件分析
   1. 前端控制器(DispatcherServlet) 
   2. 处理器映射器(HandlerMapping) 
   3. 处理器(Handler)
   4. 处理器适配器(HandlAdapter)
   5. 视图解析器(View Resolver)
   6. 视图(View)

3. RequestMapping注解

   1. RequestMapping注解的作用是建立请求URL和处理方法之间的对应关系
   2. RequestMapping注解可以作用在方法和类上
      1. 作用在类上:第一级的访问目录
      2. 作用在方法上:第二级的访问目录
      3. 细节:路径可以不编写 / 表示应用的根目录开始
      4. 细节:${ pageContext.request.contextPath }也可以省略不写，但是路径上不能写 /
   3. RequestMapping的属性
      1. path ： 默认
      2. value
      3. mthod
      4. params
      5. headers 发送的请求中必须包含的请求头

## 请求参数的绑定

### 请求参数的绑定说明

- 绑定机制

  - 表单提交的数据都是k-v格式的 username=xiaomei&password=123
  - Spring MVC的参数绑定过程及欧式把表单提交的请求参数，作为控制器中的方法的参数来进行绑定
  - 要求是：提交表单的name和参数的名称是一样的

- 支持的数据类型

  - 基本数据类型和字符串类型
  - 实体类型：那么与属性名称一致，假如包含其他引用类型，则写成对象.属性的方式：address.name
  - 集合数据类型（List、Map集合等） JSP页面编写方式:list[0].属性
  - 基本数据类型和字符串数据类型区分大小写

- 解决中文乱码的问题

  - 在web.xml中配置Spring提供的过滤器类

```xml
<!-- 配置过滤器，解决中文乱码的问题 --> <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-
class>
<!-- 指定字符集 --> <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

- 自定义类型转换器
  - 表单中提交的任何数据类型全部都是字符串类型，但是 后台定义的Integer类型也能封装上，说明Spring框架内会进行默认的数据类型转换
  - 如果想要自定义数据类型转化，可以实现Converter的接口
    - 自定义类型转换器

```java
package cn.itcast.utils;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;
import org.springframework.core.convert.converter.Converter;
/**
* 把字符串转换成日期的转换器 * @author rt
*/
public class StringToDateConverter implements Converter<String, Date>{
/**
* 进行类型转换的方法 */
	public Date convert(String source) { // 判断
		if(source == null) {
		throw new RuntimeException("参数不能为空");
	}
    	try {
			DateFormat df = new SimpleDateFormat("yyyy-MM-dd"); // 解析字符串
			Date date = df.parse(source);
			return date;
		} catch (Exception e) {
			throw new RuntimeException("类型转换错误");
		}}
}
  
```

注册自定义类型转换器，在springmvc.xml配置文件中编写配置

```xml
 
<!-- 注册自定义类型转换器 -->
    <bean id="conversionService"
class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="cn.itcast.utils.StringToDateConverter"/>
            </set>
        </property>
    </bean>
<!-- 开启Spring对MVC注解的支持 -->
<mvc:annotation-driven conversion-service="conversionService"/>
```

在控制器中使用原生的ServletAPI对象：只需要在控制器的方法参数定义HttpServletRequest和HttpServletResponse对象


## 常用的注解

### @RequestParam注解

- 作用：把请求中的指定名称的参数传递给控制器中的形参赋值

- 属性：

  - value：请求参数中的名称
  - required：请求参数中是否必须提供此参数，默认值是true，必须提供
  
- 代码

  ```java
   /**
  * 接收请求 * @return */
  @RequestMapping(path="/hello")
  public String sayHello(@RequestParam(value="username",required=false)String name) {
      System.out.println("aaaa");
      System.out.println(name);
      return "success";
  }
  ```

### @RequestBody注解

- 作用:用于获取请求体的内容(注意:get方法不可以)
- 属性
  
- required:是否必须有请求体，默认值是true
  
- 代码

  ```java
   /**
  * 接收请求 * @return */
  @RequestMapping(path="/hello")
  public String sayHello(@RequestBody String body) {
      System.out.println("aaaa");
      System.out.println(body);
      return "success";
  }
  ```

### @PathVariable注解

- 作用:拥有绑定url中的占位符的。例如:url中有/delete/{id}，{id}就是占位符 
- 属性
  
- value:指定url中的占位符名称
  
- Restful风格的URL

  - 请求路径一样，可以根据不同的请求方式去执行后台的不同方法 
  - restful风格的URL优点
    - 结构清晰 
    - 符合标准
    - 易于理解
    - 扩展方便

- 代码如下

  ```java
   <a href="user/hello/1">入门案例</a>
  /**
  * 接收请求 * @return */
  @RequestMapping(path="/hello/{id}")
  public String sayHello(@PathVariable(value="id") String id) {
      System.out.println(id);
      return "success";
  }
  ```


### @RequestHeader注解

- 作用：获取请求头的值

- 属性

  - value：请求头的名称

- 代码

  ```java
   @RequestMapping(path="/hello")
      public String sayHello(@RequestHeader(value="Accept") String header) {
          System.out.println(header);
          return "success";
      }
  ```

### CookieValue注解

```java
 @RequestMapping(path="/hello")
public String sayHello(@CookieValue(value="JSESSIONID") String cookieValue) {
    System.out.println(cookieValue);
    return "success";
}
```

### ModelAttribute注解

```java
@ModelAttribute
public User showUser(String name) {
	System.out.println("showUser执行了..."); // 模拟从数据库中查询对象
	User user = new User(); user.setName("哈哈");
    user.setPassword("123"); user.setMoney(100d);
    return user;
}
@RequestMapping(path="/updateUser")
public String updateUser(User user) {
    System.out.println(user);
    return "success";
}
```

```java
@ModelAttribute
public void showUser(String name,Map<String, User> map) {
System.out.println("showUser执行了..."); // 模拟从数据库中查询对象
User user = new User(); user.setName("哈哈"); user.setPassword("123"); user.setMoney(100d);
    map.put("abc", user);
}
@RequestMapping(path="/updateUser")
public String updateUser(@ModelAttribute(value="abc") User user) {
    System.out.println(user);
    return "success";
}
```

### SessionAttributes注解

```java
@Controller
@RequestMapping(path="/user")
@SessionAttributes(value= {"username","password","age"},types=
{String.class,Integer.class})
public class HelloController {
    /**
    * 向session中存入值 * @return
    */
    // 把数据存入到session域对象中
    @RequestMapping(path="/save")
    public String save(Model model) {
    	System.out.println("向session域中保存数据"); 
        model.addAttribute("username", "root"); 
        model.addAttribute("password", "123"); 
        model.addAttribute("age", 20);
        return "success";
    }
    @RequestMapping(path="/find")
    public String find(ModelMap modelMap) {
        String username = (String) modelMap.get("username");
        String password = (String) modelMap.get("password");
        Integer age = (Integer) modelMap.get("age");
        System.out.println(username + " : "+password +" : "+age);
        return "success";
    }
    
    @RequestMapping(path="/delete")
    public String delete(SessionStatus status) {
        status.setComplete();
        return "success";
    }
```

## Controller响应数据和结果试图

### 返回值分类

#### 返回字符串

Controller方法返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址

```java
@RequestMapping(value="/hello")
public String sayHello() {
    System.out.println("Hello SpringMVC!!"); // 跳转到XX页面
    return "success";
}
```

响应的数据：使用`model.addAttribute("user", user)`

```java
    @RequestMapping(value="/initUpdate")
    public String initUpdate(Model model) {
        // 模拟从数据库中查询的数据
        User user = new User(); user.setUsername("张三"); 
        user.setPassword("123"); user.setMoney(100d); 
        user.setBirthday(new Date());
        model.addAttribute("user", user);
        return "update";
	}
```

```jsp
<h3>修改用户</h3>
${ requestScope }
<form action="user/update" method="post">
    
	姓名:<input type="text" name="username" value="${ user.username }">
    <br> 密码:<input type="text" name="password" value="${ user.password }">
    <br> 金额:<input type="text" name="money" value="${ user.money }">
    <br> <input type="submit" value="提交">
</form>
  
```

#### 返回值是void

如果控制器的方法返回值编写成void，会自动跳转到方法名所在的JSP页面-@RequestMapping(value="/initUpdate") initUpdate的页面。

可以请求转发或者重定向到指定的页面

```java
@RequestMapping(value="/initAdd")
    public void initAdd(HttpServletRequest request,HttpServletResponse response) throws Exception {
    System.out.println("请求转发或者重定向");
    // 请求转发
    // request.getRequestDispatcher("/WEBINF/pages/add.jsp").forward(request,response);
    // 重定向
    // response.sendRedirect(request.getContextPath()+"/add2.jsp");
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=UTF-8");
	// 直接响应数据 response.getWriter().print("你好"); return;
}
```

返回值是ModelAndView对象 ModelAndView对象是Spring提供的一个对象，可以用来调整具体的JSP视图

```java
@RequestMapping(value="/findAll")
public ModelAndView findAll() throws Exception {
ModelAndView mv = new ModelAndView();
    // 跳转到list.jsp的页面 mv.setViewName("list");
// 模拟从数据库中查询所有的用户信息 List<User> users = new ArrayList<>(); User user1 = new User(); user1.setUsername("张三"); user1.setPassword("123");
	User user2 = new User();
    user2.setUsername("赵四");
    user2.setPassword("456");
  	users.add(user1);
    users.add(user2);
	// 添加对象 mv.addObject("users", users);
	return mv;
}
```

### SpringMVC框架提供的转发和重定向

#### forward请求转发

 controller方法返回String类型，想进行请求转发也可以编写成`return "forward:/user/findAll"`

```java
 
/**
* 使用forward关键字进行请求转发
* "forward:转发的JSP路径"，不走视图解析器了，所以需要编写完整的路径 * @return
* @throws Exception
*/
@RequestMapping("/delete")
public String delete() throws Exception {
	System.out.println("delete方法执行了...");
	// return "forward:/WEB-INF/pages/success.jsp";
    return "forward:/user/findAll";
}
```

#### redirect重定向

controller方法返回String类型，想进行重定向也可以编写成`return "redirect:/add.jsp"`

```java
 /**
* 重定向
* @return
* @throws Exception */
@RequestMapping("/count")
public String count() throws Exception {
	System.out.println("count方法执行了...");
    return "redirect:/add.jsp";
	// return "redirect:/user/findAll";
}
```

### ResponseBody响应json数据

DispatcherServlet会拦截到所有的资源，导致一个问题就是静态资源(img、css、js)也会被拦截到，从而

不能被使用。解决问题就是需要配置静态资源不进行拦截，在springmvc.xml配置文件添加如下配置

```xml
<!-- 设置静态资源不过滤 -->
<mvc:resources location="/css/" mapping="/css/**"/> <!-- 样式 --> 
<mvc:resources location="/images/" mapping="/images/**"/> <!-- 图片 --> 
<mvc:resources location="/js/" mapping="/js/**"/> <!-- javascript -->
```

json字符串和JavaBean对象互相转换的过程中，需要使用jackson的jar包

```xml
 	<dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.9.0</version>
    </dependency>
```

#### 使用@RequestBody获取请求体数据

```javascript
$(function(){
// 绑定点击事件 $("#btn").click(function(){
        $.ajax({
            url:"user/testJson",
            contentType:"application/json;charset=UTF-8",
            data:'{"addressName":"aa","addressNum":100}',
            dataType:"json",
            type:"post",
            success:function(data){
                alert(data);
                alert(data.addressName);
            }
}); });
});
```

```java
* 获取请求体的数据 * @param body
*/
@RequestMapping("/testJson")
public void testJson(@RequestBody String body) {
    System.out.println(body);
}
```

#### 使用@RequestBody注解把json的字符串转换成JavaBean的对象

```javascript
$(function(){
// 绑定点击事件 $("#btn").click(function(){
        $.ajax({
            url:"user/testJson",
            contentType:"application/json;charset=UTF-8",
            data:'{"addressName":"aa","addressNum":100}',
            dataType:"json",
            type:"post",
            success:function(data){
                alert(data);
                alert(data.addressName);
            }
}); });
});
```

```java
/**
* 获取请求体的数据 * @param body
*/
@RequestMapping("/testJson")
public void testJson(@RequestBody Address address) {
    System.out.println(address);
}
```

#### 使用@ResponseBody注解把JavaBean对象转换成json字符串

```javascript
$(function(){
// 绑定点击事件 $("#btn").click(function(){
            $.ajax({
                url:"user/testJson",
                contentType:"application/json;charset=UTF-8",
				data:'{"addressName":"哈哈","addressNum":100}',
                dataType:"json",
                type:"post",
                success:function(data){
                    alert(data);
                    alert(data.addressName);
            }
}); });
});
```

```java
@RequestMapping("/testJson")
public @ResponseBody Address testJson(@RequestBody Address address) {
System.out.println(address); address.setAddressName("上海"); return address;
}
  
```

## SpringMVC实现文件上传

### 文件上传的回顾

**导入文件上传的maven依赖**

```xml
 	<dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.4</version>
    </dependency>
```

**添加multipartResolver的Bean配置**

```xml
    <!-- 配置multipartResolver-->
    <bean id="multipartResolver"  class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 设置上传文件的最大尺寸为 50MB -->
    <property name="maxUploadSize" value="52428800"/>
    </bean>
```

#### 编写文件上传的JSP页面

```jsp
<h3>测试文件上传</h3>
<form action="fileupload" method="post" enctype="multipart/form-data">
    选择文件:<input type="file" name="file"/><br/>
    选择描述:<input type="text" name="desc"/><br/>
    <input type="submit" value="上传文件"/>
</form>
```

#### 编写文件上传的Controller控制器

```java
/**
* 文件上传
* @throws Exception */
@Controller
public class UploadController {
    
    @RequestMapping("fileupload")
    public String testupload(String desc,MultipartFile file
    ) throws IOException {
        System.out.println(desc);
        System.out.println(file.getOriginalFilename());
        System.out.println(file.getInputStream());
        return "success";
    }
}
```

#### 多文件上传`MultipartFile[] file` + 遍历

## SpringMVC的拦截器

SpringMVc提供了拦截器机制；允许运行目标方法之前进行一些拦截工作，或者在目标方法运行之后进行一些其他处理；

就和JavaWEB中的Filter过滤器很像

通过一个接口：HandlerIntercepter，有三个方法

- preHandle：在目标运行之前调用；返回一个boolean；是否放行

- postHandle：在目标运行之后调用：

- afterCompletion：在请求整个完成之后；来到目标页面之后

拦截器是一个接口，所以要使用它的实现方法

执行顺序是

- 拦截器的preHandle-目标方法-拦截器postHandle-页面-拦截器的afterCompletion

其他流程：

- 只要不放行就不会有后面的流程
- 已经放行的拦截器的afterCompletion总会执行

```java 
public class MyIntercepter implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        System.out.println("pre");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("post");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("after");
    }
}
```

## SpringMVC的异常处理



Controller调用service，service调用dao，异常都是向上抛出的，最终有DispatcherServlet找异常处理器进 行异常的处理。

在本类里面写的异常处理可以处理本类里的异常

### ExceptionHandler

```Java
@Controller
public class MyException {
    @RequestMapping("testException")
    public String handle01(){
        System.out.println("handle01");
        System.out.println( 10 / 0);
        return "success";
    }
    
    @ExceptionHandler(value = ArithmeticException.class)
    public String handle01Execption(Exception exception){
        return "false";
    }
}
```

也可以将异常处理写成全局

```java
@ControllerAdvice
public class MyException {
    @ExceptionHandler(value = ArithmeticException.class)
    public String handle01Execption(Exception exception){
        return "false";
    }
}
```

### ResponseStatus

自定义异常

```java

@ResponseStatus(reason = "找不到",value = HttpStatus.NOT_FOUND)
public class NotFoundException extends RuntimeException {

    private static final long serialVersionUID = 1L;

}
```


























































































