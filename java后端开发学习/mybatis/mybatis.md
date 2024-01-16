# mybatis基础学习

> 主要参考视频[黑马javaweb开发](https://www.bilibili.com/video/BV1m84y1w7Tb?p=132&vd_source=bfa68254fe5610a39db4d52bfba2d69a)

## springboot中mybatis的配置

我们需要在springboot的配置文件中对数据源以及mybatis进行相关配置

```properties
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/springboot
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=123456

#配置mybatis的日志, 指定输出到控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl

#开启mybatis的驼峰命名自动映射开关 a_column ------> aCloumn
mybatis.configuration.map-underscore-to-camel-case=true

#开启mybatis的别名映射
mybatis.type-aliases-package=com.itheima.pojo
```

其中由于实体类中属性的定义一般采用驼峰命名法，而数据表的属性名设置一般用下划线分隔。因此，需要开启mybatis的驼峰映射，从而实现属性对齐。

## 注解式mybatis开发

一般在项目的mapper目录下定义操作数据库的mapper接口,使用`@Mapper`注解表明为dao层的接口。
通过`@Select`、`@Update`等注解，结合相应的mysql语句，mybatis会自动实现相应的接口。

```java
@Mapper
public interface EmpMapper {
    @Delete("delete from emp where id = #{id}")
    public void delete(Integer id);

    /*
    useGeneratedKeys表示将插入记录的主键值返回
    keyProperty表示将返回的主键值存储到emp对象的id属性上
     */
    @Options(keyProperty = "id", useGeneratedKeys = true)
    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) " +
            "VALUES (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

    @Update("update emp set username=#{username}, name=#{name}, gender=#{gender}, image=#{image}," +
            "job=#{job}, entrydate=#{entrydate}, dept_id=#{deptId}, update_time=#{updateTime} where id=#{id}")
    public void update(Emp emp);
}
```

## mybatis的xml开发

使用注解式的开发可以帮助我们快速地实现简单的数据库操作，但如果涉及到复杂的处理，我们一般采用在xml中定义相关mapper的操作。
映射配置文件名与mapper接口名一致，且放在相同的包下。
其中`namespace`指代实现的mapper接口，而`<select>`中的`id`则指的是实现的方法。`resultType`一般定义为返回值的全限定名。我们可以开启mybatis的别名映射，从而使用别名来代替全限定名，默认别名是类名和类名首字母小写。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.EmpMapper">

    <sql id="commonSelect">
        select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
        from emp
    </sql>

<!--    resultType: 单条记录封装的类型-->
    <select id="list" resultType="com.itheima.pojo.Emp">
        <include refid="commonSelect"/>
        <where>
            <if test="name != null">
                name like concat('%', #{name}, '%')
            </if>
            <if test="gender != null">
                and gender = #{gender}
            </if>
            <if test="begin != null and end != null">
                and entrydate between #{begin} and #{end}
            </if>
        </where>
        order by update_time desc
    </select>
</mapper>
```

## 动态sql

### `<if>`标签

在mybatis映射文件中，通过\<if>标签可以实现根据传入数据动态增减sql语句

```xml
<select id="list" resultType="com.itheima.pojo.Emp">
        select * from emp
        <where>
            <if test="name != null">
                name like concat('%', #{name}, '%')
            </if>
            <if test="gender != null">
                and gender = #{gender}
            </if>
            <if test="begin != null and end != null">
                and entrydate between #{begin} and #{end}
            </if>
        </where>
        order by update_time desc
    </select>
```

当test条件成立时加入sql语句

### `<where>`标签

注意上面代码块中的\<where>标签，它可以包裹判断语句，起到sql语句中where的作用。同时，它还可以根据判断结果来删除语句中多余的“and”。

### `<set>`标签

与\<where>类似，它是update语句中“set”的标签形式，它可以根据判断结果删除多余的逗号。

```xml
<update id="update2">
    update emp
    <set>
        <if test="username != null">username = #{username},</if>
        <if test="name != null">name = #{name},</if>
        <if test="gender != null">gender = #{gender},</if>
        <if test="image != null">image = #{image},</if>
        <if test="job != null">job = #{job},</if>
        <if test="entrydate != null">entrydate = #{entrydate},</if>
        <if test="deptId != null">dept_id = #{deptId},</if>
        <if test="updateTime != null">update_time = #{updateTime}</if>
    </set>
    where id = #{id}
</update>
```

### `<foreach>`标签

```xml
<!--批量删除员工 (18,19,20)-->
<!--
    collection: 遍历的集合
    item: 遍历出来的元素
    separator: 分隔符
    open: 遍历开始前拼接的SQL片段
    close: 遍历结束后拼接的SQL片段
-->
<delete id="deleteByIds">
    delete  from emp where id in
    <foreach collection="ids" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</delete>
```

### `<sql>`和`<include>`标签

```xml
    <sql id="commonSelect">
        select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
        from emp
    </sql>

    <select id="list" resultType="com.itheima.pojo.Emp">
        <include refid="commonSelect"/>
        <where>
            <if test="name != null">
                name like concat('%', #{name}, '%')
            </if>
            <if test="gender != null">
                and gender = #{gender}
            </if>
            <if test="begin != null and end != null">
                and entrydate between #{begin} and #{end}
            </if>
        </where>
        order by update_time desc
    </select>
```