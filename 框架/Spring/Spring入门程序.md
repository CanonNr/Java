## 创建项目

为了放方便引入 jar 包使用了`Maven`构建项目

本文使用 `XML` 配置

入门学习，任何包名类名没实际意义

## 导入  jar  包

 ```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.3.RELEASE</version>
    </dependency>
</dependencies>
 ```

## 创建几个测试类

### BaseService

```java
public class BaseService {
    public String get(){
        return "Hello lksun ~";
    }
}
```

### UserService

```java
public class UserService {
    @Autowired
    BaseService baseService;
	
    public void echo(){
        System.out.println(baseService.get());
    }
}
```

### UserController

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("Spring.xml");
    UserService userService = context.getBean(UserService.class);
    userService.echo();
}
```



## 创建配置文件

在 `resources` 目录下创建一个 `Spring.xml` 文件（名字可自定义）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 两个类 -->
    <bean id="userService" class="com.lksun.service.UserService"/>
    <bean id="baseService" class="com.lksun.service.BaseService"/>
    <!-- 扫描Bean -->
    <context:component-scan base-package="com.lksun"/>
</beans>
```



## 输出结果

```java
Hello lksun ~
```

