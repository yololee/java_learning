# mybatis注解版

## 一、批量操作

### 1.批量查询

```java
@Select("<script> SELECT * FROM " + FLASH_SHOCK_RULE_TABLE_NAME +
            " WHERE id in " +
            " <foreach collection='list' item='id' index='index' open='(' separator=',' close=')' >" +
            " #{id}" +
            " </foreach>" +
            " AND removed = 0 " +
            " </script>")
    List<FlashShockRule> selectByIds(@Param("list") List<Integer> ids);
```

### 2.批量插入

```java
@Insert({
            "<script>",
            "INSERT INTO " + FLASH_SHOCK_RULE_TABLE_NAME + " (name,description,state,level,timeCondition,flashState,flashPeriod,flashAction,shockState," +
                    "shockTriggerPeriod,shockTriggerCount,shockEndPeriod,shockEndCount,shockStrategy,sourceAlarmLevel,priority,createTime,updateTime,removed) VALUES",
            "<foreach collection='list' item='item' index='index' separator=','>",
            "( #{item.name},#{item.description},#{item.state},#{item.level},#{item.timeCondition},#{item.flashState},#{item.flashPeriod},#{item.flashAction},#{item.shockState}," +
                    "#{item.shockTriggerPeriod},#{item.shockTriggerCount},#{item.shockEndPeriod},#{item.shockEndCount},#{item.shockStrategy},#{item.sourceAlarmLevel}," +
                    "#{item.priority},#{item.createTime},#{item.updateTime},#{item.removed})",
            "</foreach>",
            "</script>"
    })
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
    void insertList(@Param("list")List<FlashShockRule> flashShockRuleAddList);
```

### 3.批量更新(1)

```java
@Update("<script>" +
            "<foreach collection = 'list' item ='item' open='' close='' separator=';'>" +
            "update flash_shock_rule set   name=#{item.name},description=#{item.description},state=#{item.state},level=#{item.level}," +
            " timeCondition=#{item.timeCondition},flashState=#{item.flashState},flashPeriod=#{item.flashPeriod},flashAction=#{item.flashAction}," +
            " shockState=#{item.shockState},shockTriggerPeriod=#{item.shockTriggerPeriod},shockTriggerCount=#{item.shockTriggerCount},shockEndPeriod=#{item.shockEndPeriod}," +
            " shockEndCount=#{item.shockEndCount},shockStrategy=#{item.shockStrategy},sourceAlarmLevel=#{item.sourceAlarmLevel},priority=#{item.priority}," +
            " updateTime=#{item.updateTime} where id =#{item.id} and removed=0 " +
            "</foreach>" +
            "</script>")
    void updateBatch(@Param("list")List<FlashShockRule> flashShockRuleUpdateList);
```

### 4.批量更新(2)

```java
@Update({"<script> UPDATE " + FLASH_SHOCK_RULE_TABLE_NAME + " SET  state =#{state},updateTime=#{updateTime}"
            + " WHERE removed=0 and id IN "
            + "<foreach  collection = 'ids' item = 'id' index = 'index' open = '(' separator= ',' close = ')' >"
            + "	#{id} "
            + "</foreach>"
            + "</script>"})
    void updateBatchStateById(@Param("ids") List<Integer> ids, @Param("state") Integer state,@Param("updateTime") Long updateTime);
```

### 5.批量删除

```java
@Delete({"<script> DELETE FROM " + DYNAMIC_RING_HOST_TABLE_NAME + " WHERE id IN "
            + "<foreach  collection = 'ids' item = 'id' index = 'index' open = '(' separator= ',' close = ')' >"
            + "	#{id} "
            + "</foreach>"
            + "</script>"})
    void batchDeleteDynamicRingHost(@Param("ids") List<Integer> ids);
```

## 二、常规操作

### 1.根据id查询

```java
@Select("SELECT * from " + FLASH_SHOCK_RULE_TABLE_NAME + " where id=#{id} and removed=0")
    FlashShockRule selectById(@Param("id")Integer id);
```

### 2.根据id更新

```java
@Update("update " + FLASH_SHOCK_RULE_TABLE_NAME + " set timeCondition=#{timeCondition} where id =#{id} and removed=0")
    void updateByTimeCondition(FlashShockRule flashShockRule);
```

### 3.查询数量

```java
@Select("select count(*) from " + FLASH_SHOCK_RULE_TABLE_NAME + " where name=#{name} and removed=0")
    int selectByName(@Param("name") String name);
```

### 4.查询除自己之外的数量

