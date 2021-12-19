---
title: 'Spring AOP'
date: 2020-09-05 23:00:00
tags: [javaweb框架]
categories: javaweb
published: true
hideInList: true
feature: 
isTop: false
---

AOP: 面向切面编程：基于OOP基础之上的新的编程思想

指的是在程序运行期间，将某段代码动态的切入到指定方法的指定位置进行运行的这种编程方式，面向切面编程

场景：计算器运行计算方法的时候进行日志记录

<!-- more -->

## 第一个AOP程序

- 第一步Maven依赖导入

```xml
<!--Spring基础包-->
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>

<!--面向切面编程的包-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
        <dependency>
            <groupId>aopalliance</groupId>
            <artifactId>aopalliance</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.3.0</version>
        </dependency>

    </dependencies>
```

- 第二步将目标类和切面类(封装了通知方法(在目标方法执行前后执行的方法))加入到IOC容器中

  目标类：

```java
@Controller
public class MyMathCalculator implements Calculator {

    public int add(int a, int b) {
        return a + b;
    }
}
```

- 切面类：

```java
@Aspect
@Component
public class LogUtils {

    @Before("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
    public static void logStart(){
        System.out.println("方法开始执行");
    }

    @AfterReturning("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
    public static void logReturn(){
        System.out.println("方法开始返回");
    }

    @AfterThrowing("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
    public static void logExpect(){
        System.out.println("方法报错");
    }



    @After("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
    public static void logAfter(){
        System.out.println("方法结束");
    }

//    @Around("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
//    public static void logAround(){
//
//    }

    
}
```

- 第三步开启AOP的注解模式：

```xml
<context:component-scan base-package="com.xian" />

<aop:aspectj-autoproxy/>
```

- 测试

```java
public class TestCalculate {
    private final ApplicationContext ioc = new ClassPathXmlApplicationContext(
            "applicationContext.xml");
    
    @Test
    public void test(){
        MyMathCalculator calculator = (MyMathCalculator) ioc.getBean(MyMathCalculator.class);
        System.out.println(calculator.getClass());
        //这里获取的类一定不是原类，而是代理类
        int add = calculator.add(1, 2);
        System.out.println(add);
    }
}
```

## 注意事项：

### AOP的底层就是动态代理

组件中保存的对象就是他的代理对象，当然不是本类的类型。Spring获取没有实现接口的对象时，会自动创建一个内部类，这个内部类就是代理对象。

### 切入点表达式的写法

固定格式：`execution(访问权限符 返回值类型 全类名.方法名称(参数类型,参数类型))`

- 使用`*`代替任意方法名称，任意返回值类型，任意类名
- 使用`..`表示上一级目录，任意参数

### 通知方法的执行顺序

- 程序有报错时：
  - 方法开始执行
  - 方法报错
  - 方法结束

- 程序正常运行时
  - 方法开始执行
  - 方法开始返回
  - 方法结束

### 获取目标方法运行时使用的参数

```java
    @Before("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
    public static void logStart(JoinPoint joinPoint){
        System.out.println( joinPoint.getSignature().getName() +  "方法开始执行");
    }
```

1. 只需要为通知方法的参数列表上写一个参数：`JoinPoint joinPoint`，这个封装了当前目标方法的详细信息。
2. 告诉Spring哪个参数是用来接受异常

```java
@AfterThrowing(value = "execution(public int com.xian.impl.MyMathCalculator.*(int,int))",throwing = "e")
public static void logExpect(Exception e){
    System.out.println(e + "方法报错");
}
```

3. 唯一的要求是方法的参数列表一定不能乱写，通知方法是Spring调用的，方法乱写会出错

### 抽取可重用的切入点表达式

随便声明一个没有实现的返回void的空方法，给方法上标注@PointCut

```java
@Pointcut("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
public void myPointCut(){};

@Before("myPointCut()")
public static void logStart(JoinPoint joinPoint){
    System.out.println( joinPoint.getSignature().getName() +  "方法开始执行");
}
```

### 环绕通知

- 环绕通知是所有通知类型中功能最为强大的，能够全面地控制连接点，甚至可以控制是否执行连接点。

- 对于环绕通知来说，连接点的参数类型必须是ProceedingJoinPoint。它是 JoinPoint的子接口，允许控制何时执行，是否执行连接点。

- 在环绕通知中需要明确调用ProceedingJoinPoint的proceed()方法来执行被代理的方法。如果忘记这样做就会导致通知被执行了，但目标方法没有被执行。

- 注意：环绕通知的方法需要返回目标方法执行之后的结果，即调用 joinPoint.proceed();的返回值，否则会出现空指针异常。
- **环绕通知优先执行**

```java
@Around("execution(public int com.xian.impl.MyMathCalculator.*(int,int))")
public static Object logAround(ProceedingJoinPoint pjp){
    Object proceed = null;
    try {
        System.out.println("before");//前置通知
        proceed = pjp.proceed(pjp.getArgs());
        System.out.println("return");//返回通知
    } catch (Throwable throwable) {
        throwable.printStackTrace(); //执行方法
        System.out.println("Exception");//异常通知
    }finally {
        System.out.println("after");//后置通知
    }
    return proceed;
}
```

### 多个切面程序执行顺序

先进先出，后进后出，就是一句话，很简单

应用场景：

- AOP加日志保存到数据库
- AOP做权限验证
- AOP做安全检查
- AOP做事务

### AOP的xml配置方法

1. 将目标类和切面类都加入到IOC容器中
2. 告诉Spring哪个是切面类
3. 在切面类中使用五个通知来配置切面中的这些方法都在何时何地运行

```xml
    <aop:aspectj-autoproxy/>
    
    <bean id="calculator"  class="com.xian.impl.MyMathCalculator"/>
    <bean id="logUtils" class="com.xian.utils.LogUtils"/>
    
    <aop:config>
        <aop:aspect ref="logUtils">
<!--            切入点表达式-->
            <aop:pointcut id="mypointcut" expression="execution(* com.xian.impl.*.*(..))"/>
            <aop:after method="logAfter" pointcut-ref="mypointcut"/>
            <aop:before method="logStart" pointcut-ref="mypointcut"/>
            <aop:around method="logAround" pointcut-ref="mypointcut"/>
            <aop:after-throwing method="logExpect" pointcut-ref="mypointcut" throwing="e"/>
            <aop:after-returning method="logReturn" pointcut-ref="mypointcut" />
        </aop:aspect>
    </aop:config>
```