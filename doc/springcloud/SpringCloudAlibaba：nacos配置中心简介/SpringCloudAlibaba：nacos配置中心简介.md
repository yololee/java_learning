# SpringCloudAlibaba：nacos配置中心简介

## 一、配置中心简介

### 1.简介

Nacos 提供用于存储配置和其他元数据的 key/value 存储，为分布式系统中的外部化配置提供服务器端和客户端支持。使用 Spring Cloud Alibaba Nacos Config，您可以在 Nacos Server 集中管理你 Spring Cloud 应用的外部属性配置。
Spring Cloud Alibaba Nacos Config 是 Config Server 和 Client 的替代方案，客户端和服务器上的概念与 Spring Environment 和 PropertySource 有着一致的抽象，在特殊的 bootstrap 阶段，配置被加载到 Spring 环境中。当应用程序通过部署管道从开发到测试再到生产时，您可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容

### 2. dataid

在 Nacos Spring Cloud 中，dataId 的完整格式如下：

```java
${prefix}-${spring.profile.active}.${file-extension}
```

prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。

spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 prefix.{file-extension}。

file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

![在这里插入图片描述](https://img-blog.csdnimg.cn/f5cb4f1b779a484e838abaf040b15a38.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 3. 创建dataid

- 点击新建

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb319ad1dc5a4ff2a3ed552c7c8d674b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 填写

![在这里插入图片描述](https://img-blog.csdnimg.cn/a12e85eb5b60464dbaafa5e1c993b84c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 完成

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c6372e7aede45bb8a8ccdd89eccc1aa.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 二、代码

### 1.pom文件

```xml
<dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

### 2.application.yml

```yml
spring:
  profiles:
    active: dev
```

> spring boot中配置文件优先加载顺序：bootstrap.yml > xxx.yml > xxx.properties
>
> bootstrap.yml 由父Spring ApplicationContext加载。
>
> bootstrap.yml 可以理解成系统级别的一些参数配置，这些参数一般是不会变动的
>
> pplication.yml 可以用来定义应用级别的，如果搭配 spring-cloud-config 使用 application.yml 里面定义的文件可以实现动态替换

### 3.bootstrap.yml

```yml
server:
  port: 9004
spring:
  application:
    name: config
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848
      config:
        server-addr: 192.168.31.100:8848
        file-extension: yaml
```

> 例如：dataId为config-dev.yaml
>
> 这里的config为spring.application.name=config
>
> 这里dev为spring.profiles.active=dev
>
> 这里yaml为spring.cloud.config.file-extension=yaml

### 4.controller
另外，这里还有一个比较重要的注解@RefreshScope，主要用来让这个类下的配置内容支持动态刷新，也就是当我们的应用启动之后，修改了Nacos中的配置内容之后，这里也会马上生效。
```java
@RestController
@RefreshScope
public class ConfigController {
    @Value("${config.info}")
    private String info;
    @GetMapping("/config/info")
    public String info(){
        return info;
    }
}
```

### 5.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConfigApplication.class, args);
    }

}
```

### 6.测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/4e99c44208504dd880a2504a33ec0442.png#pic_center)


## 三、配置中心分类管理

![在这里插入图片描述](https://img-blog.csdnimg.cn/2887df40f1fb438786e4f51c18e11c82.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


1. 默认情况

   Namespace= public, Group= DEFAULT_GROUP,默认Cluster是DEFAULT

2. Namespace主要实现隔离

   比方说我们现在有三个环境：开发、测试、生产环境,我们就可以创建三个 Namespace,不同的 Namespace之间是隔离的

3. Group

   Group可以把不同的微服服务划分到同一个分组里面

4. Service

   Service就是微服务;个 Servicer可以包含多个 Cluster(集群), Nacos默认 Cluster是 DEFAULT, Cluster=是对指定微服务的个虚拟划分

5. Instance

   微服务的实例

### 1.DataID

指定spring.profile.active和配置文件的DataId来使不同环境下读取不同的数据

- 新建test的DataId

![在这里插入图片描述](https://img-blog.csdnimg.cn/e536d437a71d469b9d1ea0fe3c5c2b20.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_17,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 修改application.yml

```yml
spring:
  profiles:
    active: test
```

- 重启

![在这里插入图片描述](https://img-blog.csdnimg.cn/516d349a22384b6785feb4d5c8df0aae.png#pic_center)


### 2.Group方案

- 新建两个组test环境

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f3a324226eb4b2db13571f50c140180.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_14,color_FFFFFF,t_70,g_se,x_16#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/82cde80b457248b0a0f1d2852880c3af.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_12,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 修改bootstrap.yml为测试环境的组

```yml
server:
  port: 9004
spring:
  application:
    name: config
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848
      config:
        server-addr: 192.168.31.100:8848
        file-extension: yaml
        group: DEV_GROUP
```

- 重启

![在这里插入图片描述](https://img-blog.csdnimg.cn/96c9a05174484d79b8fd2f28a5539397.png#pic_center)


### 3.namespace

- 创建命令空间

![在这里插入图片描述](https://img-blog.csdnimg.cn/e910337c3fc143248f58ac6edb03c10e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 查看服务列表

![在这里插入图片描述](https://img-blog.csdnimg.cn/5a149fc24ecd4d3da7bbcb1862a54fe4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- dev创建配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/b393ed868c6d4895b587a04a3be7c71d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)


- 修改bootstrap.yml

```yml
server:
  port: 9004
spring:
  application:
    name: config
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848
      config:
        server-addr: 192.168.31.100:8848
        file-extension: yaml
        group: TEST_GROUP
        namespace: 88f92e71-04fa-4c13-8892-736adb162f47
```

- 重启

![在这里插入图片描述](https://img-blog.csdnimg.cn/f94d10c3c01b483499e8f9a25af74bd6.png#pic_center)