```java
@Select("select count(*) from " + FLASH_SHOCK_RULE_TABLE_NAME + " where id!=#{id} and name=#{name} and removed=0")
    int selectByIdAndName(@Param("id") int id, @Param("name") String name);
```

### 5.分页查询

```java
@Select("<script> SELECT * from " + DYNAMIC_RING_HOST_TABLE_NAME +
            "<where>" +
            "<if test='item.name != null'> name like '%" + "${item.name}" + "%'</if>" +
            "<if test='item.hostGroupId != null'> AND hostGroupId = #{item.hostGroupId} </if>" +
            "</where>  ORDER BY id  DESC </script>")
    List<DynamicRingHost> searchDynamicRingHostByDynamicRingHostQuery(@Param("item") DynamicRingHostQuery dynamicRingHostQuery);
```

### 6.分页查询(先按状态排序，在按更新时间)

```java
@Select("<script> select * from flash_shock_rule AS fs1 " +
            "<where>" +
            "<if test='item.name != null'> name like '%" + "${item.name}" + "%'</if>" +
            "(select count(1) from flash_shock_rule AS fs2 where fs1.state = fs2.state)" +
            "AND removed = 0" +
            "</where> ORDER BY state ASC , updateTime DESC" +
            "</script>")
    List<FlashShockRule> selectFlashShockByFlashShockRuleQuery(@Param("item") FlashShockRuleQuery flashShockRuleQuery);
```

### 7.多条件查询

```java
@Select("<script> SELECT * from " + ALARM_TABLE_NAME +
            "<where>" +
            "<if test='item.name != null'> name like '%" + "${item.name}" + "%'</if>" +
            "<if test='item.componentId != null'> AND componentId = #{item.componentId} </if>" +
            "<if test='item.componentInstanceId != null'> AND componentInstanceId = #{item.componentInstanceId} </if>" +
            "<if test='item.levelListStr != null'> AND level in ${item.levelListStr} </if>" +
            "<if test='item.alarmStateList != null'> AND  CONCAT(confirmState,handState) in  ${item.alarmStateListStr} </if>" +
            "<if test='item.latestHappenStartTime != null' > AND latestHappenTime <![CDATA[>=]]> #{item.latestHappenStartTime}</if>" +
            "<if test='item.latestHappenEndTime != null' > AND latestHappenTime <![CDATA[<=]]> #{item.latestHappenEndTime}</if>" +
            "<if test='item.handStartTime != null' > AND handTime <![CDATA[>=]]> #{item.handStartTime}</if>" +
            "<if test='item.handEndTime != null' > AND handTime <![CDATA[<=]]> #{item.handEndTime}</if>" +
            "<if test='item.shockMark != null'> AND shockMark = #{item.shockMark} </if>" +
            "</where>  ORDER BY id  DESC </script>")
    List<Alarm> searchListByName(@Param("item") SearchCurrentAlarmQuery searchQuery);
```

### 8.更新操作

```java
@Update("update " + FLASH_SHOCK_RULE_TABLE_NAME + " set name=#{name},description=#{description},state=#{state},level=#{level}," +
            " timeCondition=#{timeCondition},flashState=#{flashState},flashPeriod=#{flashPeriod},flashAction=#{flashAction},shockState=#{shockState}," +
            " shockTriggerPeriod=#{shockTriggerPeriod},shockTriggerCount=#{shockTriggerCount},shockEndPeriod=#{shockEndPeriod},shockEndCount=#{shockEndCount}," +
            " shockStrategy=#{shockStrategy},sourceAlarmLevel=#{sourceAlarmLevel},priority=#{priority},updateTime=#{updateTime} where id =#{id} and removed=0")
    void updateFlashShockRule(FlashShockRule flashShockRule);
```

### 9.添加操作

```java
@Insert("INSERT INTO " + FLASH_SHOCK_RULE_TABLE_NAME + " ( name,description,state,level,timeCondition,flashState,flashPeriod,flashAction," +
            "shockState,shockTriggerPeriod,shockTriggerCount,shockEndPeriod,shockEndCount,shockStrategy,sourceAlarmLevel,priority,createTime,updateTime,removed) values ( " +
            " #{name},#{description},#{state},#{level},#{timeCondition},#{flashState},#{flashPeriod},#{flashAction},#{shockState}," +
            "#{shockTriggerPeriod},#{shockTriggerCount},#{shockEndPeriod},#{shockEndCount},#{shockStrategy},#{sourceAlarmLevel},#{priority},#{createTime},#{updateTime},#{removed})" )
    @Options(useGeneratedKeys = true,keyProperty = "id",keyColumn = "id")
    void addFlashShockRule(FlashShockRule flashShockRule);
```