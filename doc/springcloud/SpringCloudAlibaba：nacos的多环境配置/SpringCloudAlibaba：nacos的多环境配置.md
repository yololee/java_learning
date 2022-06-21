# SpringCloudAlibaba：nacos的多环境配置

## 1.使用`Data ID`与`profiles`实现

`Data ID`在Nacos中，我们可以理解为就是一个Spring Cloud应用的配置文件名。我们知道默认情况下`Data ID`的名称格式是这样的：`${spring.application.name}.properties`，即：以Spring Cloud应用命名的properties文件。

实际上，`Data ID`的规则中，还包含了环境逻辑，这一点与Spring Cloud Config的设计类似。我们在应用启动时，可以通过`spring.profiles.active`来指定具体的环境名称，此时客户端就会把要获取配置的`Data ID`组织为：`${spring.application.name}-${spring.profiles.active}.properties`。

实际上，更原始且最通用的匹配规则，是这样的：`${spring.cloud.nacos.config.prefix}`-`${spring.profile.active}`.`${spring.cloud.nacos.config.file-extension}`。而上面的结果是因为`${spring.cloud.nacos.config.prefix}`和`${spring.cloud.nacos.config.file-extension}`都使用了默认值

### 案例

- 第一步：在nacos中创建俩个不同环境的配置内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/4d6512d0dd1340e4997b0340c9088945.png#pic_center)


如上图，我们为`config`应用，定义了dev和test的两个独立的环境配置。我们可以在里面定义不同的内容值，以便后续验证是否真实加载到了正确的配置。

- 第二步：在`config`应用的配置文件中，增加环境配置：`spring.profiles.active=dev`

- **第三步**：启动应用，我们可以看到日志中打印了，加载的配置文件：

```java
2021-10-25 15:04:02.675  INFO 18576 --- [           main] c.a.c.n.c.NacosPropertySourceBuilder     : Loading nacos data, dataId: 'config-test.yaml', group: 'DEFAULT_GROUP'
```

## 2.使用`Group`实现

`Group`在Nacos中是用来对`Data ID`做集合管理的重要概念。所以，如果我们把一个环境的配置视为一个集合，那么也就可以实现不同环境的配置管理。对于`Group`的用法并没有固定的规定，所以我们在实际使用的时候，需要根据我们的具体需求，可以是架构运维上对多环境的管理，也可以是业务上对不同模块的参数管理。为了避免冲突，我们需要在架构设计之初，做好一定的规划。这里，我们先来说说如何用`Group`来实现多环境配置管理的具体实现方式。

### 案例

- 第一步：先在Nacos中，通过区分`Group`来创建两个不同环境的配置内容。

![在这里插入图片描述](https://img-blog.csdnimg.cn/08a48fe6d3744466a75d4fb7b880f8a6.png#pic_center)


如上图，我们为`config`应用，定义了DEV环境和TEST环境的两个独立的配置，这两个匹配与上一种方法不同，它们的`Data ID`是完全相同的，只是`GROUP`不同

- **第二步**：在`config`应用的配置文件中，增加`Group`的指定配置：`spring.cloud.nacos.config.group=DEV_GROUP`

- **第三步**：启动应用，我们可以看到日志中打印了，加载的配置文件：

```java
2021-10-25 15:08:28.776  INFO 18552 --- [           main] c.a.c.n.c.NacosPropertySourceBuilder     : Loading nacos data, dataId: 'config-test.yaml', group: 'TEST_GROUP'
```

## 3.使用`Namespace`实现

`Namespace`在本系列教程中，应该还是第一次出现。先来看看官方的概念说明：用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的`Group`或`Data ID`的配置。`Namespace`的常用场景之一是不同环境的配置的区分隔离，例如：开发测试环境和生产环境的资源（如配置、服务）隔离等。

### 案例

- 第一步：先在Nacos中，根据环境名称来创建多个`Namespace`

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b018d98d7654f058e466a88af9869c0.png#pic_center)


- 第二步：查看配置列表

![在这里插入图片描述](https://img-blog.csdnimg.cn/2dd79c95803e42e4b7c48964027ab823.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


- **第三步**：在`config`应用的配置文件中，增加`Namespace`的指定配置，比如：`spring.cloud.nacos.config.namespace=88f92e71-04fa-4c13-8892-736adb162f47`

> 注意namespace的配置不是使用名称，而是使用Namespace的ID

- 第四步：启动应用，通过访问`localhost:9003/test`接口，验证一下返回内容是否正确。这种方式下，目前版本的日志并不会输出与`Namespace`相关的信息，所以还无法以此作为加载内容的判断依据

## 4.三种方式的优缺点

### 第一种：通过`Data ID`与`profile`实现。

- *优点*：这种方式与Spring Cloud Config的实现非常像，用过Spring Cloud Config的用户，可以毫无违和感的过渡过来，由于命名规则类似，所以要从Spring Cloud Config中做迁移也非常简单。
- *缺点*：这种方式在项目与环境多的时候，配置内容就会显得非常混乱。配置列表中会看到各种不同应用，不同环境的配置交织在一起，非常不利于管理。
- *建议*：项目不多时使用，或者可以结合`Group`对项目根据业务或者组织架构做一些拆分规划。

### 第二种：通过`Group`实现

- *优点*：通过`Group`按环境讲各个应用的配置隔离开。可以非常方便的利用`Data ID`和`Group`的搜索功能，分别从应用纬度和环境纬度来查看配置。
- *缺点*：由于会占用`Group`纬度，所以需要对`Group`的使用做好规划，毕竟与业务上的一些配置分组起冲突等问题。
- *建议*：这种方式虽然结构上比上一种更好一些，但是依然可能会有一些混乱，主要是在`Group`的管理上要做好规划和控制。

### 第三种：通过`Namespace`实现。

- *优点*：官方建议的方式，通过`Namespace`来区分不同的环境，释放了`Group`的自由度，这样可以让`Group`的使用专注于做业务层面的分组管理。同时，Nacos控制页面上对于`Namespace`也做了分组展示，不需要搜索，就可以隔离开不同的环境配置，非常易用。
- *缺点*：没有啥缺点，可能就是多引入一个概念，需要用户去理解吧。
- *建议*：直接用这种方式长远上来说会比较省心。虽然可能对小团队而言，项目不多，第一第二方式也够了，但是万一后面做大了呢？