+++
date = '2025-03-26 15:22:47'
title = '一些做过的java场景题'
featuredImage= "https://img.picui.cn/free/2025/03/26/67e3ab905b219.jpg"

+++

# CompletableFuture之控制时间

## 题目

有一个消息发送接口MessageService.send(String message)，每次消息发送需要耗时2ms；

基于以上接口，实现一个批量发送接口MessageService.batchSend(List<String> messages);

要求如下：

1）一次批量发送消息最大数量为100条

2）批量发送接口一次耗时不超过50ms。

3）要求返回消息发送是否成功的结果。

## 解决



思路：将list进行分割，然后遍历分割后的list，创建任务，然后等全部执行完

> ```xml
> <dependency>
>     <groupId>com.google.guava</groupId>
>     <artifactId>guava</artifactId>
>     <version>30.1-jre</version>
> </dependency>
> ```
>
> 使用了guava进行list的分割

代码：

```java
package com.fang.other;

import com.google.common.collect.Lists;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * @author fang
 * @version 1.0
 * @date 2025/3/26 14:41:15
 */
public class Test {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            list.add(String.valueOf(i));
        }
//        Lists.partition(list, 3);
//        计算方法耗时
        long start = System.currentTimeMillis();
        batchSend(list);
        long end = System.currentTimeMillis();
        System.out.println("方法耗时：" + (end - start));
    }

//    线程池
    private static final ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 10, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(10));
    /**
     * 一次批量发送消息最大数量为100条
     * 批量发送接口一次耗时不超过50ms。
     *
     * @param messages
     * @return
     */
    public static boolean batchSend(List<String> messages) {
        List<List<String>> partition = Lists.partition(messages, 20);
        List<CompletableFuture> futures = new ArrayList<>();
        for (List<String> one : partition) {
            CompletableFuture<Boolean> future = CompletableFuture.supplyAsync(() -> {
                for (String res : one) {
                    if (!send(res)) return false;
                }
                return true;
            },executor);
            futures.add(future);

        }
        CompletableFuture<Void> allOf = CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]));
        try {
            allOf.get();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } catch (ExecutionException e) {
            throw new RuntimeException(e);
        }
        return true;
    }

    public static boolean send(String message) {
        try {
            Thread.sleep(2);
            return true;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

```

