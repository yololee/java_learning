# Springboot面试题

### 什么是Springboot

```java
Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简化了繁重的配置，提供了各种启动器，开发者能快速上手
    
优点：
    1.独立运行，内置tomcat服务器
    2.简化配置
    3.避免大量的 Maven 导入和各种版本冲突
    4.没有代码生成，也不需要XML配置
```

### spring boot 核心配置文件是什么？bootstrap.properties 和 application.properties 有何区别 ?

```java
单纯做 Spring Boot 开发，不太容易遇到 bootstrap.properties 配置文件，但是在结合 Spring Cloud 时，这个配置就会经常遇到了，特别是在需要加载一些远程配置文件的时侯
    
bootstrap (. yml 或者 . properties)：bootstrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 bootstrap 里面的属性不能被覆盖；
    
application (. yml 或者 . properties)：由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。    
```

### Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？

```java
启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。
```

### Spring Boot 自动配置原理是什么

```java
启动类上有一个SpringBootApplication这个注解，这个注解是一个复合注解，里面有一个@EnableAutoConfiguration这个注解，他也是一个复合注解，主要功能是由里面的@import来提供的，他会扫描到一个META-INF/spring.factories的jar包，里面是以键值对存在的一些全类名的列表，然后通过ConditionOnbean，或者ConditionOnClass等注解，判断是否有依赖，如果有就自动配置类加载到Spring容器中
```

### Spring Boot 配置加载顺序

```java
在 Spring Boot 里面，可以使用以下几种方式来加载配置。
1）properties文件；
2）YAML文件；
3）系统环境变量；
4）命令行参数；
```

### Spring Boot 打成的 jar 和普通的 jar 有什么区别

```java
Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 java -jar xxx.jar 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类
    
Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。    
```

