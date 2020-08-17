---
title: Mybatis的基础使用
tags: mybatis
categories: mybatis
abbrlink: 59904
date: 2020-08-11 11:53:39
---

mybatis是目前比较主流的ORM框架，今天对mybatis的一些基础用法做一总结记录。

# 常见入参方式

# resultMap的使用

## 简单解释

```xml
<!--下面的是抄来的-->
<!--column不做限制，可以为任意表的字段，而property须为type 定义的pojo属性-->
<resultMap id="唯一的标识" type="映射的pojo对象">
  <id column="表的主键字段，或者可以为查询语句中的别名字段" jdbcType="字段类型" property="映射pojo对象的主键属性" />
  <result column="表的一个字段（可以为任意表的一个字段）" jdbcType="字段类型" property="映射到pojo对象的一个属性（须为type定义的pojo对象中的一个属性）"/>
  <association property="pojo的一个对象属性" javaType="pojo关联的pojo对象">
    <id column="关联pojo对象对应表的主键字段" jdbcType="字段类型" property="关联pojo对象的主席属性"/>
    <result  column="任意表的字段" jdbcType="字段类型" property="关联pojo对象的属性"/>
  </association>
  <!-- 集合中的property须为oftype定义的pojo对象的属性-->
  <collection property="pojo的集合属性" ofType="集合中的pojo对象">
    <id column="集合中pojo对象对应的表的主键字段" jdbcType="字段类型" property="集合中pojo对象的主键属性" />
    <result column="可以为任意表的字段" jdbcType="字段类型" property="集合中的pojo对象的属性" />  
  </collection>
</resultMap>
```

## 例子

```xml
<resultMap type="com.xqh.railway.railwaybureau.domain.Project" id="projectMap">
        <result property="id" column="ID" jdbcType="INTEGER"/>
        <result property="lineId" column="LINE_ID" jdbcType="INTEGER"/>
        <result property="name" column="NAME" jdbcType="VARCHAR"/>
        <result property="type" column="TYPE" jdbcType="VARCHAR"/>
        <result property="projectType" column="PROJECT_TYPE" jdbcType="VARCHAR"/>
        <result property="status" column="STATUS" jdbcType="VARCHAR"/>
        <result property="startLength" column="START_LENGTH" jdbcType="NUMERIC"/>
        <result property="endLength" column="END_LENGTH" jdbcType="NUMERIC"/>
        <result property="createTime" column="CREATE_TIME" jdbcType="TIMESTAMP"/>
        <result property="modifyTime" column="MODIFY_TIME" jdbcType="TIMESTAMP"/>
        <result property="endTime" column="END_TIME" jdbcType="TIMESTAMP"/>
        <result property="plan" column="PLAN" jdbcType="NUMERIC"/>
        <result property="complete" column="COMPLETE" jdbcType="NUMERIC"/>
        <result property="planDay" column="PLAN_DAY" jdbcType="INTEGER"/>
        <result property="useDay" column="USE_DAY" jdbcType="INTEGER"/>
        <association property="line" javaType="com.xqh.railway.railwaybureau.domain.Line">
            <id column="LID" jdbcType="INTEGER" property="id"/>
            <result  column="LNAME" jdbcType="VARCHAR" property="name"/>
        </association>
        <collection property="testUnitList" ofType="com.xqh.railway.system.domain.Unit">
            <id column="UNIT_ID" jdbcType="INTEGER" property="unitId"/>
            <result column="UNIT_NAME" jdbcType="VARCHAR" property="unitName"/>
        </collection>
    </resultMap>
```

# mybatis中`<![CDATA[]]>`的作用

在使用mybatis 时我们sql是写在xml 映射文件中，如果写的sql中有一些特殊的字符的话，在解析xml文件的时候会被转义，但我们不希望他被转义，所以我们要使用<![CDATA[ ]]>来解决。

被`<![CDATA[]]>`这个标记所包含的内容将表示为**纯文本**，比如`<![CDATA[<]]>`表示文本内容`“<”`。
此标记用于xml文档中，我们先来看看使用转义符的情况。我们知道，在xml中，`”<”`、`”>”`、`”&”`等字符是不能直接存入的，否则xml语法检查时会**报错**，如果想在xml中使用这些符号，必须将其转义为实体，如`”&lt;”`、`”&gt;”`、`”&amp;”`，这样才能保存进xml文档。

在XML中，需要转义的字符有：

```
在XML中，需要转义的字符有：
&　　　&amp;
<　　　&lt;
>　　　&gt;
＂　　　&quot;
＇　　　&apos;
```

`<![CDATA[]]>`和xml转移字符的关系，它们两个看起来是不是感觉功能重复了？
　　是的，它们的功能就是一样的，只是应用场景和需求有些不同：
　　(1)`<![CDATA[]]>`不能适用所有情况，转义字符可以；
　　(2) 对于短字符串`<![CDATA[]]>`写起来啰嗦，对于长字符串转义字符写起来可读性差；
　　(3) `<![CDATA[]]>`表示xml解析器忽略解析，所以更快。

注意：

1. 此部分不能再包含`”]]>”`；
2. 不允许嵌套使用；
3. `”]]>”`这部分不能包含空格或者换行。
4. mybatis中的`<if test=""></if>、<where></where>、<choose></choose>、<trim></trim>` 等这些标签不能写到CDATA中。否则标签将不会被mybatis解析。