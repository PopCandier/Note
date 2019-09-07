#### SpringMVC

M:Model

V:View

C:Controller->DispatcherServlet

Front Controller = DispatcherServlet

前段总控制器



J2EE规范

Servlet的匹配规范

servlet的精准匹配，还有模糊匹配

在创建一个web项目的时候

`@WebServlet("/")   @WebServlet("/*")`

@RestController=@Controller+@ResponseBody

spring驱动，补课

springmvc的配置bean WebMvcProperies

SpringBoot允许通过application.properiese定义配置

配置前缀，spring.mvc

spring.mvc.servlet

HandlerMapping 

寻找Request URI匹配的Handler

处理Handler映射

Handler是处理的方法，当然也是一种实例

Request->Handler->执行结果->返回(Rest)->普通文本

方法拦截器

接口HandlerInterceptor

处理顺序：

preHandle(true)

->handle(其实就是url对应的方法)

->postHandle->afterCompletion



#### 异常处理

##### Servlet 标准

web.xml(.xsd)，错误页面

```xml
<error-page>
	<error-code>404</error-code><!--处理状态码-->
    <exception-type></exception-type><!--处理异常类型-->
    <location>/404.html</location><!--处理页面-->
</error-page>
```

eclipse中的display工具，可以执行这个页面的调试代码

##### Spring MVC

```java
@ExceptionHandler //挂载到响应拦截方法，主要处理异常，传入异常的class
@RestControllerAdvice(basePage="响应包名，表示只拦截这个包下的controller")=@ControllerAdvice+@ResposneBody
@ControllerAdvice专门拦截@Controller
```

- 

##### Spring Boot

```java
@SpringBootApplication
public class SaoApplication extends WebMvcConfigurerAdapter
		implements ErrorPageRegistrar {
	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND,"/404.html"));
	}

	public static void main(String[] args) {
		SpringApplication.run(SaoApplication.class, args);
	}

	public void addInterceptors(InterceptorRegistry registry) {//添加拦截器
		registry.addInterceptor(new DefaultHandlerInterceptor());
	}
}

```

另一个controller中

```java
 @GetMapping("/404.html")
    public Object pageNotFount(HttpStatus status, HttpServletRequest request,
                               Throwable throwable){
        Map<String,Object> errors = new HashMap<>();

        errors.put("statusCode",request.getAttribute("javax.servlet.error.status_code"));
        errors.put("statusUri",request.getAttribute("javax.servlet.error.status_uri"));

        return errors;
    }
```

- 实现ErrorPageRegistrar
  状态码，比较通用，不需要理解springWebMVC的异常体系

  不足：页面处理路径必须固定

- 注册ErrorPage对象

- 实现ErrorPage对象中的Path路径web服务



#### 视图技术

##### View

render方法，处理页面渲染的方法 例如，Velocity/jsp/Thymeleaf

##### ViewResolver

ViewResolver=页面+解析器->resolveViewName

寻找合适的对象

RequestURI->RequestMappingHandlerMapping->

HandleMethod->retunr "viewName" ->

完整的页面名称=prefix+"viewName"+suffix->ViewResolver->View

->render->HTML

`最佳匹配原则`

`ContentNegotiationViewResolever`

用于处理多个ViewResolver:JSP,Velocity,Thymeleaf

Spring Boot解析完整页面的路径

spring.view.prefix+Hand;erMethod reutrn+spring.view.suffix

##### Thymeleaf

ThymeleafAutoConfiguration

```xml
<!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf -->
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.9.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

配置项前缀:spring.thymeleaf

模版寻找前缀:spring.thymeleaf.prefix

模版寻后缀:spring.thymeleaf.suffix

```html
<!DOCTYPE html>
<html  xmlns:th="http://www.thymeleaf.org" >
<head>
<meta charset="UTF-8" />
<title>Spring MVC + Thymeleaf Example</title>
</head>
<body>
   
</body>
</html>
```



#### 国际化(i18n)

Locale/LoacleContext

LocaleResolver

`accapect-type`

```html
https://blog.csdn.net/u012100371/article/details/78199568
```





关于返回的请求，还有responsebody的处理

RequestResponseBodyMenthodProcessor



#### 关于Spring MVC中的接收类型的

问题：为什么第一个次是json，后来加了Xml后变成了

xml

回答：Spring Boot应用默认没有增加Xml处理器

实现，所以最后采用了轮训的方式，是否可以

canWrite(POJO)，如果能返回true，说明可以

序列化改POJO对象，那么Jackson2刚好可以

处理，这也是为什么我们导入了xml转换的包

的时候，会转化成xml



问题：当Accept请求头未被定制的时候，为什么

还是Json来处理、

回答：这个依赖与messageConverters的插入顺序



问题：优先级是默认的吗，可以修改吗

回答：是可以调整的，通过

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    //额外的配置拓展，主要是为了对自定义媒体类型，MIME进行拓展
    //关于接受的到参数，返回json或者xml
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        //由于list，先进入先服务的特点，如果你希望调整服务的顺序的话
        //可以直接设置
        converters.set(0,new MappingJackson2HttpMessageConverter());//强制提升为最高级别
        //converters.add(new MappingJackson2HttpMessageConverter());
    }
}
```

