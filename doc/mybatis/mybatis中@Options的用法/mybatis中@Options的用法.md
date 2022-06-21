# Mybatis中@Options注解的使用场景

### 场景一：查询数据时

```java
@Select("select * from " + DYNAMIC_RING_HOST_TABLE_NAME + " where id=#{id}")
    @Options(useCache = true,flushCache = Options.FlushCachePolicy.FALSE,timeout = 10000)
    DynamicRingHost selectById(@Param("id") int id);
```

```java
 @Options(useCache = true,flushCache = Options.FlushCachePolicy.DEFAULT,timeout = 10000)

- useCache = true
    -true 表示将会缓存本次查询结果，以提高下次查询速度 默认为true
- flushCache = Options.FlushCachePolicy.FALSE
    -表示查询时不刷新缓存
- timeout = 10000
    -查询结果缓存10000秒
```



### 场景二：插入数据时

dynamic_ring_host表有一个id自增长主键，如何在插入数据后自动获取到该主键值呢？可以使用[@Options](http://www.51code.com/)注解：

```java
@Insert("INSERT INTO " + DYNAMIC_RING_HOST_TABLE_NAME + " (name,ipAddr,port,snmpProtocol,username,password,hostGroupId,state,createTime,updateTime) values " +
            " (#{name},#{ipAddr},#{port},#{snmpProtocol},#{username},#{password},#{hostGroupId},#{state},#{createTime},#{updateTime})")
    @Options(useGeneratedKeys = true,keyProperty = "id",keyColumn = "id")
    void addDynamicRingHost(DynamicRingHost dynamicRingHost);
```

设置@Options属性userGeneratedKeys的值为true，并指定实例对象中主键的属性名keyProperty以及在数据库中的字段名keyColumn。这样在dynamic_ring_host插入数据后，id属性会被自动赋值。

```java
@Options(useGeneratedKeys = true,keyProperty = "id",keyColumn = "id")
- useGeneratedKeys = true
    -true 自动递增 默认为false
- keyProperty = "id"
    -id 这个id为实体类中的id
- keyColumn = "id"
  	-id 这个id为数据库中的字段id  
```

当然flushCache 仍然可以设置，表示插入数据后是否更新缓存，默认是true。