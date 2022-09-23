---
title: docker部署
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-22 23:37:29
password:
summary: 本文介绍如何使用docker进行项目部署
tags:
- 原创
- docker
categories:
- 项目部署
---
# Docker通过jar包部署并运行
    1. 上传jar到服务器的指定目录

    2. 在该目录下创建Dockerfile 文件

            vim Dockerfile
        注意：
            X86和ARM 的jdk版本不一样

            X86的Dockerfile文件如下：

            FROM java:8
            MAINTAINER bingo
            ADD demo-0.0.1-SNAPSHOT.jar demo.jar
            EXPOSE 8080
            ENTRYPOINT ["java","-jar","demo.jar"]

        ARM的Dockerfile文件如下：

            FROM arm64v8/openjdk:8
            MAINTAINER duan
            ADD springBootDocker.jar demo.jar
            EXPOSE 8888   
            ENTRYPOINT ["java","-jar","demo.jar"]

        说明：

            from java:8   拉取一个jdk为1.8的docker image
            maintainer  作者是duan
            demo-0.0.1-SNAPSHOT.jar 就是你上传的jar包，替换为jar包的名称
            demo.jar  是你将该jar包重新命名为什么名称，在容器中运行
            expose  该容器暴露的端口是多少，就是jar在容器中以多少端口运行
            entrypoint 容器启动之后执行的命令，java -jar demo.jar  即启动jar
    3. 创建好Dockerfile文件之后，执行命令 构建镜像


            docker build -t my/demo .


        注意最后的 .  表示 Dockerfile 文件在当前目录下
        my/demo  构建之后镜像名称

    4. 镜像构建成功之后，就可以运行容器了

            docker run -d --restart=always --name demo -p 8080:8080  my/demo   
        
    5. 一些常用的docker命令

            docker ps :查看正在运行中的容器
            docker ps - a :  查看所有的容器
            docker stop/start 容器名 ：启动停止容器
            docker rm 容器id:删除容器
            docker images: 查看镜像
            docker rmi 镜像id:删除镜像
---

# 通过jdk容器进行部署
####原理：
原理其实是运行一个java镜像，运行时 通过java -jar 命令进行启动。每次更新可以更换对应的jar，重启容器就可以

#### 启动命令：
```shell
docker run -d  --restart=always -v /project/xxl_job:/jar -v /project/xxl_job:/project/xxl_job -p 8080:8080 --name xxl-job java /usr/bin/java -jar -Duser.timezone=GMT+08 /project/xxl_job/xxl-job.jar

```

* -v /project/xxl_job:/project/xxl_job ：宿主机目录：docker容器虚拟目录
* -p 8080:8080 ：对外暴露端口：docker容器真实目录
* --name xxl-job   容器的名字
* java /usr/bin/java -jar java指的是下载的java镜像 后面是启动命令
* -Duser.timezone=GMT+08  指定的时区

---
# 通过maven插件自动化部署
待补充