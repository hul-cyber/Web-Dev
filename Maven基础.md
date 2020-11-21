# Maven介绍

在了解Maven之前，我们先来看看一个Java项目需要的东西。首先，我们需要确定引入哪些依赖包。例如，如果我们需要用到`commons logging`，我们就必须把`commons logging`的jar包放入`classpath`。如果我们还需要`log4j`，就需要把`log4j`相关的jar包都放到`classpath`中。这些就是依赖包的管理。
其次，我们要确定项目的目录结构。例如，`src`目录存放`Java`源码，`resources`目录存放配置文件，`bin`目录存放编译生成的`.class`文件。

此外，我们还需要配置环境，例如JDK的版本，编译打包的流程，当前代码的版本号。

最后，除了使用Eclipse这样的IDE进行编译外，我们还必须能通过命令行工具进行编译，才能够让项目在一个独立的服务器上编译、测试、部署。

这些工作难度不大，但是非常琐碎且耗时。如果每一个项目都自己搞一套配置，肯定会一团糟。我们需要的是一个标准化的Java项目管理和构建工具。

Maven就是是专门为Java项目打造的管理和构建工具，它的主要功能有：
1. 提供了一套标准化的项目结构；
2. 提供了一套标准化的构建流程（编译，测试，打包，发布……）；
3. 提供了一套依赖管理机制。

## Maven的项目结构
一个使用Maven管理的普通的Java项目，它的目录结构默认如下：
```
a-maven-project
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       ├── java
│       └── resources
└── target
```
项目的根目录`a-maven-project`是项目名，它有一个项目描述文件`pom.xml`，存放Java源码的目录是`src/main/java`，存放资源文件的目录是`src/main/resources`，存放测试源码的目录是`src/test/java`，存放测试资源的目录是`src/test/resources`，最后，所有编译、打包生成的文件都放在target目录里。这些就是一个Maven项目的标准目录结构。
所有的目录结构都是约定好的标准结构，我们千万不要随意修改目录结构。使用标准结构不需要做任何配置，Maven就可以正常使用。
我们再来看最关键的一个项目描述文件`pom.xml`，它的内容长得像下面：
```xml
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>
	<artifactId>hello</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<properties>
        ...
	</properties>
	<dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
	</dependencies>
</project>
```
其中，`groupId`类似于`Java`的包名，通常是公司或组织名称，`artifactId`类似于`Java`的类名，通常是项目名称，再加上`version`，一个Maven工程就是由`groupId`，`artifactId`和`version`作为唯一标识。我们在引用其他第三方库的时候，也是通过这3个变量确定。例如，依赖`commons-logging`：
```xml
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
```
使用<dependency>声明一个依赖后，Maven就会自动下载这个依赖包并把它放到classpath中。

## 小结
Maven是一个Java项目的管理和构建工具：
1. Maven使用pom.xml定义项目内容，并使用预设的目录结构；
2. 在Maven中声明一个依赖项可以自动下载并导入classpath；
3. Maven使用groupId，artifactId和version唯一定位一个依赖。

# 依赖管理
如果我们的项目依赖第三方的jar包，例如commons logging，那么问题来了：commons logging发布的jar包在哪下载？

如果我们还希望依赖log4j，那么使用log4j需要哪些jar包？

类似的依赖还包括：JUnit，JavaMail，MySQL驱动等等，一个可行的方法是通过搜索引擎搜索到项目的官网，然后手动下载zip包，解压，放入classpath。但是，这个过程非常繁琐。

