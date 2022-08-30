---
title: Mybatis逆向工程
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-22 17:36:32
password:
summary: Mybatis配置文件
tags:
- 原创
- 逆向工程
categories:
- 数据库
---

* ## 逆向工程所依赖的jar包

```
 <dependencies>
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.4.5</version>
       </dependency>
       <dependency>
           <groupId>org.mybatis.generator</groupId>
           <artifactId>mybatis-generator-core</artifactId>
           <version>1.3.5</version>
       </dependency>
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>5.1.6</version>
       </dependency>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-jdbc</artifactId>
           <version>5.0.2.RELEASE</version>
       </dependency>
   </dependencies>
```
* ## generator.xml配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
<!--生成的jar包都是3点多-->
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
<!--            suppressAllComments属性值：-->
<!--            ture:自动生成实体类，SQL映射文件时没有注释-->
<!--            fauls:自动生成实体类，SQL映射文件有注释-->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!-- 数据库连接信息 驱动类 连接地址 用户名 密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/spring"
                        userId="root"
                        password="123456">
        </jdbcConnection>
<!--    forceBigDecimals属性值：
        ture:把数据表中的DECTMAL和NUMERIC类型解析为java.math.BigDecimal类型
        false:解析为Integer类型
        -->
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!--targetProject属性值:实体类的生成位置
        targetPackage属性值:实体类所在包的路径 -->
        <!-- 生成po类的位置 -->
        <javaModelGenerator targetPackage="cn.itcast.ssm.po" targetProject=".\src\main\java">
<!--            trimStrings属性值:-->
<!--            true:对数据库的查询结果进行trim（去空格）操作-->
<!--            false:不进行trim操作-->
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!--
        targetProject属性值:SQL映射文件的生成位置
        targetPackage属性值:SQL映射文件所在包的路径
        -->
        <!-- mapper映射文件的生成位置 -->
        <sqlMapGenerator targetPackage="cn.itcast.ssm.mapper"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="cn.itcast.ssm.mapper"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 指定数据库表 -->
        <table tableName="account"></table>
        <table tableName="user"></table>
    </context>
</generatorConfiguration>

```

运行测试方法类

```
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.exception.InvalidConfigurationException;
import org.mybatis.generator.exception.XMLParserException;
import org.mybatis.generator.internal.DefaultShellCallback;

import java.io.File;
import java.io.IOException;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;


public class Test {
    public static void main(String[] args) throws IOException, XMLParserException, SQLException, InterruptedException, InvalidConfigurationException {
        File file = new File("src/main/resources/generator.xml");//配置文件
        List<String> warnings = new ArrayList();

        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(file);
        DefaultShellCallback callback = new DefaultShellCallback(true);
        //逆向工程的核心类
        MyBatisGenerator generator = new MyBatisGenerator(config,callback,warnings);
        generator.generate(null);


    }
}

```
