### MyBatis笔记

数据库的框架

在Spring JDBC中存在template

方法的封装 jdbcTemplate

资源管理 dataSource

结果集的处理 rowMapper



下划线转驼峰方法-》mybaits方法中

SqlSessionFactoryBuilder：主要用于生产

SqlSessionFactory，所以这个是一个方法级别

SqlSessionFactory 一直产生对象，单例

SqlSession 存在一次事务，一次会话 request

Mapper 依旧方法级别



别名的设定，也可以设置类型包的扫描
typeAliasRegistry内置的方言初始化



mybatis的类型自定义转换

TypeHandlerRegistry，也可以实现相应接口

**问题抽取**

如果将一个BLOG转化成一个对象

文本中表示的大文本设置成对象

可能是存储json转换成对象，在mybatis实现中

是bytes类型，所以，我们需要自定义转换器。

mysql 5.7 版本，支持json对象存储



专门生成数据库到对象的映射的核心接口

ObjectFactory

默认实现DefaultObjectFactory

每次取出数据的时候，某个电商项目

创建电商的对象，订阅数*3

继承ObjectFactory同时重写create方法

也要注册到全局文件





#### 一些标签的使用

if

choose

trim

```xml
<trim prefix="(" suffix=")" suffixOverrides=",">
    <if test="empId != null">
    	emp_id,
    </if>
     <if test="empName != null">
    	emp_name,
    </if>
</trim>
```

关于插入的时候，你不确定哪些字段是存在的情况下

你可以使用trim，可以通过suffixOverrides的指定，消除掉这个符号

例如，如果empId存在，那么就会拼接上，然后由于是最后一个

最后一个带了`，`是会有语法错误，但是这个trim可以帮你消除掉这个逗号

语句将会通过

foreach

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>

<delete id="xx" parameterType="java.util.List">
    delete from _emp where emp_id in
    <foreach collection="list" item="item" 
             open="(" separator="," close=")">
        #{item.empId,jdbcType=VARCHAR}
    </foreach>
</delete>

```

**批量操作**

java代码中的for比较消耗性能

mybatis foreach的动态更新

也就是会拼接sql语句

```sql
insert into (id,name,addr) values
(?,?,?),(?,?,?),(?,?,?),(?,?,?),(?,?,?)
```

但其实，如果插入的数据过多，也是会有问题，拼接的语句过长

所以，例如mysql的语句限制为4M，超出会报错

所以mybatis里有一个叫做 batch

在官网setting中

有一个属性叫做`defaultExecutorType`

有三种属性对应，默认是simple

simple->statement

reuse->preparstatment

batch->在preparstament积累起来，然后一起发送给数据库

```java
 /**
     * 原生JDBC的批量操作方式 ps.addBatch()
     * @throws IOException
     */
    @Test
    public void testJdbcBatch() throws IOException {
        Connection conn = null;
        PreparedStatement ps = null;

        try {
            // 注册 JDBC 驱动
            Class.forName("com.mysql.jdbc.Driver");

            // 打开连接
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/gp-mybatis?useUnicode=true&characterEncoding=utf-8&rewriteBatchedStatements=true", "root", "123456");
            ps = conn.prepareStatement(
                    "INSERT into blog values (?, ?, ?)");

            for (int i = 1000; i < 101000; i++) {
                Blog blog = new Blog();
                ps.setInt(1, i);
                ps.setString(2, String.valueOf(i));
                ps.setInt(3, 1001);
                ps.addBatch();//<---批量
            }

            ps.executeBatch();
            // conn.commit();
            ps.close();
            conn.close();
        } catch (SQLException se) {
            se.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (ps != null) ps.close();
            } catch (SQLException se2) {
            }
            try {
                if (conn != null) conn.close();
            } catch (SQLException se) {
                se.printStackTrace();
            }
        }
    }
```



**嵌套结果**

```xml
<!-- 根据文章查询作者，一对一查询的结果，嵌套查询 -->
    <resultMap id="BlogWithAuthorResultMap" type="com.gupaoedu.domain.associate.BlogAndAuthor">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <!-- 联合查询，将author的属性映射到ResultMap -->
        <association property="author" javaType="com.gupaoedu.domain.Author">
            <id column="author_id" property="authorId"/>
            <result column="author_name" property="authorName"/>
        </association>
    </resultMap>


<!-- 根据文章查询作者，一对一，嵌套结果，无N+1问题 -->
    <select id="selectBlogWithAuthorResult" resultMap="BlogWithAuthorResultMap" >
        select b.bid, b.name, b.author_id, a.author_id , a.author_name
        from blog b
        left join author a
        on b.author_id=a.author_id
        where b.bid = #{bid, jdbcType=INTEGER}
    </select>
