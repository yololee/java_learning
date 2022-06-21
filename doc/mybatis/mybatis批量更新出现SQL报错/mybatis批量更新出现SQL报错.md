# mybatis批量更新出现SQL报错

## 一、问题重现

### 1.配置文件

```yaml
spring:
  #DataSource数据源
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/mybatis_test?useSSL=false&amp
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver

#MyBatis配置
mybatis:
  type-aliases-package: com.hl.mybatis.pojo #别名定义
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #指定 MyBatis 所用日志的具体实现，未指定时将自动查找
    map-underscore-to-camel-case: true #开启自动驼峰命名规则（camel case）映射
    lazy-loading-enabled: true #开启延时加载开关
    aggressive-lazy-loading: false #将积极加载改为消极加载（即按需加载）,默认值就是false
    lazy-load-trigger-methods: "" #阻挡不相干的操作触发，实现懒加载
    cache-enabled: true #打开全局缓存开关（二级环境），默认值就是true
```

### 2.sql

```java
@Update({"<script>" +
            "<foreach item='item' collection='list' index='index' open='' close='' separator=';'>" +
            " UPDATE tb_user " +
            "<set>" +
            "<if test='item.userAccount != null'>user_account = #{item.userAccount},</if>" +
            "<if test='item.userPassword != null'>user_password=#{item.userPassword}</if>" +
            "</set>" +
            " WHERE user_id = #{item.userId} " +
            "</foreach>" +
            "</script>"})
    int updateBatch(@Param("list")List<UserInfo> userInfoList);
```

### 3.测试

查看控制台错误

![在这里插入图片描述](https://img-blog.csdnimg.cn/c0fb2d732e54400489b09b3f68c206d1.png#pic_center)


<font color ='red'>发现这里告诉我有一个语法错误，然后发现user_id，有一个符号。</font>>

这里经过测试更新一条是成功的

![在这里插入图片描述](https://img-blog.csdnimg.cn/58dc8dee6ae7447ba81a62096745ccf4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 二、问题分析

<font color='red'>Mybatis映射文件中的sql语句默认是不支持以" ; " 结尾的，也就是不支持多条sql语句的执行</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/426d24f337a34c7684b8df7239ef0839.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


但是在SQL编辑器中执行多条sql语句的时候是可以以分号结尾的，如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/018ccff64e4643d08424056ba49b6a34.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 三、解决方法

在application.properties配置文中的数据源url后面添加一个参数

<font color='red'>&allowMultiQueries=true</font>【允许sql语句中有多个insert或者update语句 == 支持sql批量操作】

原来的配置文件：

```yaml
url: jdbc:mysql://127.0.0.1:3306/mybatis_test?useSSL=false&amp
```

现在的配置文件

```yaml
url: jdbc:mysql://127.0.0.1:3306/mybatis_test?useSSL=false&amp&&allowMultiQueries=true
```

> 再次测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/ceba66955c30497193ca5b7d579c2ed5.png#pic_center)