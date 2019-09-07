#### Servlet 到 ApplicationContext

IOC中的核心是ApplicationContext

###### 讨论Spring中的设计概念

一切从创建包开始，对于spring来说，最重要的概念

莫非于面向bean进行编程，而在此之上定义的规范

就是BeanFactory，作为spring的最顶层的借口，他的出现意味着

每一个实现他的类都将会成为一个创建Bean的工厂，且在ioc为

单例存在，因此这个借口定义了`getBean`，

他定义了规范

spring中的规范立足于此之上、被定义在`beans`包下面

这个包主要用来定义规范

而与之想对的，是规范的实现`context`包下的ApplicationContext

家族。

![1556207706989](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1556207706989.png)

可以说，ApplicationContext某种意义上，增强了Bean容器的能力。

![1556207725290](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1556207725290.png)

而为了解耦，Spring还定义了`ApplicationContextAware`这样的方法

```java
/**
 * 通过解耦的方式获得IOC的顶层设计
 * 后面将会通过一个监听器去扫描所有的类
 * 只要实现了此接口，将自动调用
 * setApplicationContext方法，将IOC容器，注入到
 * 目标类中
 * @author Pop
 * @date 2019/4/25 23:27
 */
public interface ApplicationContextAware {

    /**
     * 扫描器，将会在初始化的时候，自动容器注入到这里
     * @param applicationContext
     */
    void setApplicationContext(PApplicationContext applicationContext);
}
```

我们还建立了support包，表示这个是这个类型的增强。

附加存在的情况，还有就是BeanDefintion和BeanWrapper，前者存储bean的配置信息

后者存储实例化的信息，主要从getBean开始。



某个类输出后需要实现的借口，initAware



#### SpringMVC

```xml
 <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
```

spring中的convert，用于类型的转换

```java
PView view = viewResolver.resolveViewName(mv.getViewName,"国际化忽略")

￥{test} 匹配这样的字符
Pattern pattern = Pattern.compile("￥\\{[^\\}]+\\}",Pattern.CASE..)表示逐个匹配


//个人认为非常实用的替换匹配

RandomAccessFile ra = new RandomAccessFile(this.viewFile,"r");

String line = null;
Pattern pattern = Pattern.complie("￥\\{[^\\}]\\}","...匹配模式")；
while(null!=(line=ra.readLine())){
    line = new String(line.getBytes("ISO-8859-1"),"utf-8");
    
     Matcher matcher = pattern.matcher(line);//得到具体匹配的
     while(matcher.find()){
     	String paramName = matcher.group();//￥{test}
     	paramName=paramName.repaceAll("￥\\{|\\}","");
     	Object paramValue = model.get(paramName);
     	if(null==paramValue){continue;}
     	//替换第一个
     	line = matcher.replaceFirst(paramValue.toString());
     	matcher = pattern.matcher(line);//接着匹配下一个
     }
}


```

#### 切面

代理的顶层接口 AopProxy

在设计AOP的时候，我们设计了最顶层的接口AopProxy

并定义了两个方法

```java
public interface AopProxy {
    Object getProxy();
    Object getProxy(@Nullable ClassLoader classLoader);
}

```

然后我们完成了两个实现，一个是cglib，一个是jdk

说完实现，我们来回忆一下接下来我们做了什么

我们要准确的记录每个方法具体调用细节，在以往的操作中

我们会在xml等配置文件中定义具体的切入规则和点

而为了记录这些细节，我们设计了AdviceSupport

这个设计中存放了目标的实例还有class等信息。

接着就是将需要aop的相关通知组成一个list

method将会作为key，其它的通知将会和本身方法一起，构成调用chain

所以基本上一个adviceSupport会对应一个类。

