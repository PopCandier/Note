#### Spring框架的前世今生

spring的诞生就是为了简化开发。

命名规范，行为特征，通用逻辑

DI->IOC<-AOP 基于OOP（面向对象编程），BOP(面向Bean编程)

核心容器。Beans，Core，Context，Expression



版本的命名规则

若用X.Y.Z表示，Y偶数表示稳定，Y表示开发版本

0.01  1.0.0 2.6.32

* X 非负整数 表示主版本(Major)表示修改了参数，修改了方法，Api的时候，需要递增
* Y 非负整数 表示副版本   扩展了功能，并不影响api，需要递增
* Z 非负整数，修改了bug，递增

gradle 配置环境 gradle.org



#### 简化Spring的过程



![1555341785956](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1555341785956.png)



回顾一下，配置web.xml

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <servlet>
    <servlet-name>popMvc</servlet-name>
    <servlet-class>v1.servlet.PopDispatchServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:application.properties</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>popMvc</servlet-name>
    <url-pattern>/*</url-pattern>
  </servlet-mapping>

</web-app>
```

需要注意的是，放进去的Servlet一定要继承HttpServlet



```java
public class PopDispatchServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }

    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
    }
}
```

##### 关于表达式的解决

`Matcher matcher = Matchers.complie("\\[|\\]");`

|表示除了什么，都匹配

##### Spring中的转换器

手写spring中，我们为了将得到的参数由String，变成响应的类型

用到了我们自己的转换器，虽然有些简陋

```java
public Object convert(Class<?> type,String value){
    if(type == Integer.class){
        return Integer.valueOf(value);
    }else if(type == Double.class){
        //... todo
    }
   	return value;
}
```

但是在spring中存在属于他自己的转换器，`Converter` ，并且有诸多实现



#### 面试题

##### 问：Spring中的bean是否是线程安全的？

**答**：Spring中的Bean是否线程安全，与Spring并没有关系，他只是

帮你初始化好了，并且缓存到ioc容器中，本身对bean并没有什么处理

你的bean是否线程安全取决于Bean的本身，bean本身就是我们自己写的代码



**问**：Spring中的Bean是如何被回收的？

**答**：这取决于Bean的生命周期，他的生命周期如何，那么他就什么时候被回收

例如，Spring所管辖的Bean的生命周期有几种，singleton，prototype，session，request。。

对于singleton而言，他的生命周期是随着spring容器的消亡而消亡

而对于prototype(多例)而言，因为是new出来的，所以只要GC不可达的时候，将会被GC回收

其他的，Session，就是会话关闭，还有request请求结束，才会消亡。



#### Spring中的 ioc

IOC，和DI的理解

##### 对象和对象关系怎么表示？

Xml，properties，表示关系

**描述对象关系的文件存放在哪里**

classpath/filesystem/

**如何统一配置文件的标准**

BeanDefinition保存诸多来源不同的资源

**如何针对不同的资源进行解析**

`策略模式`

 BeanFactory 定义容器

BeanDefinition 存储bena信息，

BeanDefinitionReader 读取bean信息

定位->加载->注册

**从DispacthServlet开始**

BeanDefinitionParserDelegate

最终在LoadDefinition中完成了ioc容器的初始化

#### Spring中的DI	



##### 基于Annotation IOC容器初始化



**流程**

定位Bean扫描路径->读取元数据->解析

->注册Bean



`AnnotationConfigApllcationContext`

在实例化这个对象的时候，很多步骤都是为了判断，这个注解所包含的元信息(MetaData)，例如是否需要代理，是否是单例，是否有依赖等等东西。



##### 依赖注入

IOC容器

BeanFactroy getBean

getBean的实现 AbstractBeanFactory



* 实例化的策略
  * 默认实例化策略`SimpleInstantiateionStrateu`
* 存储实例的包装，这个实例存在一些配置，关联，如果每次都需要联系就很麻烦，但是如果将联系存储起来，就可以提高性能
* * BeanWrapper

###### 依赖注入发生的时间

* 用户第一次调用getBean方法
* 在配置文件中配置了<bean> lazy-init=false，那么在解析Bean的时候就会直接初始化，不会等到getBean的时候再初始化。



AbstractAutowireCapableBeanFactory设置

中

createBeanIntance  根据具体策略实例化Bean，但是返回的BeanWapper

populateBean完成正在依赖注入的方法



FactoryBean Spring内部实现的一种规范&开头作为beanName



Spring中的所有容器都是FactoryBean，也就是工厂Bean，也就是



#### Spring 中的 AOP

```xml
<!-- 注解驱动加上这句话 同时这句话，将会开启Cglib代理-->
	<!--<aop:aspectj-autoproxy proxy-target-class="true"/>-->

<bean id="xmlAspect" class="com.gupaoedu.vip.aop.aspect.XmlAspect"></bean>
	 <!--AOP配置 -->
	<aop:config>
		 <!--声明一个切面,并注入切面Bean,相当于@Aspect -->
		<aop:aspect ref="xmlAspect" >
			 <!--配置一个切入点,相当于@Pointcut -->
			<aop:pointcut expression="execution(* com.gupaoedu.vip.aop.service..*(..))" id="simplePointcut"/>
			 <!--配置通知,相当于@Before、@After、@AfterReturn、@Around、@AfterThrowing -->
			<aop:before pointcut-ref="simplePointcut" method="before"/>
			<aop:after pointcut-ref="simplePointcut" method="after"/>
			<aop:after-returning pointcut-ref="simplePointcut" method="afterReturn"/>
			<aop:after-throwing pointcut-ref="simplePointcut" method="afterThrow" throwing="ex"/>
			<aop:around pointcut-ref="simplePointcut"  method="around"/>
		</aop:aspect>
	</aop:config>
```

Java版本

```java
/**
 * XML版Aspect切面Bean(理解为TrsactionManager)
 * @author Tom
 */
