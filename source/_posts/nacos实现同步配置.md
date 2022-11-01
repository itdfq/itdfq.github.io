---
title: nacos实现同步配置
top: false
cover: false
toc: true
mathjax: true
date: 2022-11-01 11:54:19
password:
summary: 本文主要讲解使用nacos获取配置，以及监听配置自动更新
tags:
- 原创
- 服务注册发现
categories:
- 微服务
---
# nacos配置

### pom依赖

        <!--nacos-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba.nacos/nacos-client -->
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
            <version>2.0.3</version>
        </dependency>

### nacos配置
1. 首先要先加入一个bootstrap.yml 这个是最先进行加载的
2. 具体的配置如下

  ```yaml
spring:
  application:
    name: nacos # 服务名称
  profiles:
    active: dev #开发环境，这里是dev
  cloud:
    nacos:
      server-addr: 192.168.0.240:8848 # Nacos地址
      config:
        file-extension: yaml # 文件后缀名
```
3. 在nacos的管理界面新增配置
   ![img.png](https://img-blog.csdnimg.cn/78ab1966f20c44e79ce9d72a7df8c130.png)
   注意：命名的方式 应该与bootstrap.yml中的保持一致
   例如：
   ![img.png](https://img-blog.csdnimg.cn/ff8fb497b4514ad698ca14124d91eba0.png)
### 读取nacos配置方式以及自动更新

#### 通过注解
```java
@RefreshScope //第一种热更新方式，加入这个注解，可以在控制台中更新配置
public class ConfigController {

    @Value("${zooksd.namesds}")
    private String test; //这个可以直接读取到nacos中的注解信息

    @Autowired
    private PatternProperties patternProperties;
    @GetMapping("/get")
    public String get(){
        return test;
    }
}
```

#### 通过配置类实现
```java
/**
 * @author duanfangqin 2022/7/7 15:17
 * @implNote
 */
@Component
@Data
@ConfigurationProperties(prefix = "zooksd")
public class PatternProperties {
    public String namesds;
}
```
注意这里的prefix需要与配置信息对应
