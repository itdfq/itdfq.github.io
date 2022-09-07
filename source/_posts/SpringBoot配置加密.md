---
title: SpringBoot配置加密
top: false
cover: false
toc: true
mathjax: true
date: 2022-09-06 16:30:54
password:
summary: 本文主要介绍SpringBoot使用jasypt，对配置进行加密
tags:
- 原创
- java
categories:
- SpringBoot
---
### SpringBoot配置文件加密
---
#### 依赖
<!--        jasypt 配置加密-->
        <dependency>
            <groupId>com.github.ulisesbocchio</groupId>
            <artifactId>jasypt-spring-boot-starter</artifactId>
            <version>2.1.2</version>
        </dependency>
#### 设置秘钥
jasypt.encryptor.password=秘钥
开发环境可以直接在配置文件中指定，生产环境可以使用启动命理   --jasypt.encryptor.password=XXX 进行配置
####加密解密
```java
  package com.itdfq.springboot;

import org.jasypt.encryption.StringEncryptor;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

/**
 * @author duanfangqin 2022/7/11 10:56
 * @implNote
 */
@SpringBootTest
public class JasyptTest {
    @Autowired
    private StringEncryptor stringEncryptor;

    @Test
    public void encrypt(){

        System.out.println(stringEncryptor.encrypt("qweasd123"));
    }
    @Test
    public void decrypt(){
        System.out.println(stringEncryptor.decrypt("zI16ovOHuYiPIQaFx9cVlJF30bg5h3ql"));
    }
}
```
####配置文件密码设置
```properties
#Redis服务器地址
spring.redis.host=119.3.234.108
#服务器密码
spring.redis.password=ENC(nwElWn8r6aqVHDdAStIRiEXPAiE57qUF)
```
ENC(加密之后的密码)