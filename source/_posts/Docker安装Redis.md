---
title: Docker安装Redis
top: false
cover: false
toc: true
mathjax: true
date: 2022-07-29 11:40:13
summary: 本文介绍如何使用docker安装Redis，并挂载数据板
tags:
- 原创
- docker
categories:
- 软件安装
---
docker安装redis
---
1. docker pull redis 拉取镜像
   拉取docker镜像
2. 获取redis.conf文件
```shell
	# bind 192.168.1.100 10.0.0.1
	# bind 127.0.0.1 ::1
	#bind 127.0.0.1
	
	protected-mode no
	
	port 6379
	
	tcp-backlog 511
	
	requirepass 000415
	
	timeout 0
	
	tcp-keepalive 300
	
	daemonize no
	
	supervised no
	
	pidfile /var/run/redis_6379.pid
	
	loglevel notice
	
	logfile ""
	
	databases 30
	
	always-show-logo yes
	
	save 900 1
	save 300 10
	save 60 10000
	
	stop-writes-on-bgsave-error yes
	
	rdbcompression yes
	
	rdbchecksum yes
	
	dbfilename dump.rdb
	
	dir ./
	
	replica-serve-stale-data yes
	
	replica-read-only yes
	
	repl-diskless-sync no
	
	repl-disable-tcp-nodelay no
	
	replica-priority 100
	
	lazyfree-lazy-eviction no
	lazyfree-lazy-expire no
	lazyfree-lazy-server-del no
	replica-lazy-flush no
	
	appendonly yes
	
	appendfilename "appendonly.aof"
	
	no-appendfsync-on-rewrite no
	
	auto-aof-rewrite-percentage 100
	auto-aof-rewrite-min-size 64mb
	
	aof-load-truncated yes
	
	aof-use-rdb-preamble yes
	
	lua-time-limit 5000
	
	slowlog-max-len 128
	
	notify-keyspace-events ""
	
	hash-max-ziplist-entries 512
	hash-max-ziplist-value 64
	
	list-max-ziplist-size -2
	
	list-compress-depth 0
	
	set-max-intset-entries 512
	
	zset-max-ziplist-entries 128
	zset-max-ziplist-value 64
	
	hll-sparse-max-bytes 3000
	
	stream-node-max-bytes 4096
	stream-node-max-entries 100
	
	activerehashing yes
	
	hz 10
	
	dynamic-hz yes
	
	aof-rewrite-incremental-fsync yes
	
	rdb-save-incremental-fsync yes

```
3. `docker run --restart=always --log-opt max-size=100m --log-opt max-file=2 -p 6379:6379 --name myredis -v /usr/software/redis/redis.conf:/etc/redis/redis.conf -v /usr/software/redis/data:/data -d redis redis-server /etc/redis/redis.conf  --appendonly yes  --requirepass 密码`
   /usr/software/redis/redis.conf是服务器文件地址 映射到docker 容器内部的地址/etc/redis/redis.conf
   同理：挂载的数据盘：/usr/software/redis/data:/data
   --appendonly yes  --requirepass 密码  后台启动 设置密码
4. 启动是否成功可以通过 `docker logs` 容器名 进行查看
5. 进入容器并启动redis `docker exec -it myredis redis-cli`

