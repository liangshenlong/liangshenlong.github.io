---
title: 简单的aop应用2
date: 2018-03-31 14:50:45
tags: [aop,spring]
categories: [Java]
---
在利用aop记录job执行流水时，如果通过@Before、@After记录job的开始和结束，则必须保证job名字、方法、执行日期的唯一性，否则无法在在after中确定结束的job是哪一个。

一旦同一时间存在多个相同job处理的情况就会导致job结束时更新其他job的开始记录。
<!--more-->
这种情况可以使用@Around处理，代码如下：

```Java
@Service
@Aspect
@Lazy(false)
public class JobAspect {
    private static Logger logger = LoggerFactory.getLogger(JobAspect.class);

    @Autowired
    private JobRecordService jobRecordService;

    @Pointcut("execution(public * com.liangsl.app.job.*.execute(..))")
    public void execute(){}

    @Around("execute()")
    public void around(ProceedingJoinPoint proceedingJoinPoint){
        String jobId = IdGen.uuid();
        String class_name = proceedingJoinPoint.getTarget().getClass().getName();
        String method_name = proceedingJoinPoint.getSignature().getName();
        logger.info(class_name+" - "+method_name+" begin! jobId:["+jobId+"]");
        insertJobRecord(jobId,class_name,method_name);
        try {
            proceedingJoinPoint.proceed();
            updateJobRecord(jobId,"END");
        } catch (Throwable e) {
            logger.info(class_name+" - "+method_name+" error! jobId:["+jobId+"]",e);
            updateJobRecord(jobId,"ERROR");
        }

        logger.info(class_name+" - "+method_name+" end! jobId:["+jobId+"]");
    }

}
```

将proceedingJoinPoint.proceed();抛出的异常捕获，实现@AfterThrowing功能，但是要注意execute()方法中的异常逻辑，哪些该抛出，哪些需要自行捕获处理
