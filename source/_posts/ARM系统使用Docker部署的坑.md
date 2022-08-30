---
title: ARM架构使用Docker部署的坑
top: false
cover: false
toc: true
mathjax: true
date: 2020-07-30 17:36:32
password:
summary: 本文主要介绍ARM架构的一些问题。
tags:
- 原创
- 项目部署
categories:
- 项目部署
---

* ## 需要注意 arm的jdk

```
FROM arm64v8/openjdk:8
MAINTAINER duan
ADD springBootDocker.jar demo.jar
EXPOSE 8888   
ENTRYPOINT ["java","-jar","demo.jar"]

```

创建好Dockerfile文件之后，执行命令 构建镜像：

      docker build -t my/demo .

  注意最后的 .  表示 Dockerfile 文件在当前目录下

   my/demo  构建之后镜像名称


   镜像构建成功之后，就可以运行容器了：

       docker run -d --name demo -p 8080:8080 my/demo
