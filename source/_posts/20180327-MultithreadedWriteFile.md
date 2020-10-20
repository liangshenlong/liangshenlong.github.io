---
title: 多线程读写文件
date: 2018-03-27 11:25:54
tags: [多线程,io]
categories: [Java]
---
定时任务处理一个保存千万行数据的文件，需对其中每一行数据进行分析，根据分析结果，错误数据写入错误文件，有用数据写入有用文件同时对数据库操作，无用数据写入无用文件。

严格来讲并不是多线程读写，只是在数据处理和数据写入时通过多个线程完成。

此外，在数据写入的时候也未使用锁，开始担心数据被覆盖，但是测试结果没有问题。也可能是个隐藏的坑o(╥﹏╥)o


### 读文件
```java
@Test
public void readFile() {
    long begin = System.currentTimeMillis();
    List<String> list = new ArrayList<>();
    LineIterator lineIterator = null;
    try{
        lineIterator = FileUtils.lineIterator(new File(file));
        while (lineIterator.hasNext()){
            if(queue.size()>5){
                System.out.println("-----------------Thread sleep-----------------");
                Thread.sleep(1000);
            }
            String line = lineIterator.nextLine();
            if(list.size()>1999){

                List<String> t = new ArrayList<>();
                org.apache.commons.collections.CollectionUtils.addAll(t,  new  Object[list.size()]);
                Collections.copy(t, list);
                handle(t);
                list.clear();
                list = new ArrayList<>();
            }
            list.add(line);
        }
        if(!CollectionUtils.isEmpty(list)){
            handle(list);
        }
    }catch (Exception e){
        e.printStackTrace();
    }finally {
        LineIterator.closeQuietly(lineIterator);
    }
    long end = System.currentTimeMillis();
    System.out.println(String.valueOf(end-begin));
}

```

<!--more-->

*FileUtils.lineIterator(new File(file));*读取文件，返回一个LineIterator对象。LineIterator保存了文件的bufferedReader，并实现了Iterator接口。
![](http://ww1.sinaimg.cn/large/0069K7HCly1fprevoifyej30t208t3yp.jpg)

queue和线程池的配置如下：
```java
private BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>();
private ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(20, 30,
                1L, TimeUnit.MINUTES,queue);
```

queue是无限队列，读取文件速度远远大于数据处理的速度，queue中等待线程过多会导致内存溢出。因此在queue长度大于5时Thread.sleep()

### 处理数据
```java
public void handleList(List<String> list){
    threadPoolExecutor.submit(() -> {
        List<NonUserIntegralDto> nonUserIntegralDtos = new IntegralFileVerifyHandler().verify(list,fileName,rate,value);
        //list中全部的id
        Set<String> ids = nonUserIntegralDtos.stream().map(NonUserIntegralDto::getId).collect(Collectors.toSet());
        //根据id查询用户表
        List<UserBase> userBaseList = userBaseService.selectByidNos(ids);
        Map<String,UserBase> userBaseMap= userBaseList.stream().collect(Collectors.toMap(UserBase::getIdNo,Function.identity()));

        //id不存在系统中的list
        List<NonUserIntegralDto> idNotExistList = nonUserIntegralDtos.stream().filter(nonUserIntegral -> null == userBaseMap.get(nonUserIntegral.getId())).collect(Collectors.toList());

        //id存在系统中的list
        List<NonUserIntegralDto> idExistList = nonUserIntegralDtos.stream().
                filter(nonUserIntegral -> null != userBaseMap.get(nonUserIntegral.getId()))
                .peek(nonUserIntegralDto -> {
                    String idNo = nonUserIntegralDto.getId();
                    UserBase userBase= userBaseMap.get(idNo);
                    nonUserIntegralDto.setUserId(userBase.getId());
                }).collect(Collectors.toList());

        handleExistList(idExistList);
        handleNotExistList(idNotExistList);
    });
}

```
使用线程池threadPoolExecutor处理list

IntegralFileVerifyHandler.verify()校验数据，并将解析异常数据保存error文文件夹下

handleExistList、handleNotExistList分别处理有用数据和无用数据，同时写文件
### 写数据

```Java
try {
    FileUtils.writeLines(new File(FileInfoConst.integralFileErrorPath+fileName),errorLines,true);
} catch (IOException e) {
    logger.error("数据写入文件异常",e);
}
```
*FileUtils.writeLines*直接将list追加写入到文件。

多线程写数据到同一个错误数据文件、有用数据文件、无用数据文件。
经过测试，这种方式不会导致数据丢失。（也可能是没被我发现＿φ( °-°)/ ）
