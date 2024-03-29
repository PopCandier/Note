#### Spring Boot 编程思想



#### 从创建一个Spring Boot项目开始。

start.spring.io 我们可以很轻松的构建一个spring-boot项目

下载完，我们直接导入pom.xml文件，Maven将会帮我们直接构建spring-boot所需要的依赖。

**如何打包**

我们cd到spring-boot的根目录，执行

```
mvn package
```

不过前提是你需要拥有这个插件。

```xml
<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
```

当构建完成后，将会存在一个.jar文件，你可以直接使用

`java -jar xxxx.jar`

去运行这个打包的文件。

这里有两个打包

`.jar`文件和`.jar.original`，前者比较大，因为他将依赖的包打包进来，后者只是代码部分，所以比较小，我们可以通过tree ..看到目录结构。

```c
D:\IDEAWORKSPACE\POP\TARGET\TEMP
├─BOOT-INF
│  ├─classes   //存放应用编译后的class文件
│  │  └─thinkingSpringBoot
│  │      └─Pop
│  └─lib  //存放依赖的jar包
├─META-INF //存放应用相关的元信息 MANIFEST.MF
│  └─maven
│      └─thinkingSpringBoot
│          └─Pop
└─org  //目录存在Spring Boot 相关的 class 文件
    └─springframework
        └─boot
            └─loader
                ├─archive
                ├─data
                ├─jar
                └─util
                
D:\IDEAWORKSPACE\POP\TARGET\TEMP2
├─META-INF
│  └─maven
│      └─thinkingSpringBoot
│          └─Pop
└─thinkingSpringBoot
    └─Pop

```

为什么使用java -jar 会就会启动？

我们从解包出来的文件看出，这并不是标准的文件目录

而官方文档规定，javar -jar 命令引导的具体启动类必须配置在MANIFEST.MF资源的Main-Class中

而同时，根据 “JAR文件规范” MANIFEST.MF资源必须放在MATA-INF目录下

```mf
Manifest-Version: 1.0
Implementation-Title: Pop
Implementation-Version: 0.0.1-SNAPSHOT
Start-Class: thinkingSpringBoot.Pop.PopApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.1.5.RELEASE
Created-By: Maven Archiver 3.4.0
Main-Class: org.springframework.boot.loader.JarLauncher
```

所以很显然，能够使得这个能够运行的，奥秘在

`org.springframework.boot.loader.JarLauncher`这里面。

当然，有能够启动jar当然也有能够启动 war

`org.springframework.boot.loader.WarLauncher`

当然我们如果希望查看这个的实现，需要添加额外的依赖。

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-loader</artifactId>
			<scope>provided</scope>
		</dependency>
```

然后我们可以看到这个类的全貌

```java
public class JarLauncher extends ExecutableArchiveLauncher {

	static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";

	static final String BOOT_INF_LIB = "BOOT-INF/lib/";

	public JarLauncher() {
	}

	protected JarLauncher(Archive archive) {
		super(archive);
	}

	@Override
	protected boolean isNestedArchive(Archive.Entry entry) {
		if (entry.isDirectory()) {//对于资源的定位
			return entry.getName().equals(BOOT_INF_CLASSES);
		}
		return entry.getName().startsWith(BOOT_INF_LIB);
	}

	public static void main(String[] args) throws Exception {
		new JarLauncher().launch(args);//<----从这里开始
	}

}
```

很明显，这个是启动的关键。

```java
public abstract class Launcher {

	/**
	 * Launch the application. This method is the initial entry point that should be
	 * called by a subclass {@code public static void main(String[] args)} method.
	 * @param args the incoming arguments
	 * @throws Exception if the application fails to launch
	 */
	protected void launch(String[] args) throws Exception {
        //这一句话表示注册协议
		JarFile.registerUrlProtocolHandler();
		ClassLoader classLoader = createClassLoader(getClassPathArchives());
		launch(args, getMainClass(), classLoader);
	}
	
	//.....	
｝
```

JDK中，默认支持file、HTTP、jar等协议并且这些实现都在sun.net.ww.protocol类中，名字都为Handler,因为这是规定。

sun.net.www.protocel.${protocol}.Handler，中间表示协议名，file、jar、http、https、ftp并且以上都为java.net.URLStreamHandler实现

JarFile.registerUrlProtocolHandler();执行的时候，将会把spring-boot自己实现的Hanlder追加到java的系统属性，java.protocol.handler类中

问题在于jdk已经实现了自己的jar方法，为什么spring-boot还要有自己选择覆盖呢，这点在P49的set方法上有说明。

大概地意思是spring-boot中地jar文件是一个独立地应用，与其他地jar不一样，他除了自己地jar文件之外，还有依赖其余地jar文件，所以jdk那套不适用于他，当执行java jar 命令地时候，内部地sun.net.www.protocol.jar.Handler无法被当做class path，所以需要被覆盖。

```java
ClassLoader classLoader = createClassLoader(getClassPathArchives());
```

这一步主要是为了区别目前是通过jar驱动还是war驱动，前者将会使用sprint boot fat jar后者用于解压其内容。接着调用launcher()方法。

```java
launch(args, getMainClass(), classLoader);
```

```java
protected void launch(String[] args, String mainClass, ClassLoader classLoader)
			throws Exception {
		Thread.currentThread().setContextClassLoader(classLoader);
		createMainMethodRunner(mainClass, args, classLoader).run();
	}
```

这一步就是从/META-INF/MAINFEST.MF资源周年读取start-class文件，并调用main方法启动。

**探究WarLauncher和JarLauncher的区别**、

其实两者区别还是很小地，一个明显地差别就是，常量地设定

首先是JarLauncher

```java
static final String BOOT_INF_CLASSES = "BOOT-INF/classes/";

