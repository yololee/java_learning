# SpringCloudAlibaba：nacos配置的加载规则详解

## 加载规则

在 Nacos Spring Cloud 中，dataId 的完整格式如下：

```java
${prefix}-${spring.profile.active}.${file-extension}
```

prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。

spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 prefix.{file-extension}。

file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

## 举例

假如我们在nacos中创建了一个以下内容的配置：

- `Data ID`：alibaba-nacos-config-client.properties
- `Group`：DEFAULT_GROUP

他们与具体应用的配置关系如下：

- Data ID中的`alibaba-nacos-config-client`：对应客户端的配置`spring.cloud.nacos.config.prefix`，默认值为`${spring.application.name}`，即：服务名
- Data ID中的`properties`：对应客户端的配置`spring.cloud.nacos.config.file-extension`，默认值为`properties`
- Group的值`DEFAULT_GROUP`：对应客户端的配置`spring.cloud.nacos.config.group`，默认值为`DEFAULT_GROUP`

在采用默认值的应用要加载的配置规则就是：`Data ID=${spring.application.name}.properties`，`Group=DEFAULT_GROUP`

> 举例让大家理解这些配置之间的关系

**例子一**：如果我们不想通过服务名来加载，那么可以增加如下配置，就会加载到`Data ID=example.properties`，`Group=DEFAULT_GROUP`的配置内容了

```properties
spring.cloud.nacos.config.prefix=example
```

**例子二**：如果我们想要加载yaml格式的内容，而不是Properties格式的内容，那么可以通过如下配置，实现加载`Data ID=example.yaml`，`Group=DEFAULT_GROUP`的配置内容了：

```properties
spring.cloud.nacos.config.prefix=example
spring.cloud.nacos.config.file-extension=yaml
```

**例子三**：如果我们对配置做了分组管理，那么可以通过如下配置，实现加载`Data ID=example.yaml`，`Group=DEV_GROUP`的配置内容了：

```properties
spring.cloud.nacos.config.prefix=example
spring.cloud.nacos.config.file-extension=yaml
spring.cloud.nacos.config.group=DEV_GROUP
```

## 参考文档

[Nacos官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)