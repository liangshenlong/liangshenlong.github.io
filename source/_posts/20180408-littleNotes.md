---
title: mybatis和spring的一些笔记
date: 2018-04-08 16:37:56
tags: [mybatis,spring]
categories: [Java]
---
## mybatis批量处理
mybatis批量处理数据，数据库为oracle，使用foreach；
直接foreach separator=";"会报错，需要设置open，close属性
```Java
<update id="batchMerge" parameterType="java.util.List">
    <foreach collection="list" item="item" index="index" open="begin" close=";end;" separator=";" >
        merge into t_user_integral t
        using (select #{item.userId} user_id from dual) d
        on (t.user_id = d.user_id)
        when matched then
        update set t.curr_integral_val = t.curr_integral_val + #{item.currIntegralVal},
                   t.update_time = #{item.updateTime}
        when not matched then
        insert
        (t.id,
        t.user_id,
        t.curr_integral_val,
        t.used_integral_val,
        t.exp_integral_val,
        t.expiring_integral_val,
        t.create_time,
        t.flag)
        values
        (sys_guid(), d.user_id, #{item.currIntegralVal}, 0, 0, 0, #{item.updateTime}, '01')
    </foreach>
</update>
```
<!--more-->
或者将foreach放在merge的using语句中，但要保证on条件使用的字段结果唯一
```Java
<update id="batchMerge" parameterType="java.util.List">
	merge into t_user_integral t
    using (
	<foreach collection="list" item="item" index="index" separator="union all" >
        select #{item.userId} user_id from dual
    </foreach>
	) d
    on (t.user_id = d.user_id) --保证using查询的d.user_id不重复
    when matched then
    update set t.curr_integral_val = t.curr_integral_val + #{item.currIntegralVal},
               t.update_time = #{item.updateTime}
    when not matched then
    insert
    (t.id,
    t.user_id,
    t.curr_integral_val,
    t.used_integral_val,
    t.exp_integral_val,
    t.expiring_integral_val,
    t.create_time,
    t.flag)
    values
    (sys_guid(), d.user_id, #{item.currIntegralVal}, 0, 0, 0, #{item.updateTime}, '01')
</update>
```
## mybatis jdbcType
mybatis保存的时间丢失时分秒，原因是jdbcType使用了DATE；
如果要精确到时分秒需要使用TIMESTAMP

## static属性注入

pom.xml配置
``` Java
<profile>
    <id>local</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
		<!--ftp-->
		<p.jfFtp.ip>54.222.*.*</p.jfFtp.ip>
		<p.jfFtp.userName>userName</p.jfFtp.userName>
		<p.jfFtp.pwd>pwd</p.jfFtp.pwd>
		<p.jfFtp.port>21</p.jfFtp.port>
		<p.overFtp.ip>54.222.*.*</p.overFtp.ip>
		<p.overFtp.userName>userName</p.overFtp.userName>
		<p.overFtp.pwd>pwd</p.overFtp.pwd>
		<p.overFtp.port>21</p.overFtp.port>
    </properties>
</profile>
```
application.properties
``` Java
#ftp properties
p.jfFtp.ip = ${p.jfFtp.ip}
p.jfFtp.userName = ${p.jfFtp.userName}
p.jfFtp.pwd = ${p.jfFtp.pwd}
p.jfFtp.port = ${p.jfFtp.port}
p.overFtp.ip = ${p.overFtp.ip}
p.overFtp.userName = ${p.overFtp.userName}
p.overFtp.pwd = ${p.overFtp.pwd}
p.overFtp.port = ${p.overFtp.port}
```
spring-context.xml
``` Java
<util:properties id="APP_PROP" location="classpath:application.properties" local-override="true"/>
```
代码
``` Java
public static String jfIp;
public static String jfUserName;
public static String jfPassword;
public static int jfPort;
@Value("#{APP_PROP['p.jfFtp.ip']}")
public void setJfIp(String jfIp) {
    FtpConst.jfIp = jfIp;
}
@Value("#{APP_PROP['p.jfFtp.userName']}")
public void setJfUserName(String jfUserName) {
    FtpConst.jfUserName = jfUserName;
}
@Value("#{APP_PROP['p.jfFtp.pwd']}")
public void setJfPassword(String jfPassword) {
    FtpConst.jfPassword = jfPassword;
}
@Value("#{APP_PROP['p.jfFtp.port']}")
public void setJfPort(int jfPort) {
    FtpConst.jfPort = jfPort;
}
```

>注意把生成setter方法的static去掉
