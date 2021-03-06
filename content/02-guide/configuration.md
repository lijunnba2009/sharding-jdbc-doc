+++
toc = true
date = "2016-12-06T22:38:50+08:00"
title = "配置手册"
weight = 5
prev = "/02-guide/master-slave/"
next = "/02-guide/hint-sharding-value/"

+++

## YAML配置

## 引入maven依赖

```xml
<dependency>
    <groupId>com.dangdang</groupId>
    <artifactId>sharding-jdbc</artifactId>
    <version>${latest.release.version}</version>
</dependency>
```

### Java示例

```java
    DataSource dataSource = MasterSlaveDataSourceFactory.createDataSource(yamlFile);
```

### 配置示例

```yaml
dataSource:
  ds_default: !!org.apache.commons.dbcp.BasicDataSource
      driverClassName: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/ds_default
      username: root
      password: 
  ds_0: !!org.apache.commons.dbcp.BasicDataSource
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/ds_0
    username: root
    password: 
  ds_1: !!org.apache.commons.dbcp.BasicDataSource
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/ds_1
    username: root
    password: 
    
defaultDataSourceName: ds_default

tables:
  t_order: 
    actualDataNodes: ds_${0..1}.t_order_${0..1}
    tableStrategy: 
      inline:
        shardingColumn: id
        algorithmInlineExpression: t_order_${id % 2}
  t_order_item:
    actualDataNodes: ds_${0..1}.t_order_item_${0..1}
    tableStrategy: 
      inline:
        shardingColumn: id
        algorithmInlineExpression: t_order_item_${id % 2}

bindingTables:
  - t_order,t_order_item

defaultDatabaseStrategy:
  none:
  
defaultTableStrategy:
  none:
  
props:
  sql.show: true
```

### 配置项说明

```yaml
dataSource: 数据源配置
  <data_source_name> 可配置多个: !!数据库连接池实现类
    driverClassName: 数据库驱动类名
    url: 数据库url连接
    username: 数据库用户名
    password: 数据库密码
    ... 数据库连接池的其它属性
    
defaultDataSourceName: 默认数据源，未配置分片规则的表将通过默认数据源定位
  
tables: 分库分表配置，可配置多个logic_table_name
    <logic_table_name>: 逻辑表名
        actualDataNodes: 真实数据节点，由库名 + 表名组成，以小数点分隔。多个表以逗号分隔，支持inline表达式。不填写表示为只分库不分表。
        databaseStrategy: 分库策略，以下的分片策略只能任选其一
            standard: 标准分片策略，用于单分片键的场景
                shardingColumn: 分片列名
                preciseAlgorithmClassName: 精确的分片算法类名称，用于=和IN。该类需使用默认的构造器或者提供无参数的构造器
                rangeAlgorithmClassName: 范围的分片算法类名称，用于BETWEEN，可以不配置。该类需使用默认的构造器或者提供无参数的构造器
            complex: 复合分片策略，用于多分片键的场景
                shardingColumns : 分片列名，多个列以逗号分隔
                algorithmClassName: 分片算法类名称。该类需使用默认的构造器或者提供无参数的构造器
            inline: inline表达式分片策略
                shardingColumn : 分片列名
                algorithmExpression: 分库算法表达式，需要符合groovy动态语法
            hint: Hint分片策略
                algorithmClassName: 分片算法类名称。该类需使用默认的构造器或者提供无参数的构造器
            none: 不分片
        tableStrategy: 分表策略，同分库策略
  bindingTables: 绑定表列表
  - 逻辑表名列表，多个<logic_table_name>以逗号分隔
  
defaultDatabaseStrategy: 默认数据库分片策略，同分库策略
 
defaultTableStrategy: 默认数据表分片策略，同分库策略

props: 属性配置(可选)
    sql.show: 是否开启SQL显示，默认值: false
    executor.size: 工作线程数量，默认值: CPU核数
```

#### YAML格式特别说明
!! 表示实现类

& 表示变量定义

* 表示变量引用

- 表示多个

## Spring命名空间配置

### 引入maven依赖

```xml
<dependency>
    <groupId>io.shardingjdbc</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${latest.release.version}</version>
</dependency>
```