```java
public class AdviseSupport {
    //通知的拓展，包含运行时调用的基本信息
    //Aop所配置的信息，例如注解annotion中的，活着xml
    //中，所匹配到的正则，还有表示前置和后置的一些调用
    private Class<?> targetClass;

    public Class<?> getTargetClass() {
        return targetClass;
    }

    public Object getTarget(){
        return null;
    }

    //这个方法，将会把回调函数的值组成链
    public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, @Nullable Class<?> targetClass){
        return null;
    }
}

```

如果`AdviseSupport`是设计用来存储的，那么MethodInvocation的设计

就是为了执行advice中的链的。

这个类中定义了proceed来启动这个链。

```java
public class MethodInvocation {
    //主要用于方法的拦截，spring将会对所有的方法组成一个拦截器链
    //依次执行拦截的方法

    public MethodInvocation(
            Object proxy, @Nullable Object target, Method method, @Nullable Object[] arguments,
            @Nullable Class<?> targetClass, List<Object> interceptorsAndDynamicMethodMatchers) {
            //这里的第三个参数，就是所谓的拦截器链，也就是chain
    }

    //执行方法链的中的方法
    @Nullable
    public Object proceed() throws Throwable {
        return null;
    }
}
```

当然。并不是所有的类都可以被放入链中，在原有spring实现中，只有实现了

MethodInterceptor的类才可以被调用

这个借口的设计只有一个方法

```java
public interface MethodInterceptor {
    //跟着之前执行链的问题
    //为了统一他们，并不是随便某个一个都可以放入到这个链中去
    //只有实现了MethodInterceptor接口的类才可以
    //他可以通过不断调用invoke，来通过调用自己完成一个类似回调的方法
    //完成链中的操作

    Object invoke(MethodInvocation invocation) throws Throwable;
}

```

这个methodInvocation就是我们在上面定义的，他会不断的调用自己

来达到链式调用的目的。这样就完成了aop



IOC

refresh()

定位，加载，注册

DI

getBean

initationBean()->BeanWrapper

populateBean()->依赖注入

Aop

getBean()

AdviceSupoort的配置

通过解析配置，为每个一个方法，创建一个MethodInterceptor chain



#### Spring中的事务

##### 数据库中的事务基本原理

基于JAVA 的 jdbc api

Connection 和数据库建立的一个封装，说到底就是tcp socket

Statement语句集，解析Sql语法，协议，语言，语法

ResultSet 通过执行Sql获得结果的一个封装，在java中的体现形式

(Map + Cursor)



DataSource 有些框架将这个进行了二次封装，也可以叫做

Session，例如mybatis



    采用关系模型来组织数据结构的数据库(二维表)
    
    cle    DB2    SQLServer    Mysql     SQLite都是关系型数据库
    
    优点:容易理解,它的逻辑类似常见的表格
            使用方便,都使用sql语句,sql语句非常的成熟
            数据一致性高,冗余低,数据完整性好,便于操作
            技术成熟,功能强大,支持很多复杂操作
    
    缺点:*每次操作都要进行sql语句的解析,消耗较大
             *不能很好的满足并发需求,特别是海量数据爆发,关系型
               数据库读写能力会显得不足
             *关系型数据库往往每一步都要进行加锁的操作,也造成了
               数据库的负担
              *数据一致性高,有时也会使数据的存储不灵活

 优点:高并发,读写能力强
             弱化数据结构一致性,使用更加灵活
            有良好的可扩展性

    缺点:通用性差,没有sql语句那样通用的语句
             操作灵活导致容易出错和混乱
             没有外键关联等复杂的操作



为什么数据库会插入失败，也允许插入后悔

首先会有临时表，来临时存在数据，如果发现没有外键异常

也没有主键冲突，就会合并成功，否则失败

commit，临时表允许合并

rollback，临时表销毁，也算是零食缓冲地带



所以，这也是数据库的事务的基本原理。

遵守四个基本原则

原子性，一致性（物理中能量守恒，与原子性息息相关），

隔离性（一个事务的内部操作，是相互独立的，相互不干扰），

