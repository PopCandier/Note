#### Spring annotation

##### 传统意义上的spring如何构建

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.pop.springannotion.domain.User">
        <property name="name" value="Pop"></property>
        <property name="password" value="123"></property>
    </bean>

</beans>
```

`domain`

```java
package com.pop.springannotion.domain;

/**
 * @author Pop
 * @date 2019/4/14 22:28
 */
public class User {

    private String name;
    private String password;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

`bootstrap`

```java
package com.pop.springannotion.bootstrap;

import com.pop.springannotion.domain.User;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author Pop
 * @date 2019/4/14 22:30
 */
public class XmlConfigBootstrap {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext();

        context.setConfigLocation("classpath:/META-INF/spring/spring-context.xml");
        context.refresh();

        User user= (User) context.getBean("user");
        System.out.printf("user.name=%s,user.password",user.getName(),user.getPassword());
    }
}

```

##### Annotation

```java
@Configuration
public class UserConfiguration {

    @Bean(name="user")
    public User user(){
        User user = new User();
        user.setName("PopV5");
        return user;
    }

}
```

启动

```java
package com.pop.springannotion.bootstrap;

import com.pop.springannotion.config.UserConfiguration;
import com.pop.springannotion.domain.User;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author Pop
 * @date 2019/4/14 22:30
 */
public class AnnotationConfigBootstrap {

    public static void main(String[] args) {

        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext();

        //context.setConfigLocation("classpath:/META-INF/spring/spring-context.xml");

        context.register(UserConfiguration.class);

        context.refresh();

        User user= (User) context.getBean("user");
        System.out.printf("user.name=%s,user.password=%s",user.getName(),user.getPassword());
    }
}
```



#### 从Serlvet3.0规范说起

* `ServletContainerInitializer`->`onStartup` 当容器启动的时候



`@HandlesTypes(WebApplicationInitializer.class)`

Spring Web 的实现：`org.springframework.web.SpringServletContainerInitializer`

```java
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {

	@Override
	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
			throws ServletException {

		//.....
	}

}

```

Set原本将会从WEB-INF/classes/文件夹下去装配内容，作为servlet容器启动的时候需要的东西。

但是并不是所有的都加载，会有选择性的，这个选择就是`@HandlesTypes(WebApplicationInitializer.class)`，所以spring中能被启动时被加载的类只有继承了

`WebApplicationInitializer`的子类。

spring中存在的这个的层次关系为

* WebApplicationInitializer
  * AbstractContextLoaderInitializer
    * AbstarctDispatcherServletInitializer
      * AbstractAnnotationConrfigDispatchServletInitializer

Tomcat 6.x 实现 Servlet 2.5

Tomcat 7.x 实现 Servlet 3.0

Tomcat 8.x 实现 Servlet 3.1

Tomcat 9.x 实现 Servlet 4.0

```xml
 <plugin>
                    <groupId>org.apache.tomcat.maven</groupId>
                    <artifactId>tomcat7-maven-plugin</artifactId>
                    <version>2.1</version>
                    <executions>
                        <execution>
                            <id>tomcat-run</id>
                            <goals>
                                <goal>exec-war-only</goal>
                            </goals>
                            <phase>package</phase>
                            <configuration>
                                <path>foo</path>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
```



执行`mvn -Dmaven.test.skip -U clean package`

的时候，出现编码问题GBK

```xml
<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>
```



webapp

```xml
<?xml version="1.0" encoding="UTF-8"?>  
   
<web-app  
        version="3.0"  
        xmlns="http://java.sun.com/xml/ns/javaee"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">  
   
</web-app>

```



关于调试的小技巧，Debug的配置

Debug->Remote->修改端口，然后粘贴Command中的内容，放到控制台下，对war包进行启动

`java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=9527 target\spring-webmvc-qs-1.0-SNAPSHOT-war-exec.jar`

这里有一个东西，suspend默认是n我们改成y表示，阻塞



* `ServletContextListener`->`contextInitlaized`当`ServletContext`初始化的时候

web.xml

```xml
<webapp>
	<listener>org.springframework.web.context.ContextLoaderListener</listener>
</webapp>
```