### 配置示例
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:sharding="http://shardingjdbc.io/schema/shardingjdbc/sharding" 
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context 
                        http://www.springframework.org/schema/context/spring-context.xsd 
                        http://shardingjdbc.io/schema/shardingjdbc/sharding 
                        http://shardingjdbc.io/schema/shardingjdbc/sharding/sharding.xsd 
                        ">
    <context:property-placeholder location="classpath:conf/rdb/conf.properties" ignore-unresolvable="true" />
    
    <bean id="dbtbl_0" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/dbtbl_0" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>
    
    <bean id="dbtbl_1" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/dbtbl_1" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>
    
    <sharding:standard-strategy id="databaseStrategy" sharding-column="user_id" precise-algorithm-class="io.shardingjdbc.spring.algorithm.PreciseModuloDatabaseShardingAlgorithm" />
    <sharding:standard-strategy id="tableStrategy" sharding-column="order_id" precise-algorithm-class="io.shardingjdbc.spring.algorithm.PreciseModuloTableShardingAlgorithm" />
    
    <sharding:data-source id="shardingDataSource">
        <sharding:sharding-rule data-source-names="dbtbl_0,dbtbl_1" default-data-source-name="dbtbl_0">
            <sharding:table-rules>
                <sharding:table-rule logic-table="t_order" actual-data-nodes="dbtbl_${0..1}.t_order_${0..3}" database-strategy-ref="databaseStrategy" table-strategy-ref="tableStrategy" />
                <sharding:table-rule logic-table="t_order_item" actual-data-nodes="dbtbl_${0..1}.t_order_item_${0..3}" database-strategy-ref="databaseStrategy" table-strategy-ref="tableStrategy" />
            </sharding:table-rules>
            <sharding:binding-table-rules>
                <sharding:binding-table-rule logic-tables="t_order, t_order_item" />
            </sharding:binding-table-rules>
        </sharding:sharding-rule>
        <sharding:props>
            <prop key="sql.show">true</prop>
        </sharding:props>
    </sharding:data-source>
