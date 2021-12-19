---
title: 'Spring IOC'
date: 2020-09-03 23:00:00
tags: [javaweb框架]
categories: javaweb
published: true
hideInList: true
feature: 
isTop: false
---

通过17个实验来了解SpringIOC的基础知识

### 实验1: 第一个Spring项目

新项目的开始需要的步骤是：

导包、配置、测试

1. #### Maven 项目需要的依赖

```xml
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
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.6</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
    </dependency>
</dependencies>
```

<!-- more -->

2. #### 新建SpringConfigratioin配置文件，例如：SpringIOC.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.xian.bean.Person">
        <property name="name" value="qiqi"/>
        <property name="age" value="12"/>
    </bean>
</beans>
```

3. #### 新建容器中的一个组件（Bean），例如`Person`类

```java
package com.xian.bean;

import lombok.Data;

@Data
public class Person {
    private String name;
    private Integer age;

}
```

4. #### 测试使用的方法

```java
public class MyTest {
    @Test
    public void test01(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("springconfig.xml");
        Object person = applicationContext.getBean("person");
        System.out.println(person);
    }
}

```

#### 测试的细节：

- ApplictionContext（IOC容器的接口）
- 给容器中注册一个组件，并且从容器中按照`id`拿到了这个组件的对象
  - 组件的创建是由容器完成的
  - 容器中组件的创建在容器创建完成的时候就已经创建好了
- 同一个组件在IOC容器中是单实例
- 容器中没有组件时的错误信息：`org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'person1' available`
- IOC容器在创建这个组件的时候是使用getter/setter方法进行赋值
- JavaBean的属性名称是由getter/setter方法后面的名称首字母小写决定的

### 实验2: 根据bean的类型从IOC容器中获取bean实例

代码语法 `Object person = applicationContext.getBean(Person.class);`

注意事项：

- 如果IOC容器中这个类型的bean有多个，查找就会失败，错误信息：`org.springframework.beans.factory.NoUniqueBeanDefinitionException`
- `Object person = applicationContext.getBean("person",Person.class);`

### 实验3: 使用构造方法创建bean实例

```xml
<!--可以使用属性名称-->
    <bean id="person2" class="com.xian.bean.Person">
        <constructor-arg name="name" value="qiqi"/>
        <constructor-arg name="age" value="12"/>
    </bean>
<!--省略name属性，但是要严格按照顺序-->
    <bean id="person3" class="com.xian.bean.Person">
        <constructor-arg value="qiqi"/>
        <constructor-arg value="12"/>
    </bean>
<!--指定index参数-->
    <bean id="person4" class="com.xian.bean.Person">
        <constructor-arg value="12" index="1"/>
        <constructor-arg value="qiqi" index="0"/>
    </bean>
<!--指定type参数-->
    <bean id="person5" class="com.xian.bean.Person">
        <constructor-arg value="12" type="java.lang.Integer"/>
        <constructor-arg value="qiqi" type="java.lang.String"/>
    </bean>
```

使用名称空间：

- 名称空间是为防止标签重复而引入的带前缀的标签 使用p名称空间

`xmlns:p="http://www.springframework.org/schema/p"`

```xml
<!--    p标签赋值的方法-->
    <bean id="person6" class="com.xian.bean.Person"
        p:age="18" p:name="qiqi">
    </bean>
```

### 实验4: 正确的为各种属性赋值

```xml
<!--    默认不赋值都是null-->
    <bean id="student" class="com.xian.bean.Student">
<!--    赋值null的方法-->
        <property name="name">
            <null/>
        </property>
<!--    赋值引用类型的方法 引用外部bean-->
        <property name="father" ref="person"/>

<!--    赋值引用类型的方法 引用内部bean-->
        <property name="mother" >
<!--        内部bean不能指定id不能获取-->
            <bean class="com.xian.bean.Person"
                  p:age="18" p:name="xixi">
            </bean>
        </property>

<!--    赋值list类型属性的方法 引用内部bean-->
        <property name="friends">
            <list>
                <ref bean="person2"/>
                <ref bean="person3"/>
            </list>
        </property>

<!--    赋值map类型属性-->
        <property name="map">
            <map>
                <entry key="01" value="xiaobai"/>
                <entry key="02" value-ref="person4"/>
                <entry key="02">
                    <bean class="com.xian.bean.Person"
                          p:age="18" p:name="xixi"/>
                </entry>
            </map>
        </property>

