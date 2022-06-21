# mybatis中@Param的用法

## 一、方法有多个参数

```java
@Select("SELECT * from " + PUE_TABLE_NAME + " where resourceId = #{resourceId} and resourceType = #{resourceType} and removed =0")
    Pue selectByResourceIdAndResourceType(@Param("resourceId") int resourceId, @Param("resourceType") int resourceType);
```

## 二、方法参数要取别名

```java
@Select("SELECT * from " + PUE_TABLE_NAME + " where id = #{userId} and removed =0")
    Pue selectById(@Param("userId") int id);
```

## 三、 SQL 使用了 `$`

$ 会有注入漏洞的问题，但是有的时候你不得不使用 $ 符号，例如要传入列名或者表名的时候，这个时候必须要添加 @Param 注解

```java
@Select("select * from " + PUE_TABLE_NAME + " ORDER BY ${id} DESC")
    List<Pue> selectAllByName2(@Param("id") String id);
```

## 四、动态 SQL

如果在动态 SQL 中使用了参数作为变量，那么也需要 @Param 注解，即使你只有一个参数

```java
@Select("<script> SELECT * from " + PUE_TABLE_NAME +
            "<where>" +
            "<if test='name != null'> name like '%" + "${name}" + "%'</if>" +
            "  AND removed=0 " +
            "</where>  ORDER BY id  DESC </script>")
    List<Pue> selectAllByName(@Param("name")String name);
```

这种情况，即使只有一个参数，也需要添加 @Param 注解，而这种情况却经常被人忽略！