Maven解决了依赖管理问题。例如，我们的项目依赖`abc`这个jar包，而`abc`又依赖`xyz`这个jar包：
```
┌──────────────┐
│Sample Project│
└──────────────┘
        │
        ▼
┌──────────────┐
│     abc      │
└──────────────┘
        │
        ▼
┌──────────────┐
│     xyz      │
└──────────────┘
```
当我们声明了`abc`的依赖时，Maven自动把`abc`和`xyz`都加入了我们的项目依赖，不需要我们自己去研究`abc`是否需要依赖`xyz`。
因此，Maven的第一个作用就是解决依赖管理。我们声明了自己的项目需要`abc`，Maven会自动导入`abc`的jar包，再判断出`abc`需要`xyz`，又会自动导入`xyz`的jar包，这样，最终我们的项目会依赖`abc`和`xyz`两个jar包。
复杂依赖演示:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.4.2.RELEASE</version>
</dependency>
```
当我们声明一个`spring-boot-starter-web`依赖时，Maven会自动解析并判断最终需要大概二三十个其他依赖：
```
spring-boot-starter-web
  spring-boot-starter
    spring-boot
    sprint-boot-autoconfigure
    spring-boot-starter-logging
      logback-classic
        logback-core
        slf4j-api
      jcl-over-slf4j
        slf4j-api
      jul-to-slf4j
        slf4j-api
      log4j-over-slf4j
        slf4j-api
    spring-core
    snakeyaml
  spring-boot-starter-tomcat
    tomcat-embed-core
    tomcat-embed-el
    tomcat-embed-websocket
      tomcat-embed-core
  jackson-databind
  ...
```
## 依赖关系
Maven定义了几种依赖关系，分别是`compile`、`test`、`runtime`和`provided`：
```
scope: compile
说明: 编译时需要用到该jar包（默认）
示例: commons-logging
```
```
scope: test
说明: 编译Test时需要用到该jar包
示例: junit
```
```
scope: runtime
说明: 编译时不需要，但运行时需要用到
示例: mysql
```
```
scope: provided
说明: 编译时需要用到，但运行时由JDK或某个服务器提供
示例: servlet-api
```
其中，默认的`compile`是最常用的，Maven会把这种类型的依赖直接放入classpath。
`test`依赖表示仅在测试时使用，正常运行时并不需要。最常用的`test`依赖就是JUnit：
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.3.2</version>
    <scope>test</scope>
</dependency>
```
`runtime`依赖表示编译时不需要，但运行时需要。最典型的`runtime`依赖是JDBC驱动，例如MySQL驱动：
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.48</version>
    <scope>runtime</scope>
</dependency>
```
`provided`依赖表示编译时需要，但运行时不需要。最典型的`provided`依赖是Servlet API，编译的时候需要，但是运行时，Servlet服务器内置了相关的jar，所以运行期不需要：
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.0</version>
    <scope>provided</scope>
</dependency>
```
最后一个问题是，Maven如何知道从何处下载所需的依赖？也就是相关的jar包？答案是Maven维护了一个中央仓库（repo1.maven.org），所有第三方库将自身的jar以及相关信息上传至中央仓库，Maven就可以从中央仓库把所需依赖下载到本地。
Maven并不会每次都从中央仓库下载jar包。一个jar包一旦被下载过，就会被Maven自动缓存在本地目录（用户主目录的.m2目录），所以，除了第一次编译时因为下载需要时间会比较慢，后续过程因为有本地缓存，并不会重复下载相同的jar包。

## 唯一ID
对于某个依赖，Maven只需要3个变量即可唯一确定某个jar包：
```
groupId：属于组织的名称，类似Java的包名；
artifactId：该jar包自身的名称，类似Java的类名；
version：该jar包的版本。
```
通过上述3个变量，即可唯一确定某个jar包。Maven通过对jar包进行PGP签名确保任何一个jar包一经发布就无法修改。修改已发布jar包的唯一方法是发布一个新版本。

因此，某个jar包一旦被Maven下载过，即可永久地安全缓存在本地。

注：只有以`-SNAPSHOT`结尾的版本号会被Maven视为开发版本，开发版本每次都会重复下载，这种SNAPSHOT版本只能用于内部私有的Maven repo，公开发布的版本不允许出现SNAPSHOT。