持久性 （一旦提交，将无法回复，无法挽回）ACID



**propagation**

传播级别，TransactionDefinition  spring中的

**Propagation_Required**

支持当前事务，如果没有事务，就新建一个事务，spring中的默认事务

**Propagation_Required_new**

当A事务与B事务嵌套的时候

A事务为required，而B是new的时候，会将

A事务挂起，b的事务是一定会完成的，b的事务与a是相互独立

回滚并不能影响对方。

**Propagation_Supports**

支持当前事务，如果当前没有事务，就非事务执行

则就是，你没有我就什么也不干

.....Mandatory，也是支持，没有就抛出异常。



**隔离级别**

**Read-Uncommited** 0

会脏读，因为是读取未提交的数据（零时表的数据）

Read-commited 1

避免脏读，可以重重复读，幻读

Repeatable-Read 2

避免脏读，不可以重复读，幻读

Serializable 3

串行化读，只能一个一个读，所以不会出现

脏读，重复读，幻读等现象，但是执行效率慢



**脏读**

A 事务进行了增删改操作后未提交，B事务读取到了为提交的数据。

这个时候，如果A回滚了，那么增删改操作失效，这个时候B读到的就是

脏数据

**不可重复读**

一个事务发生了两次读操作，但是第一次和第二次之间发生了修改操作

导致这两次读操作的内容不一致

**幻读**

简单来说，就是没改到。

一个事务对某个范围内的数据进行批量修改，第二个事务在这个范围增加

一条，那么第一个事务可能会丢失掉对新数据的修改。



具体的隔离级别，需要根据业务场景和具体并发量来决定。



```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
	
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<!-- 配置事务通知属性 -->
	<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="add*" propagation="REQUIRED" rollback-for="Exception,RuntimeException,SQLException"/>
			<tx:method name="remove*" propagation="REQUIRED" rollback-for="Exception,RuntimeException,SQLException"/>
			<tx:method name="edit*" propagation="REQUIRED" rollback-for="Exception,RuntimeException,SQLException"/>
			<tx:method name="login" propagation="NOT_SUPPORTED"/>
			<tx:method name="query*" read-only="true"/>
		</tx:attributes>
	</tx:advice>

	<aop:config>
		<aop:advisor advice-ref="transactionAdvice" pointcut-ref="transactionPointcut"/>
        <aop:aspect ref="dataSource">
            <aop:pointcut id="transactionPointcut" expression="execution(public * com.gupaoedu..*.service..*Service.*(..))" />
        </aop:aspect>
    </aop:config>
```

spring中的事务顶层接口

`PlatformTransactionManager`

```java

	TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

	void commit(TransactionStatus status) throws TransactionException;

	void rollback(TransactionStatus status) throws TransactionException;

```



#### 分布式事务

cap->弱一致性->达到一致性



例如通过日志的来达到最终一致性

mysql中的binlogo





从ResultSet谈起

获得查询条件中的某一整行

//保存了，除了真正数值意外的附加信息

rs.getMetaData()





数据库的，分库分表

spring提供的动态切换组件

AbstractRoutingDataSource



### Spring5的新特性

升级JDK8，J2EE7

1、反应式编程，异步编程

2、反应式的编程

3、全面支持注解编程

4、支持函数编程

5、支持Rest风格配置

6、支持Http2.0全部支持

7、Kotlin和Spring WebFlux

8、可以直接使用lambda表达式，来注册Bean

9、Spring Web MVC 全部最新的 ServletApi

10、支持JUnit5，直接支持并发测试

11、丢弃了Hibernate3，4，只支持Hibernate5，对Portlet、

Velocity、XMLBeans，JDO，Guava终止支持

12、Spring核心容器做了一些更新

@Nullable @Logback



想要使用这些新的特性，我们需要将我们的servlet升级到3.0
因为3.0的servlet，开始支持异步。

