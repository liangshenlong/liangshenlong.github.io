---
title: idea junit时关于maven profiles的选择
date: 2018-03-31 10:26:46
tags: [maven,junit,idea]
categories: [Java]
---
为方便打包，maven pom中配置了profiles
```java
<profiles>
	 <profile>
        <id>local</id>
        <activation>
          <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
          <p.jdbc.main.url>local jdbc url</p.jdbc.main.url>
        </properties>
    </profile>
    <profile>
        <id>sit</id>
        <properties>
          <p.jdbc.logger.url>sit jdbc url</p.jdbc.logger.url>
        </properties>
    </profile>
    <profile>
        <id>uat</id>
        <properties>
          <p.jdbc.logger.url>uat jdbc url</p.jdbc.logger.url>
        </properties>
    </profile>
</profiles>
```
可以看出默认使用的是local

打包的时候只需要命令修改即可，如：
>#sit 打包命令
>
>clean install tomcat7:redeploy -P sit -Dmaven.test.skip=true


<!--more-->
然而今天在junit的时候发现连接的jdbc是sit，junit代码如下
```Java
@ContextConfiguration(locations = "classpath:spring/spring-context.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class SysParamTest {
  @Test
  public void testJob() throws InterruptedException {
    System.out.println("--------------------------test Job");
    Thread.sleep(1000000);
  }
}
```

网上搜一下说在测试类上添加@ActiveProfiles("local")，试了一下没有用

最终发现是因为同事把项目编译后的target提交了，而我更新之后就使用sit的jdbc

maven clean一下，果然有效

那么问题来了，项目中有多个profile时，junit如何选择不同的profile呢

试着改了下pom中的<activation>，无效！！clean之后也无效！！

最终发现需要修改Maven Projects中profiles
![](http://ww1.sinaimg.cn/large/0069K7HCly1fpvu9g60yfj30em0e2wfb.jpg)

勾选不同的profile，clean，run test，成功
