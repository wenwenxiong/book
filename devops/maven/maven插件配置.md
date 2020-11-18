maven优势于Ant的原因有很大一部分来自于maven减少了手动的配置，这也导致了他结构的特殊。
maven遵循约定大于配置，默认目录为：

默认值 | 类型
------|-----
目录src/main/java| java源码目录
目录src/main/resources| 资源文件目录
目录src/test/java | 测试java源码目录
目录src/test/resources | 测试资源文件目录
目录target |打包输出目录
目录target/classes|编译输出目录
目录target/test-classes|测试编译输出目录
目录target/site|项目site输出目录
目录src/main/webapp|web应用文件目录（当打包为war时），如WEB-INF/web.xml
jar|默认打包格式
*Test.java|Maven只会自动运行符合该命名规则的测试类
%user_home%/.m2|Maven默认的本地仓库目录位置
中央仓库|Maven默认使用远程中央仓库：http://repo1.maven.org/maven2
1.3|Maven Compiler插件默认以1.3编译，因此需要额外配置支持1.5

但它也支持对约定目录的自定义更改，通常不建议更改maven默认目录，这将导致maven不能正常处理对应的逻辑，例如：
```
<build>  
    <sourceDirectory>src/java</sourceDirectory>  
    <testSourceDirectory>src/test</testSourceDirectory>  
    <outputDirectory>output/classes</outputDirectory>  
    <testOutputDirectory>output/test-classes</testOutputDirectory>  
    <directory>target/jar</directory>  
</build>
```
上诉代码将java源代码存放在了src/java中，test源代码存放在了src/test中，将编译后的java文件放在了out/classes中，编译后的测试文件放在了output/test-classes中，运行后的打包文件存放在target/jar中。

通常情况下我们会修改webapp目录为WebContent目录，需做如下配置：
```
<build>
	<finalName>ThesisManage</finalName>
	<!-- 自定义maven结构目录 -->
	<sourceDirectory>src/main/java</sourceDirectory>
	<resources>
		<resource>
			<directory>src/main/resources</directory>
		</resource>
	</resources>
	<testResources>
		<testResource>
			<directory>src/test/resources</directory>
		</testResource>
	</testResources>
		
	<plugins>
		<!-- 定义编译版本为1.7，字符编码为utf8 -->
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>2.0.2</version>
			<configuration>
				<source>1.7</source>
				<target>1.7</target>
				<encoding>UTF-8</encoding>
			</configuration>
		</plugin>

		<!-- 修改webapp目录为WebContent -->
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-war-plugin</artifactId>
			<configuration>
				<!-- 设置WebContent目录为Web目录 -->
				<webappDirectory>${basedir}/WebContent</webappDirectory>
				<warSourceDirectory>${basedir}/WebContent</warSourceDirectory>
			</configuration>
		</plugin>
	</plugins>
</build>
```
其中```<webappDirectory>```：产生war前，用于存放war的目录
其次```<warSourceDirectory>```:将web项目做成eclipse下的WTP类型，即是用WebContent替换webapp

另外：```<packagingExcludes>、<warSourceExcludes>```都可以用来忽略打war时的部分包的，如：
```
<packagingExcludes>
	WEB-INF/lib/spring-2.5.**.jar,
	WEB-INF/lib/jersey-servlet-1.17.1.jar
</packagingExcludes>
```
上面表示在打war包时忽略spring-2.5的包和jersey-servlet-1.17.1.jar            

###maven配置java编译插件
java编译插件有两个目标
* compile-编译java源码
* testCompile-编译java test代码

maven编译的插件pom.xml配置如下
```
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.7.0</version>
    <configuration>
        ...
    </configuration>
</plugin>
```
在```<configuration>```与```</configuration>```之间需要配置以下
* 编译和运行java程序的jdk版本，maven默认使用```java5```。
以下配置java8。
```
<configuration>
    <source>1.8</source>
    <target>1.8</target>
    <-- other customizations -->
</configuration>
```
* 配置编译参数传递给javac编译器。
以下配置编译参数```-Xlint:unchecked```
```
<configuration>
    <!-- other configuration -->
    <compilerArgs>
        <arg>-Xlint:unchecked</arg>
    </compilerArgs>
</configuration>
```

###maven打war包插件
名称为```maven-war-plugin```。
下面是常用的配置。
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <warName>${project.artifactId}</warName>
        <warSourceDirectory>src/main/webapp</warSourceDirectory>
        <webXml>src/main/webapp/WEB-INF/web.xml</webXml>
    </configuration>
</plugin>
```
war的源文件目录配置为```<warSourceDirectory>```指定。
war包的名称为```<warName>```指定
war包的```web.xml```文件为```<webXml>```指定