web.xml的升级

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

	<display-name>Gupao Web Application</display-name>
  	
  	<!-- loading spring context start -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
	    <param-value>classpath:application-web.xml</param-value>
  	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
  	<!-- loading spring context end -->
	
	<!-- springmvc config start -->
	<servlet>
	  <servlet-name>dispatcher</servlet-name>
	  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	  <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
      </init-param>
	    <load-on-startup>1</load-on-startup>
		<async-supported>true</async-supported>
        <!--这里开启异步支持，不配置就会报错-->
	</servlet>
	<servlet-mapping>
	  <servlet-name>dispatcher</servlet-name>
	  <url-pattern>/*</url-pattern>
	</servlet-mapping>
	<!-- springmvc config end -->


</web-app>
```



对于Action/Controller的对象，我们则使用

Mono->用于返回单个对象

Flux->多个对象



```java
@RestController
public class MemberAction {


	@GetMapping(value="/getByName.json")
	@ResponseBody
	public Mono<Member> getByName(@RequestParam String name){
		Member member = new Member();
		member.setName(name);
		return Mono.justOrEmpty(member);
	}


	@GetMapping("/getAll.json")
	@ResponseBody
	public Flux<Member> preview(HttpServletRequest request, HttpServletResponse response){

		return null;
	}
	

	@PostMapping("/remove.json")
	@ResponseBody
	public Mono<Member> remove(HttpServletRequest request, HttpServletResponse response) {
		return null;
	}
	
	/**
	 * 获取上传进度
	 * @param request
	 * @param response
	 * @return
	 */
	@GetMapping(value="/edit.json")
	@ResponseBody
	public Flux<Member> edit(HttpServletRequest request, HttpServletResponse response){
		return null;
	}
	
}
```

用于替代HttpServlet，servlet的

HttpServletRequest->ServletRequest

HttpServletResponse->ServletResponse

去servlet



### 高频面试题

使用spring框架能给我们带来哪些好处？

**站在开发者角度：**

只需要关注业务代码

简化开发

DI:依赖关系一目了然，配置可见，就看得清楚

IOC：管理好系统中的Bean

万能胶：可以兼容其他框架

模块化的设计：即插即用，按需分配

自带测试组件：junit

Web MVC ：完美分离了Servlet和普通bean的依赖

声明式事务：将分功能性代码和功能性代码分离，

事务管理提前声明，就是去aop的配置



BeanFactory和ApplicationContext有什么区别？

首先ApplicationContext是BeanFacotory的子接口

getBean();

1、增加 IOC容器中Bean的监控，生命周期

2、支持国际化

3、扩展了统一资源的文件读取方式Url

可以是一个本地url，也可以是网络url

ClassPathXmlApplicationContext  xml文件的解析

FileSystemXmlApplicatioContext  文件系统配置

XmlWebApplicationContext 加载网络文件配置信息



AnnotationConfigApplicationContext  后面加的，通过注解



一些比较常用的调用器

ApplicationEvent

ContextRefreshedEvent 当容器调用refresh的时候就会调用

ContextStartedEvent

ContextStoppedEvent

ContextClosedEvent

RequestHandleredEvent;

http://www.cnblogs.com/EasonJim/p/6901575.html

**@PostConstruct**

**其实更简单的方法是使用注解：`@PostConstruct`，只需要在需要启动的时候执行的方法上标注这个注解就搞定了。**



**SpringBean**的生命周期

创建到销毁

有两组回调

1、初始时 InitalizingBean

销毁 DisposableBean 

用来监听Bean的一生一死

2、Aware接口

3、init() destory()

4、@PostContruct 和 @PreDestory



SpringBean**各作用域之间的区别**（Socpe）

总共5个范围

说道作用域也就是生命周期

Spring Bean 命各有长短

规定了5钟寿命

1、什么时候用，什么出生，用完就死了(prototype)

2、容器开始就诞生直到消亡(singleton)

//。。。。



在service获得ApplicationContext

实现ApplicationAware接口









