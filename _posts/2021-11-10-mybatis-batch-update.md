---
layout: post
title: 【Mybatis】批量更新sql语句
subtitle:  
tags: [mybatis, sql]
---

在 MyBatis 里面，如果想批量更新一个 Map ，可以使用 update 标签来定义 SQL 更新语句，然后在 foreach 标签中遍历 Map ，将 Map 对象中的每个键值对作为更新语句的参数传入。

xml文件中的动态SQL 如下：
```xml
<update id="batchUpdateSystemMarkSerial" parameterType="java.util.Map">
    <foreach collection="map.entrySet()" index="key" item="value" separator=";">
      update order_information
      set system_mark_serial = #{value}
      where id = #{key}
    </foreach>
</update>
```

mapper 文件中的接口定义如下：
```java
/**
 * 批量更新系统标签字段
 * @param key 订单id value 对应的系统标签值
 */  
int batchUpdateSystemMarkSerial(@Param("map") Map<Integer, String> idAndNewSystemMarkSerialMap);
```

测试代码
```java
@SpringBootTest
class XxlJobTest {

    @Autowired
    private OrderInformationMapper orderInformationMapper;

    @Test
    void testBatchUpdateSystemMarkSerial() {
        Map<Integer, String> idAndSystemMarkSerialMap = new HashMap<>();
        idAndSystemMarkSerialMap.put(595938, "1,3,4");
        idAndSystemMarkSerialMap.put(595939, "2,5,7,8");
        orderInformationMapper.batchUpdateSystemMarkSerial(idAndSystemMarkSerialMap);
    }

}
```

查看数据库，发现数据已经成功更新。

更细致的用法参考 [Mybatis 官网中的 Dynamic SQL](https://mybatis.org/mybatis-3/dynamic-sql.html)