#### 扩展自描述消息

可以从`WebMvcConfigurationSupport`中寻找支持的依赖

`search.maven.org`

Json格式(application/json)

```json
{
    "id": 1,
    "name": "hello"
}
```

Xml格式(application/xml)

```xml
<person>
	<id>1</id>
    <name>hello</name>
</person>
```

我们定义的Properies(application/propertis+person)

```properties
person.id=1
person.name=pop
```

#### Spring Boot JDBC

##### 数据源 DataSource

类型

-通用型数据源

javax.sql.DataSource

-分布式数据源

javax.sql.XADataSource

-嵌入式数据库（wechat）

org.springframework.jdbc.datasource.

embedded.EmbebdedDatabase

下载时，使用 reactiveweb，jdbc mysql等模块

> 如果使用spring Boot 2.0
>
>  springwebmvc默认情况下使用嵌入式tomcat
>
> 如果采用spring web flux 使用的是netty web
>
> WebFlux
>
> > Mono: 0-1 Publisher （类似于Java8中的Optional）
> >
> > Flux: 0-N Publisher (类似于java中的list)
> >
> > 传统的Servlet 采用的是HttpServletRequest HttpServletRepseon
> >
> > WebFlux采用：ServerRequest、ServlerResponse
> >
> > 不再限制与Servlet容器，可以自定义实现，例如Netty



`DataSource`

数据连接池化

DBCP



单数据源场景



多数据源场景

```java
@Configuration
public class MultipDataSourceConfiguration {
    //配置多数据源
    @Bean
    @Primary
    public DataSource masterDataSource(){
//        spring.datasource.driverClassName = com.mysql.jdbc.Driver
//        spring.datasource.url=jdbc:mysql://localhost:3306/gp
//        spring.datasource.username=root
//        spring.datasource.password=root

        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        DataSource dataSource=dataSourceBuilder.
                driverClassName("com.mysql.cj.jdbc.Driver")
                .url("jdbc:mysql://localhost:3306/gp?characterEncoding=utf8&serverTimezone=GMT%2B8")
                .username("root").password("root").build();
        return dataSource;
    }

    @Bean
    public DataSource salveDataSource(){
        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        DataSource dataSource=dataSourceBuilder.
                driverClassName("com.mysql.cj.jdbc.Driver")
                .url("jdbc:mysql://localhost:3306/ssm?characterEncoding=utf8&serverTimezone=GMT%2B8")
                .username("root").password("root").build();
        return dataSource;
    }
}
```

单数据源无法定义

@Qualifier 表示使用过@Service("xxx")中的别名，表示指定确定要注入的是哪一个

##### 事务

保证数据的完整性

@Transactional 代理执行 `transactionInterceptor`

可以指定 rollback 级别:rollbackfor,norollbaclfor

可以指定事务管理器:transactionManager()

还可以使用API的方式

PlatformTransactionManager 

隔离界别和事务传播

事务与还原点相关。

spring中的重用了JDBC API

Islation->TransactionDefinition



----

#### Spring Boot 初体验

maven.apache.org/plugin 有一些插件的命令

Effective Java II



web flux与web mvc 无法一起使用

Req->webFlux->1-n线程执行任务执行函数式任务7777



##### 构建多模块应用

1.修改主工程类型

<packaging>jar</packaging>

<packaging>pom</packaging>



2.创建一个新应用

选中当前的项目->新建module，创建

对应好java文件，直接拖过拉里，然后删除掉原来的项目src

下面的。



3 关于其他模块的选取，

最好建立一个和之前模块一样的包

由于移动过去的时候，过去的依赖可能会改变

所以，我们需要修改model的pom.xml文件

并且把响应的依赖写道主文件中去

```xml
<artifactId>model</artifactId>
```

各种其他依赖应该写好，保证正确的依赖关系

web->presistence->model



##### 关于各模块的打包

`mvn -Dmaven.test.skip -U clean package`

如果打包出现异常，多半是找不到main class

我们可以在主pom加入这个参数

```xml
<properties>
        <start-class>com.pop.springbootjdbc.SpringBootJdbcApplication</start-class>
    </properties> 
<build>
      <plugin>
          <groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <mainClass>${start-class}</mainClass>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>repackage</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</build>

```

#### 注意你的主启动在哪里的，你的mainclass就应该写在哪里

保证完整依赖关系

生成的文件，将会在target的目录下，有两个文件

你也可以通过explorer .来打开那个目录下的文件



Boot-INF是sringBoot 1.4才发布的内容

![1555141230733](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1555141230733.png)

当使用的依赖，或者插件的时候，如果版本是Milestone时候，需要

增加

```xml
<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</pluginRepository>
		<pluginRepository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</pluginRepository>
	</pluginRepositories>
```

* 在META-INF/MENIFEST.MF，有指定两个属性
  * 指定Main-class
  * Start-class