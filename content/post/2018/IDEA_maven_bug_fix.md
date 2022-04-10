+++
title = "通过一次IDEA Spring Boot 启动报错深入理解maven scope标签"
date = 2018-08-18T10:25:00+08:00
lastmod = 2022-04-10T01:46:55+08:00
tags = ["IDE", "Bug"]
categories = ["Bug"]
draft = false
toc = true
+++

## 起因 {#起因}

昨天同事突然告诉我项目无法启动了, 但是神奇的是我也在用一样的代码,
但是在我的机器上并没有出现这个问题, 所以我就开始 fuck
bug 了,报错信息大概是下面这个样子

```java
    Caused by: java.lang.IllegalStateException: Failed to introspect annotated methods on class
    org.springframework.boot.web.support.SpringBootServletInitializer
        at org.springframework.core.type.StandardAnnotationMetadata.
    getAnnotatedMethods(StandardAnnotationMetadata.java:163) ~[spring-core-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        at org.springframework.context.annotation.ConfigurationClassParser.
    retrieveBeanMethodMetadata(ConfigurationClassParser.java:380) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        at org.springframework.context.annotation.ConfigurationClassParser.
    doProcessConfigurationClass(ConfigurationClassParser.java:314) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        at org.springframework.context.annotation.ConfigurationClassParser.
    processConfigurationClass(ConfigurationClassParser.java:245) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        at org.springframework.context.annotation.ConfigurationClassParser.
    parse(ConfigurationClassParser.java:198) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        at org.springframework.context.annotation.ConfigurationClassParser.
    parse(ConfigurationClassParser.java:167) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        ... 17 common frames omitted
    Caused by: java.lang.NoClassDefFoundError: javax/servlet/ServletContext
        at java.lang.Class.getDeclaredMethods0(Native Method) ~[na:1.8.0_111]
        at java.lang.Class.privateGetDeclaredMethods(Class.java:2701) ~[na:1.8.0_111]
        at java.lang.Class.getDeclaredMethods(Class.java:1975) ~[na:1.8.0_111]
        at org.springframework.core.type.StandardAnnotationMetadata.getAnnotatedMethods
    (StandardAnnotationMetadata.java:152) ~[spring-core-4.3.10.RELEASE.jar:4.3.10.RELEASE]
        ... 22 common frames omitted
    Caused by: java.lang.ClassNotFoundException: javax.servlet.ServletContext
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381) ~[na:1.8.0_111]
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_111]
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331) ~[na:1.8.0_111]
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_111]
        ... 26 common frames omitted
```


## 解决方法 {#解决方法}

通过网上一番 Goolge, 答案如下

```text
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>provided</scope>
    </dependency>
```

将 pom.xml 文件中 tomcat 依赖中的 `scope` 注掉, 问题解决.


## scope 标签的作用 {#scope 标签的作用}

Maven 官网中是这么介绍 scope 的
[传送门](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

Dependency scope is used to limit the transitivity of a dependency, and
also to affect the classpath used for various build tasks.

There are 6 scopes available:

-   compile This is the default scope, used if none is specified. Compile
    dependencies are available in all classpaths of a project.
    Furthermore, those dependencies are propagated to dependent projects.
-   provided This is much like `compile`, but indicates you expect the JDK
    or a container to provide the dependency at runtime. For example, when
    building a web application for the Java Enterprise Edition, you would
    set the dependency on the Servlet API and related Java EE APIs to
    scope `provided` because the web container provides those classes.
    This scope is only available on the compilation and test classpath,
    and is not transitive.
-   runtime This scope indicates that the dependency is not required for
    compilation, but is for execution. It is in the runtime and test
    classpaths, but not the compile classpath.
-   test This scope indicates that the dependency is not required for
    normal use of the application, and is only available for the test
    compilation and execution phases. This scope is not transitive.
-   system This scope is similar to `provided` except that you have to
    provide the JAR which contains it explicitly. The artifact is always
    available and is not looked up in a repository.
-   import (only available in Maven 2.0.9 or later) This scope is only
    supported on a dependency of type `pom` in the
    `<dependencyManagement>` section. It indicates the dependency to be
    replaced with the effective list of dependencies in the specified
    POM's `<dependencyManagement>` section. Since they are replaced,
    dependencies with a scope of `import` do not actually participate in
    limiting the transitivity of a dependency.

> 通过上面的解释我们可以发现=provided=属性是可以在编译和运行期间引入的啊,
> 但是为啥会报错呢, 难道是 Maven 的 Bug? 还是别的原因, 这里我又深究了一下,
> 结果如下:


### 研究结果 {#研究结果}

现在几乎可以确认应该不是 Maven 的,
在[William_Cheung 的博客](https://blog.csdn.net/qq496013218/article/details/76241027)
的文章中提到这个是 IDEA 的一个 Bug,
IDEA 不会将标有=provided=加入到 classpath 中,
详细信息查看[Add an
option to provide runtime classpath with dependencies in provided
scope](https://youtrack.jetbrains.com/issue/IDEA-107048) 那么现在问题来了.....为啥我的 IDEA 可以呢???
从 youtrack 的评论中我找到了这条评论:
[Guillaume
Simard的评论](https://youtrack.jetbrains.com/issue/IDEA-107048#focus=streamItem-27-2726337-0-0) 在评论中提到,这个 bug 已经在新版中得到解决,
这里我就不搬运了,
附上连接[IntelliJ
IDEA 2018.1 Public Preview](https://blog.jetbrains.com/idea/2018/02/intellij-idea-2018-1-public-preview/)


### 结论 {#结论}

现在看来, 已经非常明了了, 这个异常的原因是低版本 IDEA 的 Bug 导致的,
所以你可以不修改 maven 的配置, 换一个新版本的 IDEA 一样可以结局问题,
至于不想换 IDEA 也不想改配置的人.....
也有解决方案[William_Cheung 的博客](https://blog.csdn.net/qq496013218/article/details/76241027)中已经很明了了,我也不在此多说了.