## Maven镜像
除了可以从Maven的中央仓库下载外，还可以从Maven的镜像仓库下载。如果访问Maven的中央仓库非常慢，我们可以选择一个速度较快的Maven的镜像仓库。Maven镜像仓库定期从中央仓库同步：
```
           slow    ┌───────────────────┐
    ┌─────────────>│Maven Central Repo.│
    │              └───────────────────┘
    │                        │
    │                        │sync
    │                        ▼
┌───────┐  fast    ┌───────────────────┐
│ User  │─────────>│Maven Mirror Repo. │
└───────┘          └───────────────────┘
```
中国区用户可以使用阿里云提供的Maven镜像仓库。使用Maven镜像仓库需要一个配置，在用户主目录下进入`.m2`目录，创建一个`settings.xml`配置文件，内容如下：
```xml
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>aliyun</name>
            <mirrorOf>central</mirrorOf>
            <!-- 国内推荐阿里云的Maven镜像 -->
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
    </mirrors>
</settings>
```
## 搜索第三方插件
最后一个问题：如果我们要引用一个第三方组件，比如`okhttp`，如何确切地获得它的`groupId`、`artifactId`和`version`？方法是通过search.maven.org搜索关键字，找到对应的组件后，直接复制.

**一个重要知识点**
在settings.xml文件中,格式一定要非常严格,即使多一个空格都不行,添加内容的时候,切记一定要按照原来的格式来办。

## 小结:
1. Maven通过解析依赖关系确定项目所需的jar包，常用的4种`scope`有：`compile`（默认），`test`，`runtime`和`provided`；

2. Maven从中央仓库下载所需的jar包并缓存在本地；

3. 可以通过镜像仓库加速下载。

# 构建流程
Maven不但有标准化的项目结构，而且还有一套标准化的构建流程，可以自动化实现编译，打包，发布，等等。

## Lifecycle和Phase
使用Maven时，我们首先要了解什么是Maven的生命周期（lifecycle）。
Maven的生命周期由一系列阶段（phase）构成，以内置的生命周期default为例，它包含以下phase：
> validate
initialize
generate-sources
process-sources
generate-resources
process-resources
compile
process-classes
generate-test-sources
process-test-sources
generate-test-resources
process-test-resources
test-compile
process-test-classes
test
prepare-package
package
pre-integration-test
integration-test
post-integration-test
verify
install
deploy

如果我们运行`mvn package`，Maven就会执行`default`生命周期，它会从开始一直运行到`package`这个phase为止：
> validate
...
package
如果我们运行`mvn compile`，Maven也会执行`default`生命周期，但这次它只会运行到`compile`，即以下几个phase
> validate
...
compile

Maven另一个常用的生命周期是`clean`，它会执行3个phase：
> pre-clean
clean （注意这个clean不是lifecycle而是phase）
post-clean

更复杂的例子是指定多个phase，例如，运行`mvn clean package`，Maven先执行`clean`生命周期并运行到`clean`这个phase，然后执行`default`生命周期并运行到`package`这个phase，实际执行的phase如下：
> pre-clean
clean （注意这个clean是phase）
validate
...
package

在实际开发过程中，经常使用的命令有：
`mvn clean`：清理所有生成的class和jar；
`mvn clean compile`：先清理，再执行到`compile`；
`mvn clean test`：先清理，再执行到`test`，因为执行`test`前必须执行`compile`，所以这里不必指定`compile`；
`mvn clean package`：先清理，再执行到`package`。
大多数phase在执行过程中，因为我们通常没有在pom.xml中配置相关的设置，所以这些phase什么事情都不做。
经常用到的phase其实只有几个：
clean：清理
compile：编译
test：运行测试
package：打包

## Goal
执行一个phase又会触发一个或多个goal：
执行的Phase: compile
对应执行的Goal: compiler:compile
执行的Phase: test
对应执行的Goal: compiler:testCompile surefire:test

goal的命名总是`abc:xyz`这种形式。

看到这里，相信大家对lifecycle、phase和goal已经明白了吧？

其实我们类比一下就明白了：
lifecycle相当于Java的package，它包含一个或多个phase；
phase相当于Java的class，它包含一个或多个goal；
goal相当于class的method，它其实才是真正干活的。

大多数情况，我们只要指定phase，就默认执行这些phase默认绑定的goal，只有少数情况，我们可以直接指定运行一个goal，例如，启动Tomcat服务器：
`mvn tomcat:run`

## 小结
Maven通过lifecycle、phase和goal来提供标准的构建流程。

最常用的构建命令是指定phase，然后让Maven执行到指定的phase：
```
mvn clean
mvn clean compile
mvn clean test
mvn clean package
```
通常情况，我们总是执行phase默认绑定的goal，因此不必指定goal。