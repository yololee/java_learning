# SpringCloudAlibaba：Nacos配置的多文件加载与共享配置

## 一、加载多个配置

我们已经知道Spring应用对Nacos中配置内容的对应关系是通过下面三个参数控制的：

- spring.cloud.nacos.config.prefix
- spring.cloud.nacos.config.file-extension
- spring.cloud.nacos.config.group

默认情况下，会加载`Data ID=${spring.application.name}.properties`，`Group=DEFAULT_GROUP`的配置。

假设现在有这样的一个需求：我们想要对所有应用的Actuator模块以及日志输出做统一的配置管理。所以，我们希望可以将Actuator模块的配置放在独立的配置文件`actuator.properties`文件中，而对于日志输出的配置放在独立的配置文件`log.properties`文件中。通过拆分这两类配置内容，希望可以做到配置的共享加载与统一管理。

### 1.解决方法

**第一步**：在Nacos中创建`Data ID=actuator.properties`，`Group=DEFAULT_GROUP`和`Data ID=log.properties`，`Group=DEFAULT_GROUP`的配置内容。

![在这里插入图片描述](https://img-blog.csdnimg.cn/978b1a9374034264915a1fa6ebf797a3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


我们这里会加载这三个配置文件

**第二步**：在Spring Cloud应用中通过使用`spring.cloud.nacos.config.ext-config`参数来配置要加载的这两个配置内容，比如：

- properties格式

```properties
server.port=9004
spring.application.name=config
spring.cloud.nacos.discovery.server-addr=192.168.31.100:8848
spring.cloud.nacos.config.server-addr=192.168.31.100:8848
spring.cloud.nacos.config.file-extension=yaml
spring.cloud.nacos.config.ext-config[0].data-id=actuator.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true
spring.cloud.nacos.config.ext-config[1].data-id=log.properties
spring.cloud.nacos.config.ext-config[1].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[1].refresh=true
```

- yml格式

```yml
server:
  port: 9004
spring:
  application:
    name: config
  cloud:
    nacos:
      config:
        ext-config:
        - data-id: actuator.properties
          group: DEFAULT_GROUP
          refresh: true
        - data-id: log.properties
          group: DEFAULT_GROUP
          refresh: true
        file-extension: yaml
        server-addr: 192.168.31.100:8848
      discovery:
        server-addr: 192.168.31.100:8848
```

可以看到，`spring.cloud.nacos.config.ext-config`配置是一个数组List类型。每个配置中包含三个参数：`data-id`、`group`，`refresh`；前两个不做赘述，与Nacos中创建的配置相互对应，`refresh`参数控制这个配置文件中的内容时候支持自动刷新，默认情况下，只有默认加载的配置才会自动刷新，对于这些扩展的配置加载内容需要配置该设置时候才会实现自动刷新。

> 注意：这个配置必须在bootstrap.properties文件中加载

**第三步**：启动测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/e168e80322c94277aecba3239e7564a9.png#pic_center)


```text
请求地址：127.0.0.1:9004/config/info

响应结果：
config-test.yml:there is test version=1.0
actuator.properties:"hello actuator"
log.properties:"hello log"
```

## 二、共享配置

通过上面加载多个配置的实现，实际上我们已经可以实现不同应用共享配置了。但是Nacos中还提供了另外一个便捷的配置方式，比如下面的设置与上面使用的配置内容是等价的：

```properties
spring.cloud.nacos.config.shared-dataids=actuator.properties,log.properties
spring.cloud.nacos.config.refreshable-dataids=actuator.properties,log.properties
```

- `spring.cloud.nacos.config.shared-dataids`参数用来配置多个共享配置的`Data Id`，多个的时候用用逗号分隔
- `spring.cloud.nacos.config.refreshable-dataids`参数用来定义哪些共享配置的`Data Id`在配置变化时，应用中可以动态刷新，多个`Data Id`之间用逗号隔开。如果没有明确配置，默认情况下所有共享配置都不支持动态刷新

## 三、配置加载的优先级

当我们加载多个配置的时候，如果存在相同的key时，我们需要深入了解配置加载的优先级关系。

在使用Nacos配置的时候，主要有以下三类配置：

- A: 通过`spring.cloud.nacos.config.shared-dataids`定义的共享配置
- B: 通过`spring.cloud.nacos.config.ext-config[n]`定义的加载配置
- C: 通过内部规则（`spring.cloud.nacos.config.prefix`、`spring.cloud.nacos.config.file-extension`、`spring.cloud.nacos.config.group`这几个参数）拼接出来的配置

> 修改bootstrap.properties配置文件

```properties
server.port=9004
spring.application.name=config
spring.cloud.nacos.discovery.server-addr=192.168.31.100:8848
spring.cloud.nacos.config.server-addr=192.168.31.100:8848
spring.cloud.nacos.config.file-extension=yaml
spring.cloud.nacos.config.ext-config[0].data-id=actuator.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true
#spring.cloud.nacos.config.ext-config[1].data-id=log.properties
#spring.cloud.nacos.config.ext-config[1].group=DEFAULT_GROUP
#spring.cloud.nacos.config.ext-config[1].refresh=true

spring.cloud.nacos.config.shared-dataids=log.properties
spring.cloud.nacos.config.refreshable-dataids=log.properties
```

> 根据上面的配置，应用分别会去加载三类不同的配置文件，启动应用的时候，将会在日志中看到如下输出：

```java
2021-10-25 16:31:20.897  INFO 14456 --- [           main] c.a.c.n.c.NacosPropertySourceBuilder     : Loading nacos data, dataId: 'log.properties', group: 'DEFAULT_GROUP'
2021-10-25 16:31:20.902  INFO 14456 --- [           main] c.a.c.n.c.NacosPropertySourceBuilder     : Loading nacos data, dataId: 'actuator.properties', group: 'DEFAULT_GROUP'
2021-10-25 16:31:20.910  INFO 14456 --- [           main] c.a.c.n.c.NacosPropertySourceBuilder     : Loading nacos data, dataId: 'config-test.yaml', group: 'DEFAULT_GROUP'
```

> 结论：后面加载的配置会覆盖之前加载的配置，所以优先级关系是：`A < B < C`