public class XmlAspect {

	private final static Logger log = Logger.getLogger(XmlAspect.class);
	
	/*
	 * 配置前置通知,使用在方法aspect()上注册的切入点
	 * 同时接受JoinPoint切入点对象,可以没有该参数
	 */
	public void before(JoinPoint joinPoint){
//		System.out.println(joinPoint.getArgs()); //获取实参列表
//		System.out.println(joinPoint.getKind());	//连接点类型，如method-execution
//		System.out.println(joinPoint.getSignature()); //获取被调用的切点
//		System.out.println(joinPoint.getTarget());	//获取目标对象
//		System.out.println(joinPoint.getThis());	//获取this的值
		
		log.info("before " + joinPoint);
	}
	
	//配置后置通知,使用在方法aspect()上注册的切入点
	public void after(JoinPoint joinPoint){
		log.info("after " + joinPoint);
	}
	
	//配置环绕通知,使用在方法aspect()上注册的切入点
	public void around(JoinPoint joinPoint){
		long start = System.currentTimeMillis();
		try {
			((ProceedingJoinPoint) joinPoint).proceed();
			long end = System.currentTimeMillis();
			log.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms!");
		} catch (Throwable e) {
			long end = System.currentTimeMillis();
			log.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms with exception : " + e.getMessage());
		}
	}
	
	//配置后置返回通知,使用在方法aspect()上注册的切入点
	public void afterReturn(JoinPoint joinPoint){
		log.info("afterReturn " + joinPoint);
	}
	
	//配置抛出异常后通知,使用在方法aspect()上注册的切入点
	public void afterThrow(JoinPoint joinPoint, Exception ex){
		log.info("afterThrow " + joinPoint + "\t" + ex.getMessage());
	}
	
}
```

* 切面(Aspect)切面，规则，具有相同规则方法的集合体execution中的所匹配到的具体方法
* 切入点（Pointcut）需要代理的具体方法
* 通知（Advice）回调
* 目标对象(Target Object)被代理的对象
* AOP代理(Aop proxy)主要两种方法，JDK,CGLIB

//前置通知，后置通知等

##### 注解Aop

```java
/**
 * Annotation版Aspect切面Bean
 * @author Tom
 */
//声明这是一个组件
@Component
//声明这是一个切面Bean，AnnotaionAspect是一个面，由框架实现的
@Aspect
public class AnnotaionAspect {

	private final static Logger log = Logger.getLogger(AnnotaionAspect.class);
	
	//配置切入点,该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点
	//切点的集合，这个表达式所描述的是一个虚拟面（规则）
	//就是为了Annotation扫描时能够拿到注解中的内容
	@Pointcut("execution(* com.gupaoedu.vip.aop.service..*(..))")
	public void aspect(){}
	
	/*
	 * 配置前置通知,使用在方法aspect()上注册的切入点
	 * 同时接受JoinPoint切入点对象,可以没有该参数
	 */
	@Before("aspect()")
	public void before(JoinPoint joinPoint){
		log.info("before " + joinPoint);
	}
	
	//配置后置通知,使用在方法aspect()上注册的切入点
	@After("aspect()")
	public void after(JoinPoint joinPoint){
		log.info("after " + joinPoint);
	}
	
	//配置环绕通知,使用在方法aspect()上注册的切入点
	@Around("aspect()")
	public void around(JoinPoint joinPoint){
		long start = System.currentTimeMillis();
		try {
			((ProceedingJoinPoint) joinPoint).proceed();
			long end = System.currentTimeMillis();
			log.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms!");
		} catch (Throwable e) {
			long end = System.currentTimeMillis();
			log.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms with exception : " + e.getMessage());
		}
	}
	
	//配置后置返回通知,使用在方法aspect()上注册的切入点
	@AfterReturning("aspect()")
	public void afterReturn(JoinPoint joinPoint){
		log.info("afterReturn " + joinPoint);
	}
	