```



**嵌套结果**

```xml
<!-- 另一种联合查询(一对一)的实现，但是这种方式有“N+1”的问题 -->
    <resultMap id="BlogWithAuthorQueryMap" type="com.gupaoedu.domain.associate.BlogAndAuthor">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <association property="author" javaType="com.gupaoedu.domain.Author"
                     column="author_id" select="selectAuthor"/> <!-- selectAuthor 定义在下面-->
    </resultMap>

 <!-- 嵌套查询 -->
    <select id="selectAuthor" parameterType="int" resultType="com.gupaoedu.domain.Author">
        select author_id authorId, author_name authorName
        from author where author_id = #{authorId}
    </select>
```

定义了相应的结果集去查询，就是select中配置的语句

而这个N+1问题的产生，是因为查询多少条，就会多少个selectAuthor一起调用，为了避免这个情况，我们可以在配置文件中使用

```xml
 <!-- 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。默认 false  -->
        <setting name="lazyLoadingEnabled" value="true"/>
```

这样只会在特定方法，例如getAuthor的时候才会调用



翻页（分页）

逻辑翻页：假翻页，加载到内存再筛选 rowBundns

物理翻页：语句开始翻页

**N+1问题**





Mybatis是否支持Mapper的继承

如果你的字段发生了改变，不得不改映射文件。

字段表发生了变化

解决思路：

继承

通用Mapper



一些进一步封装的MyBatis的插件

tk mybatis

MyBaits-Plus



### 这节课 Mybatis的介绍

* 解析配置文件 mybatis-config.xml  mapper.xml
* 创建工厂类 sqlsessinFactory
* 创建会话  sqlSession
* 会话操作数据库



#### 缓存

一级缓存的工作位置与维护对象

Session 级别的

SqlSession->Executor(Local Cache)->databsae

同一个Session，将会享有一级缓存，测试一级缓存的时候

需要关闭二级缓存。

那么什么时候一级缓存会清空呢，当你执行了更新，删除操作的时候

一级缓存就会失效。

那么为什么我们执行增删改操作的时候，会清空缓存呢

主要是因为在

update标签或者insert 标签上，有一个隐藏的属性

```xml
<update id="updateByPrimaryKey" parameterType="blog" flushCache="true">
        update blog
        set name = #{name,jdbcType=VARCHAR}
        where bid = #{bid,jdbcType=INTEGER}
    </update>

```

当然，这个语句默认是false，flushCache，但是在Select中，这个是false的，因此Select将不会刷新缓存



二级缓存

作用于namesapce

在nameSapce中，加上<cache/>,开启二级缓存，同时在全局文件中，也要配置EnableCache =true

```xml
<cache type="org.apache.ibatis.cache.impl.PerpetualCache" 也是默认的实现
               size="1024"  		可以存放多少对象
               eviction="LRU"		当size满了后，采取什么算法，这个是最少使用 least recenty use
       								除此之外还有 fifo weak soft
               flushInterval="120000"  缓存刷新时间
               readOnly="false"/> 
		<!--是否只读，
		如果是false，说明可copy 序列化，
		所以说，相对应的namesapce下的对象必须实现序列化
		这种更安全
		true的情况下，返回的是同一个对象，速度更快
-->
```

XMLConfigBuilder

一些配置的解析和默认的值设定



1，解析了什么文件 mybatis-config.xml  mapper.xml

2，怎么解析的	通过XMLConfigBuilder每个标签解析

存放到相应的注册器中

3，产生了什么对象	DefaultSqlSessionFactory

4，结果存放在了哪里  Configuration



在openSession的時候

会根据配置写初始化执行器

BatchExecutor  对preparstatment的addBatch的封装

支持批量插入，更新，删除，执行的时候execuBatch

ReuseExecutor 对statement重用，会用map缓存起来

SimpleExecutor 普通statement封装，用完就关

这三种执行器的区别，update方法



##### 真正匹配方法到sql语句的关键部分

DefaultSqlSession

selectList

MappedStatement ms = configurationg.getMappedStatement(stat,emt);





Mybatis可以被拦截的四大对象

parameterHandler

Statementhanlder

ResultSetHandler

Excutor



用来存放所有的配置信息的类

Configuration

映射的关系，都会存放到Configuration

DefaultSqlSession->Executor

StatementHandler

ParameterHandler

ResultSetHanlder

Mapper接口如何找到映射器sql

->MapperProxy



解析Mapper映射器，入参，返回结构

MappedStatement(paramentType,returnType之类的)



### 掌握myBatis的插件原理，和定义插件编写

### 如何集成，集成的好处



不修改原有代码，怎么改变和增强对象的行为？

插件的拦截链怎么形成？如何做到层层拦截？

责任链

**有哪些对象允许代理？什么都会都会拦截吗？**

并不，只有四大天王才可以

官网插件拥有有什么方法，才会操作

**怎么创建代理**



**如何创建一个插件**

目标类需要实现Interceptor接口

并且完成注解

```java
/*
 * The MIT License (MIT)
 *
 * Copyright (c) 2014-2017 abel533@gmail.com
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 */