	static final String BOOT_INF_LIB = "BOOT-INF/lib/";

```

其次是WarLauncher

```java
private static final String WEB_INF = "WEB-INF/";

	private static final String WEB_INF_CLASSES = WEB_INF + "classes/";

	private static final String WEB_INF_LIB = WEB_INF + "lib/";

	private static final String WEB_INF_LIB_PROVIDED = WEB_INF + "lib-provided/";
```

war得定义很类似传统地serlvet容器地定义 web-inf/classes 之类地

我们再打包一次，看看区别

```java
<groupId>thinkingSpringBoot</groupId>
	<artifactId>Pop</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
```

执行 mvn clean package

```java
├─META-INF
│  └─maven
│      └─thinkingSpringBoot
│          └─Pop
├─org
│  └─springframework
│      └─boot
│          └─loader
│              ├─archive
│              ├─data
│              ├─jar
│              └─util
└─WEB-INF
    ├─classes
    │  └─thinkingSpringBoot
    │      └─Pop
    ├─lib
    └─lib-provided
```

其实

```java
private static final String WEB_INF_LIB_PROVIDED = WEB_INF + "lib-provided/";
```

是spring-boot地war湿度有地，

这个目录只有一个

spring-boot-loader-x.x.x.RELEASE.jar

文件,不过更具serlvet规范,WBE-INF/lib-provided中地jar不会被servlet容器读取，会被忽略。

这样设计好处是，war是一种兼容地措施，既可以被warLauncher启动，又可以兼容servlet容器环境。换言之，warLauncher与JarLauncher并没本质区别，所以建议Spring-boot应用使用非传统web部署地时候，尽可能使用jar地方式。



#### 理解固化地Maven

此固化仅仅限制于spring-boot应用。

当然我们通过单击继承地方式，会有些限制

```xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```

可以通过这种依赖地方式替换

```xml
<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>2.1.5.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

不过，如果你这样打包会出问题，因为他会认为web.xml是必选，所以在你设置打包方式为

war地情况下，它会依赖maven-war-plugin插件，而web-inf/web.xml又是必须地，所以会报错。我们可以通过以下方式修改。

```java
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.2:war (default-war) on project Pop: Error assembling WAR:
 webxml attribute is required (or pre-existing WEB-INF/web.xml if executing in update mode) -> [Help 1]

```

```xml
 <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-war-plugin</artifactId>
         <configuration>
            <failOnMissingWebXml>false</failOnMissingWebXml>
         </configuration>
      </plugin>
```

##### spring-boot-starter-parent与spring-boot-dependencies的区别

首先从pom文件地继承方式上来看，dependencies是parent地父类，换言之，我们选择直接继承dependencies都是可以的

当然。这个插件地版本，我们可以进到dependencies中，看看对应地版本具体是什么样的，为了兼容。

不过当我们运行java -jar命令地时候

```
D:\IdeaWorkSpace\Pop\target>java -jar Pop-0.0.1-SNAPSHOT.war
Pop-0.0.1-SNAPSHOT.war中没有主清单属性
```

会有这样地提示，不过结合前面地理论，首先war肯定是执行地了地，但是真正启动地确实依赖spring-boot-maven-plugin插件，所有java -jar对应地插件没有执行，所以我们需要对插件指定版本。不过这个时候插件依旧无法执行，所以我们还需要额外地配置。

```xml
<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<version>2.1.5.RELEASE</version>
			<executions>
				<execution>
					<goals>
						<goal>
							repackage
						</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
```

##### 总结

* 关于maven-war-plugin地差异，是因为版本地差异，2.2版本会出现web.xml必填写地选项，而3.1调整了其默认地必填行为
* 另外，在单独引用spring-boot-maven-plugin地时候，需要配置<goal>元素，否则不会将spring-boot地引导依赖重新打包（repackage）进jar，所以无法进行引导当前应用。、
* 最后一点是，一般来说我们不用使用spring-boot-dependencies来作为maven项目地parent，尽管spring-boot-starter-parent与spring-boot-dependencies，两者是继承关系。



```
2019-06-15 14:51:04.681  INFO 11348 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080
 (http) with context path ''
```

我们使用

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

就将tomcat容器引导加载我们地应用，明明都不要安装，这是为什么