	//配置抛出异常后通知,使用在方法aspect()上注册的切入点
	@AfterThrowing(pointcut="aspect()", throwing="ex")
	public void afterThrow(JoinPoint joinPoint, Exception ex){
		log.info("afterThrow " + joinPoint + "\t" + ex.getMessage());
	}
	
}
```

##### 入口

`initializeBean方法`

创建代理对象

将BeanWarrper

![1555939539086](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1555939539086.png)

最后我们会在DefaultAopProxyFacoty中找到生成

Aop代理的工厂，AopProxy，即这个时候，才是创建Aop真正开始的时候

代理中的责任链

MethodBeforeAdviceInterceptor

Method*Intercept

关于这么多拦截器如何排序

因为有前置，和后置的关系

AbstractAdvisorAutoProxyCreator

findEliglbleAdvisors中

有个sortAdvisors

其实本质上，也就是实现了

Compare<T>接口，

#### IOC/DI/AOP的总结

IOC的一个终极目标是为了

将BeanDefinition保存在

DefaultListableBeanFactory

中的BeanDefinition的配置信息，所谓的定位（getResource），

加载(reader.loadBeanDefinition)，注册(RegisterBeanDefintion)

在Di过程中，入口在getbean开始

为了将bean包装成BeanWrapper

其中会包含许多，例如代理对象信息

自己的源信息，还有其他的东西，

但凡有关联的信息，都会被保存起来

最后保存在

factoryBeanuinstanceCache中

这是用来保存代理对象的，或者一些实例对象的缓存map

这算是IOC容器的最终样子

所在类为

AbstractAutowireCapableBeanFactory

不过在docreateBean方法中，spring做出了相应的优化，这种优化指的是

先会去缓存里去取，FactoryBeanRegisterSupport中的

Map<String,Object> factroyBeanObjectCache中,会存储例如单例之类的缓存

FacotryBeanInstanceCache

而在依赖注入的时候

从populateBean中开始，中间会根据依赖关系

在spring中，就是我所熟悉的字段的赋值之类的

有引用的类型，还以其它九大基本类型

当所有的问题解决ok后，使用BeanWrapperImpl的setValue方法

进行赋值。

**循环赋值的问题**

class A{	

​	B b;

}

class B{

A a;

}

//手写源码再说

**AOP**

从inializeBean

调用getProxy并且放到BeanWarpper中，也就是关联的Proxy信息



#### SpringMVC

DispatcherServlet->HandlerMapping->Controller->

ModelAndView->ViewResolver->View

更多细节，从DispatcherServlet中的`initStrategies`开始

initStrategies初始化阶段

调用方法为doService->doDispatch

##### SpringMVC中的九大组件



###### HanderMappings

根据url与controller，method构成一对一对应。

@RequestMapping

###### HanlderAdapters

适配器（亡羊补牢）具体去匹配对应参数的方法

###### HanlderExceptionResolvers

处理Hanlder中异常的处理

404、500，403。。。对应的页面，更加友好。

###### ViewResolvers

视图的解析器，freemark，最终都会被解释成HTML能够解释的页面进行解释。

###### RequestToViewNameTranslator

request中获得viewname，可能一个request中存在viewName

但是也存在我不设置viewName的情况，所以他的作用就出来了、

###### LocaleResolver

国际化的解析器，根据不同的语言环境来切换，这就是

local

###### ThemeResolver

主题切换器。

###### MultipartResolver

用来处理上传文件请求。

最终的一个实现类AbstractMultipartHttpServletRequest

存在

MultiValueMap<String,MultipartFile> multipartFile

这个键值对，String为具体的文件名。可以直接得到文件

比较方便。

###### FlashMapManager

管理Flash(闪存)Map

回顾一下淘宝订单的场景

当我们付款的时候，会重定向(redirect),

并同时显示你的付款的信息。

在JAVAEE规范中redirect后的数据要取到必须从url中去拿才可以

从url拿一个是url长度问题，还有一个是安全问题

那么，在spring中，可以将这个关键的东西

存入Request.setAttribute中，设置一个关键的key

接着从FlashMap中使用去取出这个key，从mode中取值

所以主要是用来参数中转的。



##### 初始化阶段

HttpServletBean->init()

initServletBean()->FrameworkServlet->initWebApplicationContext

DispatcherServlet->initStrategies

##### 调用阶段

DispatcherServlet->doService

doDispatch->getHandler

->AbstractHandlerMapping

preHandle,postHandle，前置拦截，后置拦截



#### 关于SpringMVC优化的几个建议

**1、Controller如果能保持单例，尽量使用单例**

**2、@RequestParam,@PathVariable**

@RequestMapping("/main/{id}")

public String xxx(@PathVariable("id") String id)

能写注解，尽量写

**3、Spring MVC 并没有对url和method的对应关系进行缓存，**

只是对url与controller的缓存，因为支持热部署，所以需要动态

去找。

建议：自己对url和method的关系进行缓存

写请求拦截。

```java

public class MyInterceptor implements HandlerInterceptor {
    private final static Logger logger = LoggerFactory.getLogger(MyInterceptor.class);
 
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("进入 preHandle 方法..." + request.getRequestURL().toString() + "," + request.getRequestURI());
        return true;
    }
 
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("进入 postHandle 方法..." + request.getRequestURL().toString() + "," + request.getRequestURI());
    }
 
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("进入 afterCompletion 方法..." + request.getRequestURL().toString() + "," + request.getRequestURI());
    }

```

配置

```xml
 <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <mvc:exclude-mapping path="/user/index"/>
            <bean class="com.springmvc.intercepter.MyInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>

```

