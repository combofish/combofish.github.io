---
layout: post
title: "Getting started with Maven"
date: "2021-04-12 22：52：00"
tag: Maven Java Idea
---
- [Maven](#org61d8abf)
  - [为什么使用 Maven ?](#org0626909)
  - [Maven 简介](#orga7959ba)
  - [标准目录结构](#orge179b8e)
    - [一般的工程包括这四部分](#org93465ce)
    - [maven 标准目录结构](#org94dd83c)
  - [Maven POM](#orgc2b6ac5)
  - [生命周期](#org0185c3e)
  - [常用命令](#orgbabeaa6)
  - [添加依赖](#org082919c)
    - [依赖关系](#orgebb7a1f)
  - [仓库](#org000e77d)
    - [Maven 仓库有三种类型：](#orgc2d7e17)
    - [本地仓库（local）](#org62840f1)
  - [Idea 设置 Maven](#org0a5b030)


<a id="org61d8abf"></a>

# Maven


<a id="org0626909"></a>

## 为什么使用 Maven ?

当如果我们的项目依赖第三方的jar包，例如 JDBC 驱动： mysql-connector-java ，我们需要手动下载，并配置加载到工程中。 需要测试时，我们需要 JUnit 包，又要重复上述过程，新建了一个工程，又需要重新配置。

-   **有没有一个工具可以帮我们自动下载并导入包呢？** -> Maven


<a id="orga7959ba"></a>

## Maven 简介

Maven 是一个标准化的Java项目管理和构建工具。解决了依赖管理问题。

-   提供了一套标准化的项目结构；
-   提供了一套标准化的构建流程（编译，测试，打包，发布……）；
-   提供了一套依赖管理机制。


<a id="orge179b8e"></a>

## 标准目录结构


<a id="org93465ce"></a>

### 一般的工程包括这四部分

-   核心代码部分
-   配置文件部分
-   测试代码部分
-   测试配置文件


<a id="org94dd83c"></a>

### maven 标准目录结构

-   一个使用Maven管理的普通的Java项目，它的目录结构默认如下：

```sh
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

| 目录                 | 作用                                   |
|-------------------- |-------------------------------------- |
| /src/main/java       | 项目的java源代码                       |
| /src/main/resources  | 项目的资源，配置文件，比如说property文件，springmvc.xml |
| /src/test/java       | 项目的测试类，比如说Junit代码          |
| /src/test/resources  | 测试用的资源                           |
| /src/main/webapp     | Web应用文件目录，web项目的信息         |
| /target              | 打包输出目录                           |
| /target/classes      | 编译输出目录                           |
| /target/test-classes | 测试编译输出目录                       |


<a id="orgc2b6ac5"></a>

## Maven POM

**POM** ( Project Object Model，项目对象模型 ) 是 Maven 工程的基本工作单元，是一个XML文件，包含了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。

-   执行任务或目标时，Maven 会在当前目录中查找 POM, 一般是 pom.xml 。它读取 POM，获取所需的配置信息，然后执行目标。
-   一个典型的 pom.xml , 该文件需要 project 元素和三个必需字段：groupId，artifactId，version。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>untitled2</artifactId>
  <version>1.0-SNAPSHOT</version>
</project>
```

-   project 工程的根标签。
-   modelVersion 模型版本。
-   groupId 工程组的标识。它在一个组织或者项目中通常是唯一的。
-   artifactId 工程的标识。它通常是工程的名称。groupId 和 artifactId 一起定义了 artifact 在仓库中的位置。
-   version 工程的版本号。


<a id="org0185c3e"></a>

## 生命周期

-   Maven 构建生命周期定义了一个项目构建跟发布的过程。

一个典型的 Maven 构建（build）生命周期是由以下几个阶段的序列组成的：

| 阶段 | 处理     | 描述 |                              |
|--- |-------- |---- |---------------------------- |
| 验证 | validate | 验证项目 | 验证项目是否正确且所有必须信息是可用的 |
| 编译 | compile  | 执行编译 | 源代码编译在此阶段完成       |
| 测试 | test     | 测试 | 使用适当的单元测试框架（例如JUnit）运行测试。 |
| 包装 | package  | 打包 | 创建JAR/WAR包如在 pom.xml 中定义提及的包 |
| 检查 | verify   | 检查 | 对集成测试的结果进行检查，以保证质量达标 |
| 安装 | install  | 安装 | 安装打包的项目到本地仓库，以供其他项目使用 |
| 部署 | deploy   | 部署 | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程 |


<a id="orgbabeaa6"></a>

## 常用命令

```sh
mvn compile	# 编译项目的源代码。
mvn test	# 使用合适的单元测试框架运行测试（Juint是其中之一）。
mvn install 	# 安装项目包到本地仓库，这样项目包可以用作其他本地项目的依赖。
mvn deploy	# 将最终的项目包复制到远程仓库中与其他开发者和项目共享。
mvn package	# 将编译后的代码打包成可分发格式的文件，比如JAR、WAR或者EAR文件。
mvn tomcat:run  # 启动Tomcat服务器
```


<a id="org082919c"></a>

## 添加依赖

-   在 pom.xml 文件中配置依赖

```xml
<dependencies>
  <dependency>
  <groupId>commons-logging</groupId>
  <artifactId>commons-logging</artifactId>
  <version>1.2</version>
  </dependency>
</dependencies>
```

使用<dependency>声明一个依赖后，Maven就会自动下载这个依赖包并把它放到classpath中。

-   groupId：属于组织的名称，类似Java的包名；
-   artifactId：该jar包自身的名称，类似Java的类名；
-   version：该jar包的版本。

通过上述3个变量，即可唯一确定某个jar包。Maven通过对jar包进行PGP签名确保任何一个jar包一经发布就无法修改。修改已发布jar包的唯一方法是发布一个新版本。 因此，某个jar包一旦被Maven下载过，即可永久地安全缓存在本地。

-   注：只有以-SNAPSHOT结尾的版本号会被Maven视为开发版本，开发版本每次都会重复下载，这种SNAPSHOT版本只能用于内部私有的Maven repo，公开发布的版本不允许出现SNAPSHOT。


<a id="orgebb7a1f"></a>

### 依赖关系

Maven定义了几种依赖关系，分别是compile、test、runtime和provided：

| scope    | 说明                     | 示例            |
|-------- |------------------------ |--------------- |
| compile  | 编译时需要用到该jar包（默认） | commons-logging |
| test     | 编译Test时需要用到该jar包 | junit           |
| runtime  | 编译时不需要，但运行时需要用到 | mysql           |
| provided | 编译时需要用到，但运行时由JDK或某个服务器提供 | servlet-api     |

其中，默认的compile是最常用的，Maven会把这种类型的依赖直接放入classpath。

-   test依赖表示仅在测试时使用，正常运行时并不需要。最常用的test依赖就是JUnit：

```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-api</artifactId>
  <version>5.3.2</version>
  <scope>test</scope>
</dependency>
```

-   runtime依赖表示编译时不需要，但运行时需要。最典型的runtime依赖是JDBC驱动，例如MySQL驱动：

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.48</version>
  <scope>runtime</scope>
</dependency>
```

-   provided依赖表示编译时需要，但运行时不需要。最典型的provided依赖是Servlet API，编译的时候需要，但是运行时，Servlet服务器内置了相关的jar，所以运行期不需要：

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.0</version>
  <scope>provided</scope>
</dependency>
```


<a id="org000e77d"></a>

## 仓库

**仓库** 是 Maven 下载所需依赖的地方，也就是存相关jar包的地方。

-   Maven维护了一个中央仓库（repo1.maven.org），所有第三方库将自身的jar以及相关信息上传至中央仓库，Maven就可以从中央仓库把所需依赖下载到本地。


<a id="orgc2d7e17"></a>

### Maven 仓库有三种类型：

-   中央仓库 其实我们使用的大多数第三方模块都是这个用法，例如，我们使用commons logging、log4j这些第三方模块，就是第三方模块的开发者自己把编译好的jar包发布到Maven的中央仓库中。

-   私有仓库 私有仓库是指公司内部如果不希望把源码和jar包放到公网上，那么可以搭建私有仓库。私有仓库总是在公司内部使用，它只需要在本地的~/.m2/settings.xml中配置好，使用方式和中央仓位没有任何区别。

-   本地仓库 本地仓库是指把本地开发的项目“发布”在本地，这样其他项目可以通过本地仓库引用它。但是我们不推荐把自己的模块安装到Maven的本地仓库，因为每次修改某个模块的源码，都需要重新安装，非常容易出现版本不一致的情况。更好的方法是使用模块化编译，在编译的时候，告诉Maven几个模块之间存在依赖关系，需要一块编译，Maven就会自动按依赖顺序编译这些模块。


<a id="org62840f1"></a>

### 本地仓库（local）

Maven 的本地仓库，在安装 Maven 后并不会创建，它是在第一次执行 maven 命令的时候才被创建。 运行 Maven 的时候，Maven 所需要的任何构件都是直接从本地仓库获取的。如果本地仓库没有，它会首先尝试从远程仓库下载构件至本地仓库，然后再使用本地仓库的构件。

-   默认情况下，不管Linux还是 Windows，每个用户在自己的用户目录下都有一个路径名为 %HOME/.m2/respository/ 的仓库目录。
-   Maven 本地仓库默认被创建在 %USER<sub>HOME</sub>% 目录下。要修改默认位置，在 %M2<sub>HOME</sub>%\conf 目录中的 Maven 的 settings.xml 文件中定义另一个路径。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
			      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/path/to/MyLocalRepository</localRepository>
</settings>
```


<a id="org0a5b030"></a>

## Idea 设置 Maven

如下：Settings -> Build,Execution,Deployment -> Build Tools -> Maven -> Maven home path
