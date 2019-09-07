#### Spring Boot的另一种理解

服务于spring 框架的框架



##### 约定由于配置的体现

* maven 的目录结构
* spring-boot-starter-web。内置tomcat、resource、template、static
* 默认的application.properties



#### SpringBoot里面没有新技术

* AutoConfiguration 自动装配
* Starter
* Actuator
* SpringBoot CLI



AutoConfigurationImportSelector与AutoConfigurationPackages.Registrar

这两个类位于@EnableAutoConfiguration中

在spring中，可不可以根据上下文来激活不同的bean

动态注入

ImportSelector
ImportBeanDefinitionRegistrar

首先实现ImportSelector

[SpringBoot自动装配扩展](https://github.com/PopCandier/SpringBootDemo/blob/master/src/main/java/autoconfig/Spring%20Boot%20autoconfig.md)

SPI 扩展点

也就是我们之前的META-INF/spring.factories 下面的内容

写在这里的内容将会被自动装配，为了导入的时候将会内容，key就是注解的路径名

* 一样的目录结构
* 一样的文件名
* 一样的key

ConditionalOnBean

条件注解，首先，他的用法和上面一样

META-INF/spring-autoconfigure-metadata.properties

可以在spring-boot,autoconfiguration包下找到。

```properties
com.pop.Person.ConditionalOnClass = com.pop.Teacher
# 表示当 com.pop.Teacher 存在的时候，Person对象将会被被spring容器接管
```



#### 什么是starter

一个模块，可以自定义，基于条件自动装配

其实就可以将模块的内的类，自动装配托管到spring中

starter的命名规则

spring-boot-starter-{name}

{name}-spring-boot-starter  非官方的



```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
    <!--由于一开始spring只支持ylm文件，加上这个后，可以支持xml，和properties-->
</dependency>
```



#### spring-boot-starter-logging

mic老师突然讲了这个