package com.github.pagehelper;

import com.github.pagehelper.cache.Cache;
import com.github.pagehelper.cache.CacheFactory;
import com.github.pagehelper.util.MSUtils;
import com.github.pagehelper.util.StringUtil;
import org.apache.ibatis.cache.CacheKey;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;

import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Properties;

/**
 * Mybatis - 通用分页拦截器<br/>
 * 项目地址 : http://git.oschina.net/free/Mybatis_PageHelper
 *
 * @author liuzh/abel533/isea533
 * @version 5.0.0
 */
@SuppressWarnings({"rawtypes", "unchecked"})
@Intercepts(
    {
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
    }
)
public class PageInterceptor implements Interceptor {
    //缓存count查询的ms
    protected Cache<CacheKey, MappedStatement> msCountMap = null;
    private Dialect dialect;
    private String default_dialect_class = "com.github.pagehelper.PageHelper";
    private Field additionalParametersField;

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        try {
            Object[] args = invocation.getArgs();
            MappedStatement ms = (MappedStatement) args[0];
            Object parameter = args[1];
            RowBounds rowBounds = (RowBounds) args[2];
            ResultHandler resultHandler = (ResultHandler) args[3];
            Executor executor = (Executor) invocation.getTarget();
            CacheKey cacheKey;
            BoundSql boundSql;
            //由于逻辑关系，只会进入一次
            if(args.length == 4){
                //4 个参数时
                boundSql = ms.getBoundSql(parameter);
                cacheKey = executor.createCacheKey(ms, parameter, rowBounds, boundSql);
            } else {
                //6 个参数时
                cacheKey = (CacheKey) args[4];
                boundSql = (BoundSql) args[5];
            }
            List resultList;
            //调用方法判断是否需要进行分页，如果不需要，直接返回结果
            if (!dialect.skip(ms, parameter, rowBounds)) {
                //反射获取动态参数
                Map<String, Object> additionalParameters = (Map<String, Object>) additionalParametersField.get(boundSql);
                //判断是否需要进行 count 查询
                if (dialect.beforeCount(ms, parameter, rowBounds)) {
                    //创建 count 查询的缓存 key
                    CacheKey countKey = executor.createCacheKey(ms, parameter, RowBounds.DEFAULT, boundSql);
                    countKey.update("_Count");
                    MappedStatement countMs = msCountMap.get(countKey);
                    if (countMs == null) {
                        //根据当前的 ms 创建一个返回值为 Long 类型的 ms
                        countMs = MSUtils.newCountMappedStatement(ms);
                        msCountMap.put(countKey, countMs);
                    }
                    //调用方言获取 count sql
                    String countSql = dialect.getCountSql(ms, boundSql, parameter, rowBounds, countKey);
                    BoundSql countBoundSql = new BoundSql(ms.getConfiguration(), countSql, boundSql.getParameterMappings(), parameter);
                    //当使用动态 SQL 时，可能会产生临时的参数，这些参数需要手动设置到新的 BoundSql 中
                    for (String key : additionalParameters.keySet()) {
                        countBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
                    }
                    //执行 count 查询
                    Object countResultList = executor.query(countMs, parameter, RowBounds.DEFAULT, resultHandler, countKey, countBoundSql);
                    Long count = (Long) ((List) countResultList).get(0);
                    //处理查询总数
                    //返回 true 时继续分页查询，false 时直接返回
                    if (!dialect.afterCount(count, parameter, rowBounds)) {
                        //当查询总数为 0 时，直接返回空的结果
                        return dialect.afterPage(new ArrayList(), parameter, rowBounds);
                    }
                }
                //判断是否需要进行分页查询
                if (dialect.beforePage(ms, parameter, rowBounds)) {
                    //生成分页的缓存 key
                    CacheKey pageKey = cacheKey;
                    //处理参数对象
                    parameter = dialect.processParameterObject(ms, parameter, boundSql, pageKey);
                    //调用方言获取分页 sql
                    String pageSql = dialect.getPageSql(ms, boundSql, parameter, rowBounds, pageKey);
                    BoundSql pageBoundSql = new BoundSql(ms.getConfiguration(), pageSql, boundSql.getParameterMappings(), parameter);
                    //设置动态参数
                    for (String key : additionalParameters.keySet()) {
                        pageBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
                    }
                    //执行分页查询
                    resultList = executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, pageKey, pageBoundSql);
                } else {
                    //不执行分页的情况下，也不执行内存分页
                    resultList = executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, cacheKey, boundSql);
                }
            } else {
                //rowBounds用参数值，不使用分页插件处理时，仍然支持默认的内存分页
                resultList = executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
            }
            return dialect.afterPage(resultList, parameter, rowBounds);
        } finally {
            dialect.afterAll();
        }
    }
