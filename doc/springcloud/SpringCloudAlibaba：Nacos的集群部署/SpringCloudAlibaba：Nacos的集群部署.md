# SpringCloudAlibaba：Nacos的集群部署

根据官方文档的介绍，Nacos的集群架构大致如下图所示（省略了集中化存储信息的MySQL）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/fbdad346f21f4f7fad511624431804be.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 集群搭建

### 1.环境准备

```
192.168.31.100:80  jdk1.8 mysql nginx
192.168.31.100:8848  jdk1.8 nacos
192.168.31.100:8849  jdk1.8 nacos
192.168.31.100:8850  jdk1.8 nacos
```

> 注意：这里的例子仅用于本地学习测试使用，实际生产环境必须部署在不同的节点上，才能起到高可用的效果。另外，Nacos的集群需要3个或3个以上的节点，并且确保这三个节点之间是可以互相访问的。

### 2.数据持久化

对于数据源的修改，在上一篇[《Nacos的数据持久化》](https://blog.csdn.net/weixin_43296313/article/details/120967493)中已经说明缘由，如果还不了解的话，可以先读一下这篇再回来看这里。

### 3.docker安装nginx，nacos

#### 1.创建Nacos的工作目录

```shell
mkdir -p nacos_8848/init.d nacos_8848/logs nacos_8848/env
mkdir -p nacos_8849/init.d nacos_8849/logs nacos_8849/env
mkdir -p nacos_8850/init.d nacos_8850/logs nacos_8850/env
```

#### 2.调整custom.properties

```properties
# 在每一个节点的init.d目录创建custom.properties文件，文件内容如下

// 添加以下配置
#spring.security.enabled=false
#management.security=false
#security.basic.enabled=false
#nacos.security.ignore.urls=/**
#management.metrics.export.elastic.host=http://localhost:9200
# metrics for prometheus
management.endpoints.web.exposure.include=*

# metrics for elastic search
#management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200

# metrics for influx
#management.metrics.export.influx.enabled=false
#management.metrics.export.influx.db=springboot
#management.metrics.export.influx.uri=http://localhost:8086
#management.metrics.export.influx.auto-create-db=true
#management.metrics.export.influx.consistency=one
#management.metrics.export.influx.compressed=true
```

#### 3.调整nacos-hostname.env

```properties
# 在每一个节点的env目录创建nacos-hostname.env文件，文件内容如下

#nacos dev env
# 首选主机模式
PREFER_HOST_MODE=hostname
# 当前主机的IP
NACOS_SERVER_IP=192.168.1.160
# 集群的各个节点
NACOS_SERVERS=192.168.1.160:8848 192.168.1.161:8848 192.168.1.162:8848
# 数据库的配置
MYSQL_SERVICE_HOST=192.168.1.100
MYSQL_SERVICE_DB_NAME=nacos
MYSQL_SERVICE_PORT=3306
MYSQL_SERVICE_USER=root
MYSQL_SERVICE_PASSWORD=root

# 从节点 这里就使用单节点测试,因此就不配置从节点
#MYSQL_SLAVE_SERVICE_HOST=xxx 
#MYSQL_SLAVE_SERVICE_PORT=3306

# JVM参数 默认是2G 如果使用虚拟机,内存没有2G,就需要调整这里的参数,否则将无法启动
JVM_XMS=256m
JVM_XMX=256m
JVM_XMN=256m
```

#### 4.docker启动(3个节点)

```shell
# 8848节点
docker run -d -p 8848:8848 \
         --env-file=/huanglei/v-nacos/nacos_cluster/nacos_8848/env/nacos-hostname.env \
        -v /huanglei/v-nacos/nacos_cluster/nacos_8848/init.d/custom.properties:/home/nacos/init.d/custom.properties \
        -v /huanglei/v-nacos/nacos_cluster/nacos_8848/logs:/home/nacos/logs \
         --restart=always \
         --name=nacos_8848 nacos/nacos-server
 
# 8849节点 
docker run -d -p 8849:8848 \
          --env-file=/huanglei/v-nacos/nacos_cluster/nacos_8849/env/nacos-hostname.env \
          -v /huanglei/v-nacos/nacos_cluster/nacos_8849/init.d/custom.properties:/home/nacos/init.d/custom.properties \
          -v /huanglei/v-nacos/nacos_cluster/nacos_8849/logs:/home/nacos/logs \
          --restart=always \
          --name=nacos_8849 nacos/nacos-server

# 8850节点
docker run -d -p 8850:8848 \
           --env-file=/huanglei/v-nacos/nacos_cluster/nacos_8850/env/nacos-hostname.env \
          -v /huanglei/v-nacos/nacos_cluster/nacos_8850/init.d/custom.properties:/home/nacos/init.d/custom.properties \
           -v /huanglei/v-nacos/nacos_cluster/nacos_8850/logs:/home/nacos/logs \
           --restart=always \
           --name=nacos_8850 nacos/nacos-server
           
# 注意：--env-file上面的这个必须用这个，不可以用-e-file（颜色是紫色才是正确的）          
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/950249f368234948985c5d80cd218923.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


#### 5.测试

这里访问8848、8849、8850端口都可以

![在这里插入图片描述](https://img-blog.csdnimg.cn/158148d1776b4f179208deaf3b4833cb.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 4.Nginx代理Nacos集群

在Nginx配置文件的http段中，我们可以加入下面的配置内容：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b5f40c147bac46e7b192461a21ea4b7b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_13,color_FFFFFF,t_70,g_se,x_16#pic_center)


这样，当我们访问：`http://localhost:8080/nacos/`的时候，就会被负载均衡的代理到之前我们启动的三个Nacos实例上了。这里我们没有配置`upstream`的具体策略，默认会使用线性轮训的方式，如果有需要，也可以配置上更为复杂的分发策略。这部分是Nginx的使用内容，这里就不作具体介绍了。