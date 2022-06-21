# mybatis的逆向工程

### 1.导入依赖

```xml
<dependencies>
    <!-- mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.46</version>
    </dependency>
</dependencies>
```

### 2.引入插件

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.2</version>
        <!--设置配置文件的位置-->
        <configuration>
          <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
          <overwrite>true</overwrite>
          <verbose>true</verbose>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.18</version>
          </dependency>
        </dependencies>
      </plugin>
    </plugins>
</build>
```

### 3.mybatis逆向工程的核心配置文件

一般叫==generatorConfig.xml==，在resources下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>
    <!--配置文件-->
    <properties resource="config.properties"/>


    <!--产生最简单的sql  CRUD-->
    <context id="Mysql" targetRuntime="MyBatis3simple">
        <!--<context id="Mysql" targetRuntime="MyBatis3">-->
        <!--产生序列化  hashcode    toString 方法-->
        <!--<plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"></plugin>-->
        <!--<plugin type="org.mybatis.generator.plugins.ToStringPlugin"></plugin>-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--数据库的连接-->
        <jdbcConnection driverClass="${driverClassName}"
                        connectionURL="${jdbc_url}"
                        userId="${jdbc_username}"
                        password="${jdbc_password}"/>

        <!--指定生成的类型为java类型，避免数据库中number等类型字段 false时解析成Integer  true时解析成BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!-- 指定生成pojo的包和此包在项目中的地址； -->
        <javaModelGenerator targetPackage="com.itheima.domain"
                            targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true"/>
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- 指定生成pojo的映射xml文件的所在包和此包在项目中的地址 -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!-- 指定生成dao接口的所在包,以及包在项目中的地址 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.itheima.dao"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>


        <!-- 配置表名跟pojo名  该table节点可以多个 -->
        <table tableName="ss_role" domainObjectName="Role"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
        <table tableName="ss_user" domainObjectName="User"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
    </context>
</generatorConfiguration>
```

### 4.jdbc连接参数

==config.properties==

```xml
driverClassName=com.mysql.jdbc.Driver
jdbc_url=jdbc:mysql:///db_heima_mm?characterEncoding=utf8&serverTimezone=UTC
jdbc_username=root
jdbc_password=root
```

==注意：连接数据库的时候必须要有设置字符编码和时区==