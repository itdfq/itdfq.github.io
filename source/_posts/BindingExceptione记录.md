---
title: "BindingExceptioné记录"
top: false
cover: false
toc: true
mathjax: true
date: 2022-09-07 14:40:01
password:
summary: org.apache.ibatis.binding.BindingException Invalid bound statement (not found)错误解决
tags:
- 原创
- bug
categories:
- java
---
#### 出现原因：
1. xml中的mapper标签的namespace 是否对应
2. @MapperScan("com.itdfq.springboot.db.dao")设置mapper扫描
3. xml文件可能没有放在resource目录下，这样build的时候可能不会build xml文件；需要设置如下：
```xhtml
    <build>
         <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```