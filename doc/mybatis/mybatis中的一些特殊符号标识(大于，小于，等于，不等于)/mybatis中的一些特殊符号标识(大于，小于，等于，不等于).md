# mybatis中的一些特殊符号标识(大于，小于，等于，不等于)

| 特殊字符 | xml中代替符号 | `<![CDATA[]]>`                           |
| -------- | ------------- | ---------------------------------------- |
| **&**    | `&amp;`       |                                          |
| **<**    | `&lt;`        | `a<![CDATA[<]]>b`                        |
| **>**    | `&gt;`        | `a<![CDATA[>]]>b`                        |
| **"**    | `&quot;`      |                                          |
| **'**    | `&apos`       |                                          |
| **a<=b** | `a&lt;=b`     | `a<![CDATA[<=]]>b`                       |
| **a>=b** | `a&gt;=b`     | `a<![CDATA[>=]]>b`                       |
| **a!=b** | `a!=b`        | `a<![CDATA[<>]]>b`或者`a<![CDATA[!=]]>b` |

> 案例

```java
@Select("<script> SELECT * from " + ALARM_TABLE_NAME +
            "<where>" +
            "<if test='item.name != null'> name like '%" + "${item.name}" + "%'</if>" +
            "<if test='item.levelListStr != null'> AND level in ${item.levelListStr} </if>" +
            "<if test='item.alarmStateList != null'> AND  CONCAT(confirmState,handState) in  ${item.alarmStateListStr} </if>" +
            "<if test='item.handStartTime != null' > AND handTime <![CDATA[>=]]> #{item.handStartTime}</if>" +
            "<if test='item.handEndTime != null' > AND handTime <![CDATA[<=]]> #{item.handEndTime}</if>" +
            "</where>  ORDER BY id  DESC </script>")
    List<Alarm> searchListByName(@Param("item") SearchCurrentAlarmQuery searchQuery);

```