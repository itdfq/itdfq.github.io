---
layout: posts
title: 分批删除Redis避免宕机
top: false
cover: false
toc: true
mathjax: true
date: 2024-04-03 14:40:58
password:
summary: 需要删除redis缓存中的数据,数据量比较大,需要分批处理,避免redis宕机
tags:
  - 中间件
  - redis
categories:
  - redis
---
## 分批删除Redis避免宕机

### 方式一

使用for循环分批去获取数据,去删除JOOM*的数据

```java
    @Test
    public void deleteTest()throws Exception{
        ScanOptions options = ScanOptions.scanOptions()
                .match("JOOM*")
                .count(1000)  // 每次迭代返回的 key 的数量
                .build();
        redisTemplate.executeWithStickyConnection((RedisCallback<Void>) connection -> {
            try {
                Cursor<byte[]> cursor = connection.scan(options);
                while (cursor.hasNext()) {
                    byte[] key = cursor.next();
                    // 处理 key
                    String keyStr = new String(key);

                    Long del = connection.del(key);
                    System.out.println("当前的key:"+keyStr+"结果："+del);
                }
                cursor.close();
                return null;
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
    }
```


## 方式二
使用多线程去优化

```java
package com.epean.trade.lms.tiktok;

import com.epean.trade.BaseTest;
import com.taobao.api.internal.util.NamedThreadFactory;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.springframework.data.redis.core.Cursor;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ScanOptions;

import javax.annotation.Resource;
import java.io.IOException;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @author dfq 2023/9/23 16:48
 * @implNote
 */
@Slf4j
public class TiktokDelteStockLockTest extends BaseTest {
    @Resource
    private RedisTemplate redisTemplate;

    public ThreadPoolExecutor executor = new ThreadPoolExecutor(10,10,0, TimeUnit.MINUTES,new LinkedBlockingQueue<>(1000),new NamedThreadFactory("tiktok删除标零补货分布式锁线程池"));

    @Test
    public void deleteTest()throws Exception{
        ScanOptions options = ScanOptions.scanOptions()
                .match("TIKTOK_UPDATE*")
                .count(10000)  // 每次迭代返回的 key 的数量
                .build();
        redisTemplate.executeWithStickyConnection((RedisCallback<Void>) connection -> {
            try {
                Cursor<byte[]> cursor = connection.scan(options);
                while (cursor.hasNext()) {
                    executor.submit(()->{
                        byte[] key = cursor.next();
                        // 处理 key
                        String keyStr = new String(key);
                        Long del = connection.del(key);
                    });
                }
                cursor.close();
                return null;
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
    }

}


```