<!--    赋值properties类型属性-->
        <property name="properties">
            <props>
                <prop key="username">xiaoxi</prop>
            </props>
        </property>
    </bean>
```

使用utils名称空间

`xmlns:util="http://www.springframework.org/schema/util"`

```xml
<util:map id="myMap">
    <entry key="01" value="xiaobai"/>
    <entry key="02" value-ref="person4"/>
</util:map>

<bean id = "student2" class="com.xian.bean.Student">
    <property name="map" ref="myMap"/>
</bean>
```

级联属性赋值

```xml
        <property name="father" ref="person"/>
        <property name="father.name" value="xiaoxiao"/>
```

可以修改属性的属性，但是原来属性已经被修改了



### 实验5: 通过静态工厂的方法创建的bean、实例工厂方法创建的bean

bean的创建默认就是框架利用反射new出来的bean实例

工厂模式的好处就是创建的细节我们不需要知道，工厂帮我们创建对象；有一个专门的类帮我们创建，这个类就叫做工厂类。



静态工厂：工厂本身不用创建对象；通过静态方法调用创建对象：对象= 工厂类.工厂方法名（）

实例工厂：工厂本身也需要被创建



#### 静态工厂：不需要创建工厂本身，工厂方法一定是静态方法

```java
@Data
public class House {

    private String door;
    private String windows;
    private Integer tall;
}


/**我是两个类的分割线
*/
import com.xian.bean.House;

public class HouseStaticFactory {
    public static House getHouse(String s) {
        House house = new House();
        house.setDoor("xioami");
        house.setTall(100);
        house.setWindows(s);
        return house;
    }
}
```

通过`constructor-arg`来进行传参

```xml
<bean id="myhouse" class="com.xian.factory.HouseStaticFactory"
      factory-method="getHouse">
    <constructor-arg value="xiaomei"/>
</bean>
```

#### 实例工厂：工厂本身也需要被创建

实例工厂类的创建，方法不需要是静态的。

```java
public class HouseInstanceFactory {
    
    public House getHouse(String s) {
        House house = new House();
        house.setDoor("xioami");
        house.setTall(100);
        house.setWindows(s);
        return house;
    }
}
```

bean配置类，Spring会使用`factory-bean`来确定工厂类。

```xml
<bean id="houseInstanceFactory" class="com.xian.factory.HouseInstanceFactory"/>

<bean id="house2" factory-bean="houseInstanceFactory" factory-method="getHouse">
    <constructor-arg value="huahua"/>
</bean>
```



实现`FactoryBean<>`接口来创建一个Spring认识的工厂类

```java
public class FactoryBeanImpl implements FactoryBean<House> {

    public House getObject() throws Exception {
        House house = new House();
        house.setDoor("xioami");
        house.setTall(100);
        return house;
    }

    public Class<?> getObjectType() {
        return House.class;
    }

//是否是单实例
    public boolean isSingleton() {
        return false;
    }
}

```

直接配置，会被Spring自动识别

```xml
    <bean id="factoryBeanImpl" class="com.xian.factory.FactoryBeanImpl"/>
```

### 实验6: 通过继承实现bean配置信息的重用

```xml
<!--    通过abstract属性创建一个模板bean-->
    <bean id="person" class="com.xian.bean.Person" abstract="true">
        <property name="name" value="qiqi"/>
        <property name="age" value="12"/>
    </bean>

<!--    通过parent属性继承模板bean-->
    <bean id="xiaoxiao" parent="person">
        <property name="name" value="xiaoxiao"
    </bean>
```

不能获取该bean的错误：





### 实验7: bean之间的依赖

只改变bean之间的创建顺序，并没有实际上的依赖

```xml
    <bean id="person" class="com.xian.bean.Person" />
    <bean id="student" class="com.xian.bean.Student" depends-on="person"/>
```



### 实验8: 测试bean的作用域，分别创建单实例和多实例的bean

```xml
    <bean id="person" class="com.xian.bean.Person" scope="prototype"/>
    <bean id="person1" class="com.xian.bean.Person" scope="singleton"/>
    <bean id="person2" class="com.xian.bean.Person" scope="session"/>
    <bean id="person3" class="com.xian.bean.Person" scope="request"/>