//......

}

```

Mybatis的插件实现原理是什么

动态代理和责任链

1，四大对象什么时候被代理，也就是：代理对象是什么时候创建的

openSession的时候，获取会话的时候，就会创建。



实现拦截器的时候必须实现的四个方法

intercept：真正被拦截的方法

plugin，mybatis将会调用插件的这个方法，将你所自定义的插件

放到Configuation中的interceptChain中去，mybaits提供了

默认的包装实现，我们只要调用就可以了

return Plugin.wrap(target,this);



插件的应用场景

从PageHelper讲起

```java
PageHelper.startPage(1,20);
List<Emp;oyee> emps = employeeService.getAll();
```

为什么只是设置了在了前面，就会拥有分页的功能

主要是因为在底层使用了ThreadLocal<Page>，它将会让每个线程

拥有自己的副本来保存值，这样，就可以在底部按照不同方言

去拼接参数



mybatis插件的存在意义

由于Invocation中存储了参数args[] method 还有target目标代理对象

所以，我们可以使用我们自己的注解，来设置下一个语句会进入哪个库或者表

1，水平的分表

2，权限的控制 不同用户角色，给予自动数据过滤

3，数据库比较敏感字段，例如身份证

3600000000-》36000******

完成自己的logging日志，需要获得时间，并且

完成参数的填充，输出完整的查询对戏那个

RowBounds 来转换成，由逻辑翻页抓成物理翻页



### 整合Spring

1，管理对象

2，通过一个Template封装方法

MyBatis->mybatis-spring<-spring

中间桥梁



1，SqlSessionFactory是什么时候创建

2,SqlSession怎么获取的，为什么不用他来getMapper

3，为什么@Autowired注入一个Mapper接口，可以直接使用？

在IOC的容器里面我们注册的是什么？注册的时候发生了什么事

```xml
 <!--mybatis 和Spring整合 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybatis-spring.version}</version>
        </dependency>
```

spring.xml中

```xml
<!-- 在Spring启动时创建 sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="classpath:mybatis-config.xml"></property>
        <property name="mapperLocations" value="classpath:mapper/*.xml"></property>
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

mybatis-spring 实现了spring中的接口



```xml
<!--配置扫描器，将mybatis的接口实现加入到  IOC容器中  -->
<!--
    <mybatis-spring:scan #base-package="com.gupaoedu.crud.dao"/>
-->
    <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.gupaoedu.crud.dao"/>
    </bean>
```



那么spring中在@Autowired中发生了什么

解析，配置文件，创建爱你



与spring整合后，为什么不能使用默认的SqlSession

的默认实现，DefaultSqlSession

因为，DefaultSqlSession作为一个单例存在，在多线程情况下

会存在线程安全问题

所以spring中提供了线程安全的实现sqlSessionTemplate

如果你不希望通过注解的方式，来获得SqlSessionTemplate

可以继承SqlSessionDaoSupport有一个getSqlSessio()

的方法，来获得这个SqlSessionTemplate

这个类似Hibernate中的HibernateDaoSupport



扫描是什么时候扫描的，会调用spring的BeanDefinitionRe

注册时候，使用的是MapperFactoryBean

在注册的时候会直接替换掉，也就是

在你不想创建实现类的时候，并且还要注入接口的时候

就是替换原本应该注入进去的接口，变成了实现了SqlSessionFactory

中的MapperFactoryBean。



一个线程，都有自己的持有器，一个入DefaultSqlSession



#### 存在于MyBatis中的体系结构与工作原理

工厂:SqlSessionFactory

单例：SqlSessionFactory Configuration

建造者: SqlSessionFactoryBuilder

装饰者：CachingExector simple resue batch

LRUCahe FifoCache PerpectualCache

代理 Spring 集成 Mybatis SqlSessionInterceptor

Plugin 通过plugin，延迟加载 ProxyFactory

Log：日志模块

ConnectionLogger  TransactionLogger

PooledConnection：连接池中的管理

适配器 具体的日志实现 sl4j  适配

模版方法：Executor，Basecutor



#### 手写MyBatis的框架

-》为什么要手写

简化jdbc操作

-》设计中的需要思考的问题

应用层的api

核心对象

1，配置类

2，执行器

3，应用层的API，按照

4，代理对象

id->方法名称

namespace 接口类型

statment 真正的语句



操作流程

看老师的画的示意图



先从设计图开始



已经完成基本架构



1、支持参数的预编译

2、结果集的自动处理

3、对Excutor的职责细化



功能增强：

1、支持注解的方式配置Sql

2，查询实现缓存

3，实现插件