</beans>
```
### 标签说明

#### \<sharding:data-source/\>

定义sharding-jdbc数据源

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*         |
| ----------------------------- | ------------ |  --------- | ------ | -------------- |
| id                            | 属性         |  String     |   是   | Spring Bean ID |
| sharding-rule                 | 标签         |   -         |   是   | 分片规则        |
| binding-table-rules?          | 标签         |   -         |   否   | 绑定表规则       |
| props?                        | 标签         |   -         |   否   | 相关属性配置     |

#### \<sharding:sharding-rule/>

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*                                                                |
| ----------------------------- | ------------ | ---------- | ------ | --------------------------------------------------------------------- |
| data-source-names             | 属性         | String      |   是   | 数据源Bean列表，多个Bean以逗号分隔                                        |
| default-data-source-name      | 属性         | String      |   否   | 默认数据源名称，未配置分片规则的表将通过默认数据源定位                        |
| default-database-strategy-ref | 属性         | String      |   否   | 默认分库策略，对应\<sharding:xxx-strategy>中的策略id，不填则使用不分库的策略 |
| default-table-strategy-ref    | 属性         | String      |   否   | 默认分表策略，对应\<sharding:xxx-strategy>中的策略id，不填则使用不分表的策略 |
| table-rules                   | 标签         |   -         |   是   | 分片规则列表                                                            |

#### \<sharding:table-rules/>

| *名称*                         | *类型*      | *数据类型*  |  *必填* | *说明*  |
| ----------------------------- | ----------- | ---------- | ------ | ------- |
| table-rule+                   | 标签         |   -        |   是  | 分片规则 |

#### \<sharding:table-rule/>

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*  |
| --------------------          | ------------ | ---------- | ------ | ------- |
| logic-table                   | 属性         |  String     |   是   | 逻辑表名 |
| actual-data-nodes             | 属性         |  String     |   否   | 真实数据节点，由库名 + 表名组成，以小数点分隔。多个表以逗号分隔，支持inline表达式。不填写表示为只分库不分表                        |
| database-strategy-ref         | 属性         |  String     |   否   | 分库策略，对应\<sharding:xxx-strategy>中的策略id，不填则使用\<sharding:sharding-rule/>配置的default-database-strategy-ref   |
| table-strategy-ref            | 属性         |  String     |   否   | 分表策略，对应\<sharding:xxx-strategy>中的略id，不填则使用\<sharding:sharding-rule/>配置的default-table-strategy-ref        |

#### \<sharding:binding-table-rules/>

| *名称*                         | *类型*      | *数据类型*  |  *必填* | *说明*  |
| ----------------------------- | ----------- |  --------- | ------ | ------- |
| binding-table-rule            | 标签         |   -         |   是  | 绑定规则 |

#### \<sharding:binding-table-rule/>

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*                   |
| ----------------------------- | ------------ | ---------- | ------ | ------------------------ |
| logic-tables                  | 属性         |  String     |   是   | 逻辑表名，多个表名以逗号分隔 |

#### \<sharding:standard-strategy/>

标准分片策略，用于单分片键的场景

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*                                                                |
| ----------------------------- | ------------ | ---------- | ------ | --------------------------------------------------------------------- |
| sharding-column               | 属性         |  String     |   是   | 分片列名                                                               |
| precise-algorithm-class       | 属性         |  String     |   是   | 精确的分片算法类名称，用于=和IN。该类需使用默认的构造器或者提供无参数的构造器   |
| range-algorithm-class         | 属性         |  String     |   否   | 范围的分片算法类名称，用于BETWEEN。该类需使用默认的构造器或者提供无参数的构造器 |

#### \<sharding:complex-strategy/>

复合分片策略，用于多分片键的场景

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*                                              |
| ----------------------------- | ------------ | ---------- | ------ | --------------------------------------------------- |
| sharding-columns              | 属性         |  String     |   是  | 分片列名，多个列以逗号分隔                              |
| algorithm-class               | 属性         |  String     |   是  | 分片算法全类名，该类需使用默认的构造器或者提供无参数的构造器 |

#### \<sharding:inline-strategy/>

inline表达式分片策略

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*       |
| ----------------------------- | ------------ | ---------- | ------ | ------------ |
| sharding-column               | 属性         |  String     |   是   | 分片列名      |
| algorithm-expression          | 属性         |  String     |   是   | 分片算法表达式 |

#### \<sharding:hint-database-strategy/>

Hint方式分片策略

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*                                              |
| ----------------------------- | ------------ | ---------- | ------ | --------------------------------------------------- |
| algorithm-class               | 属性         |  String     |   是  | 分片算法全类名，该类需使用默认的构造器或者提供无参数的构造器 |

#### \<sharding:none-strategy/>

不分片的策略

#### \<sharding:props/\>

| *名称*                                | *类型*       | *数据类型*  | *必填* | *说明*                              |
| ------------------------------------ | ------------ | ---------- | ----- | ----------------------------------- |
| sql.show                             | 属性         |  boolean   |   是   | 是否开启SQL显示，默认为false不开启     |
| executor.size                        | 属性         |  int       |   否   | 最大工作线程数量                      |

#### \<master-slave:data-source/\>

定义sharding-jdbc读写分离的数据源

| *名称*                         | *类型*       | *数据类型*  |  *必填* | *说明*                                   |
| ----------------------------- | ------------ |  --------- | ------ | ---------------------------------------- |
| id                            | 属性         |  String     |   是   | Spring Bean ID                           |
| master-data-source-name       | 标签         |   -         |   是   | 主库数据源Bean ID                         |
| slave-data-source-names       | 标签         |   -         |   是   | 从库数据源Bean列表，多个Bean以逗号分隔       |
| strategy-ref?                 | 标签         |   -         |   否   | 主从库复杂策略Bean ID，可以使用自定义复杂策略 |
| strategy-type?                | 标签         |  String     |   否   | 主从库复杂策略类型<br />可选值：ROUND_ROBIN, RANDOM<br />默认值：ROUND_ROBIN |

#### Spring格式特别说明
如需使用inline表达式，需配置ignore-unresolvable为true，否则placeholder会把inline表达式当成属性key值导致出错. 


## 分片算法表达式语法说明

### inline表达式特别说明
${begin..end} 表示范围区间

${[unit1, unit2, unitX]} 表示枚举值

inline表达式中连续多个${...}表达式，整个inline最终的结果将会根据每个子表达式的结果进行笛卡尔组合，例如正式表inline表达式如下：
```groovy
dbtbl_${['online', 'offline']}_${1..3}
```
最终会解析为dbtbl_online_1，dbtbl_online_2，dbtbl_online_3，dbtbl_offline_1，dbtbl_offline_2和dbtbl_offline_3这6张表。

### 字符串内嵌groovy代码
表达式本质上是一段字符串，字符串中使用${}来嵌入groovy代码。

```groovy 
data_source_${id % 2 + 1}
```

上面的表达式中data_source_是字符串前缀，id % 2 + 1是groovy代码。