```

容器中有这样一个属性`scope`，这个属性有下面的值

- prototype 多实例 每次创建的都是不同的对象
- singleton 单实例（默认）



### 实验9: 创建带有生命周期方法的Bean

生命周期，ioc容器中注册的bean

- 单实例，容器启动的时候就会创建好，容器关闭也会销毁创建的bean
- 多实例，获取的时候才会创建

bean的生命周期 构造器- 初始化方法- （容器关闭）销毁方法

```xml
<bean id="house" class="com.xian.bean.House"
destroy-method="destory" init-method="init"/>
```

### 实验10: 测试bean的后置处理器

实现BeanPostProcessor接口的方法

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之前调用");
        return null;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之后调用");
        return null;
    }
}
```

在bean中的配置

```xml
<bean id="myBeanPostProcessor" class="com.xian.bean.MyBeanPostProcessor"/>
```

### 实验11: 引用外部属性文件

可以创建数据库连接池（使用Spring管理连接池）

从IOC容器拿到数据库连接池

### 实验12: 基于XML的自动装配

```xml
<!--    自动装配-->
<bean id="person111" class="com.xian.bean.Student" autowire="byType"/>
```

- autowire="byType"
  按照属性的类型去容器中寻找到这个组件去给他赋值，找不到装配null找到多个时就会报错
- autowire="byName"
  按照属性名作为id去容器中寻找这个组件，给他赋值，如果找不到就装配null
- autowire="constructor"
  按照构造器去进行赋值，
  - 先按照有参构造器参数的类型进行赋值，成功就赋值；没有就直接装配为null
  - 如果找到多个类型的组件，就以参数的名称为id进行装配，成功就进行赋值；没有就直接装配为null

### 实验13：[SpEL测试]

就是Spring的表达式语言`#{}`

```xml
<bean id="person111" class="com.xian.bean.Student" >
    <property name="age" value="#{12*12}"/>
    <property name="name" value="#{house2.windows}"/>
</bean>
```

静态方法的调用`UUID。randomUUID().toString()` `#{T(全类名).静态方法名()}`

静态方法的调用`#{对象.方法名()}`

### 实验14: 使用注解将bean添加到容器中

通过给bean上添加注解可以将这个组件添加到IOC容器中

Spring有四个注解：

@Controller 给控制层添加的注解

@Services 给业务层添加注解

@Repository 给数据库(持久化层添加注解)

@Component 可以给不属于以上几层的组件添加注解

但是这个注解的信息是给程序员看的

使用注解将组件快速添加到容器中需要几步

- 给需要添加的组价上标注上述注解中的任何一个

- 告诉Spring要自动扫描：依赖context名称空间

```xml
<context:component-scan base-package="com.xian"/>
```

- 一定要导入AOP包

```java
@Data
@Controller
@Scope(value = "singleton") // 可以添加实例类型的注解
public class Ha {
    private Integer age;
}
```

可以按照某种方法排除不需要扫描的类：分别是指定的注解和指定的类名称

```xml
<context:component-scan base-package="com.xian">
    <context:exclude-filter type="annotation" expression=""/>
    <context:exclude-filter type="assignable" expression=""/>
</context:component-scan>
```

可以只扫描一定的包`use-default-filters="false"`

```xml
<context:component-scan base-package="com.xian" use-default-filters="false">
    <context:include-filter type="assignable" expression=""/>
</context:component-scan>
```

### 实验15: 使用@Autowired自动装配

```java
@Data
@Controller
public class Person {

    @Autowired
    private House house;

}
```

- 先按照参数的类型进行自动装配，只有一个就成功装配；没有就直接报错
- 如果找到多个匹配类型的组件，就以参数的名称为id进行装配，成功就成功装配；没有就直接报错
- @Qualifier 指定一个名称为id来进行自动装配，找不到就报错
- @Autowired(required= false) 找不到就自动装配null

### 实验16: 使用Spring的单元测试

@ContextConfigration(location= "") 用来指定Spring的配置文件的位置

@Runwith(Spring) 使用Spring的单元测试来执行标注了@Test的方法

好处是不用使用`getbean`可以使用`@Autowired`自动获取

### 实验17: 泛型依赖注入