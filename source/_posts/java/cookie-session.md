---
title: 'Cookie与Session'
date: 2020-08-23 21:20:00
tags: [javaweb]
categories: javaweb
published: true
hideInList: true
feature: 
isTop: false
---

## 什么是Cookie

1. cookie翻译过来是饼干的意思。
2. cookie是服务器通知客户端保存键值对的一种技术，这些信息由客户端保存
3. 客户端有了cookie之后可以发送给服务器
4. 每个cookie的大小不得超过4kb

<!-- more -->

## Cookie的创建

```java
protected void createCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    System.out.println("chuang");
    //创建cookie对象
    Cookie cookie = new Cookie("key1","value1");
    //通知客户端保存cookie
    resp.addCookie(cookie);
    resp.getWriter().write("cookie 创建成功");
}
```

## Cookie的获取

### 获取全部Cookie

```java
protected void getCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    System.out.println("chuang");
    //获取cookie对象
    Cookie[] cookies = req.getCookies();
    for (Cookie cookie : cookies) {
        resp.getWriter().write(cookie.getName() +"+"+ cookie.getValue() + "<br/>");
    }
}
```

### 获取单个cookie

```java
    for (Cookie cookie : cookies) {
        if("cookieneededname" = cookie.getName()){
            resp.getWriter().write(cookie.getName() +"+"+ cookie.getValue() + "<br/>");
        }
    }
```

## Cookie值的修改

### 方案一：重新创建

1. 先创建一个要修改的同名的Cookie对象
2. 在构造器中赋予新的Cookie值
3. 调用`response。addCookie(Cookie)`通知客户端保存

### 方案二：使用API修改

1. 先查找到需要修改的Cookie对象
2. 调用`setValue()`方法赋予新的Cookie值
3. 调用`response。addCookie(Cookie)`通知客户端保存

```java
protected void updateCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //获取cookie对象
    Cookie[] cookies = req.getCookies();
    for (Cookie cookie : cookies) {
        if ("cookieName".equals(cookie.getName())) {
            cookie.setValue("okok");
            resp.addCookie(cookie);
            resp.getWriter().write(cookie.getName() + "+" + cookie.getValue() + "<br/>");
        }
    }
}
```

## Cookie 的生命控制

Cookie的生命控制指如何管理Cookie什么时候被销毁(删除)

`setMaxAge()`

- 正数，表示在指定的秒数后过期
- 负数，表示浏览器一关，cookie就会被删除(默认值就是-1)
- 零，表示马上删除Cookie

无论设置何种生命控制都需要调用`resp.addCookie(cookie);`

```java
protected void deleteNow(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //获取cookie对象
    Cookie[] cookies = req.getCookies();
    for (Cookie cookie : cookies) {
        if ("cookieName".equals(cookie.getName())) {
            cookie.setMaxAge(0);
            resp.addCookie(cookie);
            resp.getWriter().write(cookie.getName() + "+" + cookie.getValue() + "<br/>");
        }
    }
}
```

## Cookie有效路径Path的设置

Cookie的path属性可以有效的过滤哪些Cookie可以发送给服务器，哪些不发

Path属性是通过请求的地址来进行有效的过滤。

| CookieA | path=/cookie |
| ------- | ------------ |
| CookieB | path=/       |

请求地址如下：

| `http://ip:port/工程路径/a.html`        | CookieA发送  CookieB不发送 |
| --------------------------------------- | -------------------------- |
| `http://ip:port/工程路径/cookie/a.html` | CookieA不发送  CookieB发送 |

设置方法为：`cookie.setPath(req.getContextPath() + "/cookie");`



## 什么是Session会话

1. session就是一个接口(HttpSession)
2. Session就是会话，他用来维护一个客户端和服务器中间关联
3. 每个客户端都有自己的一个Session会话
4. Session会话中，我们经常用来保存用户登录信息

## 如何创建Session和获取(ID号是否为新)

创建Session和获取Session的API都是一样的。

request.getSession()

- 第一次调用是：创建Session会话
- 之后：获取之前创建的Session会话

`isNew()`判断Session是否是新创建出来的

- true表示刚创建出来
- FALSE表示获取之前创建

每个Session都有一个身份证号。也就是ID值，这个ID值是唯一的

`getId()`可以得到Session会话的ID值

## Session域数据的存取

```java
protected void setAttribute(HttpServletRequest req,HttpServletResponse resp) throws ServletException, IOException {
	req.getSession().setAttribute("key1", "value1");
	resp.getWriter().write("已经往 Session 中保存了数据"); }
protected void getAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	Object attribute = req.getSession().getAttribute("key1");
	resp.getWriter().write("从 Session 中获取出 key1 的数据是:" + attribute); }
```

## Session生命周期控制

`public void setMaxInactiveInterval(int interval)`设置Session的超时时间(以秒为单位)，超过指定时间，Session就会被销毁。

- 值为正数时，设定Session的超时时长
- 值为负数时，设定Session永不超时(极少使用)

`public int getMaxInactiveInterval()`获取 Session 的超时时间

`publicvoidinvalidate()`让当前Session会话马上超时无效。

Session的默认超时时长为30分钟

在 Tomcat 服务器的配置文件 web.xml 中默认有以下的配置，它就表示配置了当前 Tomcat 服务器下所有的 Session 超时配置默认时长为:30 分钟。

```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

若需要修改web工程默认的Session时长为其他值，可以自己在web.xml中修改上述配置

## Session的工作方式



![Session](https://tva1.sinaimg.cn/large/007S8ZIlly1gi1ozspa7uj31m50u0qbd.jpg)






























