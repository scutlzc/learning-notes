# maven学习笔记

> 定义：maven是一款服务于java平台的自动化构建工具

> 构建的各个环节
>
> - 清理clean：将以前编译得到的旧文件class字节码文件删除
> - 编译compile：将java源程序编译成class字节码文件
> - 测试test：自动测试，自动调用junit程序
> - 报告report：测试程序执行的结果
> - 打包package：动态Web工程打War包，java工程打jar包
> - 安装install：Maven特定的概念-----将打包得到的文件复制到“仓库”中的指定位置
> - 部署deploy：将动态Web工程生成的war包复制到Servlet容器下，使其可以运行

## 仓库和坐标

pom.xml：Project Object Model 项目对象类型。它是maven的核心配置文件，所有的构建的配置都在这里设置。

坐标：使用下图的三个向量在仓库中定义唯一的maven工程

<img src="E:\learning-notes\java后端开发学习\maven学习\1.png" alt="maven向量演示" style="zoom:80%;" />

标签`<dependency>`中包括的是项目所依赖的文件。

仓库分为：

- 本地仓库：位于当前电脑上的仓库，一般在**c:\Usrs[登录当前系统的用户名].m2\repository**。
- 远程仓库
  - 私服：搭建在局域网中，一般公司都会有私服，私服一般使用nexus来搭建。
  - 中央仓库：架设在Internet上，springframework就是在中央仓库上

## 依赖

maven在解析依赖信息时首先会到本地仓库寻找依赖的jar包，对于没有找到的jar包会去中央仓库去寻找maven坐标来获取jar包，获取到jar包后保存到本地仓库。对于中央仓库也找不到的jar包依赖，项目会编译失败。

如果是自己开发的maven工程，通过install命令将被依赖的maven工程的jar包导入到本地仓库中。

### 依赖范围

如上图中，scope表示依赖的范围

> - **compile**，默认值，适用于所有阶段（开发、测试、部署、运行），本jar会一直存在所有阶段。
> - **provided，**只在开发、测试阶段使用，目的是不让Servlet容器和你本地仓库的jar包冲突 。如servlet.jar。
> - **runtime，**只在运行时使用，如JDBC驱动，适用运行和测试阶段。
> - **test，**只在测试时使用，用于编译和运行测试代码。不会随项目发布。
> - **system，**类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

## 生命周期

maven有三套相互独立的生命周期。

### Clean Lifecycle

它进行真正构建前的一些清理工作，包括三个阶段：

- pre-clean 执行一些需要在clean之前完成的工作
- clean 移除所有上一次构建生成的文件
- post-clean 执行一些需要在clean之后立刻完成的工作

### Default Lifecycle

构建的核心部分，编译，测试，打包，部署等。

- validate
- generate-sources
- process-sources
- generate-resources
- process-resources 复制并处理资源文件，至目标目录，准备打包
- compile 编译项目的源代码
- process-classes
- generate-test-sources
- process-test-sources
- generate-test-resources
- process-test-resources 复制并处理资源文件，至目标测试目录
- test-compile 编译测试源代码
- process-test-classes
- test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署
- prepare-package
- package 接受编译好的代码，打包成可发布的格式，如 JAR
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install 将包安装至本地仓库，以让其它项目依赖。
- deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享

### Site Lifecycle

生成项目报告，站点，发布站点

- pre-site 执行一些需要在生成站点文档之前完成的工作
- site 生成项目的站点文档
- post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
- site-deploy 将生成的站点文档部署到特定的服务器上

## maven工程的依赖高级特性

### 依赖的传递性

<img src="E:\learning-notes\java后端开发学习\maven学习\2.png" style="zoom:80%;" />

WebMavenDemo项目依赖JavaMavenService1 JavaMavenService1项目依赖JavaMavenService2

pom.xml文件配置好依赖关系后，必须首先mvn install后，依赖的jar包才能使用。

- WebMavenDemo的pom.xml文件想能编译通过，JavaMavenService1必须mvn install

- JavaMavenService的pom.xml文件想能编译通过，JavaMavenService2必须mvn install

**非compile范围的依赖不能传递。**

### 依赖版本的原则

**1、路径最短优先原则**

<img src="E:\learning-notes\java后端开发学习\maven学习\3.png" style="zoom: 80%;" />

Service2的log4j的版本是1.2.7版本，Service1排除了此包的依赖，自己加了一个Log4j的1.2.9的版本，那么WebMavenDemo项目遵守路径最短优先原则，Log4j的版本和Sercive1的版本一致。

**2、路径相同先声明优先原则**

<img src="E:\learning-notes\java后端开发学习\maven学习\4.png" style="zoom:80%;" />

这种场景依赖关系发生了变化，WebMavenDemo项目依赖Sercive1和Service2，它俩是同一个路径，那么谁在WebMavenDemo的pom.xml中先声明的依赖就用谁的版本。

## build配置

```xml
<build>
　　<!-- 项目的名字 -->
　　<finalName>WebMavenDemo</finalName>
　　<!-- 描述项目中资源的位置 -->
　　<resources>
　　　　<!-- 自定义资源1 -->
　　　　<resource>
　　　　　　<!-- 资源目录 -->
　　　　　　<directory>src/main/java</directory>
　　　　　　<!-- 包括哪些文件参与打包 -->
　　　　　　<includes>
　　　　　　　　<include>**/*.xml</include>
　　　　　　</includes>
　　　　　　<!-- 排除哪些文件不参与打包 -->
　　　　　　<excludes>
　　　　　　　　<exclude>**/*.txt</exclude>
　　　　　　　　　　<exclude>**/*.doc</exclude>
　　　　　　</excludes>
　　　　</resource>
　　</resources>
　　<!-- 设置构建时候的插件 -->
　　<plugins>
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-compiler-plugin</artifactId>
　　　　　　<version>2.1</version>
　　　　　　<configuration>
　　　　　　　　<!-- 源代码编译版本 -->
　　　　　　　　<source>1.8</source>
　　　　　　　　<!-- 目标平台编译版本 -->
　　　　　　　　<target>1.8</target>
　　　　　　</configuration>
　　　　</plugin>
　　　　<!-- 资源插件（资源的插件） -->
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-resources-plugin</artifactId>
　　　　　　<version>2.1</version>
　　　　　　<executions>
　　　　　　　　<execution>
　　　　　　　　　　<phase>compile</phase>
　　　　　　　　</execution>
　　　　　　</executions>
　　　　　　<configuration>
　　　　　　　　<encoding>UTF-8</encoding>
　　　　　　</configuration>
　　　　</plugin>
　　　　<!-- war插件(将项目打成war包) -->
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-war-plugin</artifactId>
　　　　　　<version>2.1</version>
　　　　　　<configuration>
　　　　　　　　<!-- war包名字 -->
　　　　　　　　<warName>WebMavenDemo1</warName>
　　　　　　</configuration>
　　　　</plugin>
　　</plugins>
</build>
```

> maven依赖项版本查询网站：http://mvnrepository.com/
