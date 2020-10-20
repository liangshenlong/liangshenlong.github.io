---
title: 简单的aop应用
date: 2018-03-29 16:50:21
tags: [aop,spring]
categories: [Java]
---
一个简单的aop应用，实现保存定时任务的执行记录。

首先在xml文件中开启aop自动代理
```Java
<!-- 定义spring的aop支持  强制spring 使用CGLIB来生成AOP代理类-->
<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>
```
<!--more-->
JobAspect：
```Java
/**
 * job任务执行aop，记录job执行流水信息
 * Created by liangsl on 2018/3/29
 */
@Service
@Aspect
public class JobAspect {
    private static Logger logger = LoggerFactory.getLogger(JobAspect.class);

    @Autowired
    private JobRecordService jobRecordService;

    @Pointcut("execution(public * com.liangsl.app.job.*.execute(..))")
    public void execute(){}

    @Before("execute()")
    public void before(JoinPoint point){
        String class_name = point.getTarget().getClass().getName();
        String method_name = point.getSignature().getName();

        //db 插入job执行记录

        logger.info(class_name+" - "+method_name+" before ");
    }

    @AfterReturning("execute()")
    public void afterReturning(JoinPoint point){
        String class_name = point.getTarget().getClass().getName();
        String method_name = point.getSignature().getName();
        //db 更新job执行记录
        logger.info(class_name+" - "+method_name+" afterReturning ");
    }

    @AfterThrowing(pointcut = "execute()",throwing = "ex")
    public void afterThrowing(JoinPoint point,Throwable ex){
        String class_name = point.getTarget().getClass().getName();
        String method_name = point.getSignature().getName();
        //db 更新job执行记录
        logger.info(class_name+" - "+method_name+" afterThrowing :"+ex.toString());
    }
}


```

**这里需要注意*@Aspect*不会被spring管理，需要增加注解*@Service***


Pointcut中execution说明
![](https://upload-images.jianshu.io/upload_images/44770-67db7001e9c5e9e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/631)
@After为无论方法是否出现异常都执行，@AfterReturning只有正常返回时执行

使用@AfterReturning、@AfterThrowing分情况更新任务记录



参考：
[spring in action](https://www.jianshu.com/p/8b95db8d7a1f)
[JoinPoint](https://blog.csdn.net/qq_32719003/article/details/71404428)
