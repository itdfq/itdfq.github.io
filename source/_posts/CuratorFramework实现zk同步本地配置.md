---
title: CuratorFramework实现zk同步本地配置
top: false
cover: false
toc: true
mathjax: true
date: 2021-10-25 17:56:20
password:
summary: zookeeper
tags:
- 原创
- zookeeper
categories:
- java
---
## SpringBoot连接Zk
---


### 连接配置


 - pom文件导入

	```java
	<!--            zk客户端-->
	        <dependency>
	            <groupId>org.apache.curator</groupId>
	            <artifactId>curator-recipes</artifactId>
	            <version>4.3.0</version>
	        </dependency>
	        <dependency>
	            <groupId>org.apache.curator</groupId>
	            <artifactId>curator-framework</artifactId>
	            <version>4.0.1</version>
	        </dependency>
	
	        <dependency>
	            <groupId>org.apache.curator</groupId>
	            <artifactId>curator-client</artifactId>
	            <version>4.0.1</version>
	            <exclusions>
	                <exclusion>
	                    <groupId>log4j</groupId>
	                    <artifactId>log4j</artifactId>
	                </exclusion>
	            </exclusions>
	        </dependency>
	```

 - 连接配置

	```java
	zk:
	  url: ###
	  projectName: /itdfq/zookeeperDemo
	  #休眠时间
	  baseSleepTimeMs: 1000
	  #失败重试次数
	  maxRetries: 3
	```
	

 - 连接设置

	```java
	
	    /**
	     * Zookeeper 框架式客户端
	     *
	     * @return CuratorFramework
	     */
	    @Bean
	    public CuratorFramework getCuratorFramework() {
	        //重试策略
	        RetryPolicy retryPolicy = new ExponentialBackoffRetry(zkProperties.getBaseSleepTimeMs(), zkProperties.getMaxRetries());
	        return CuratorFrameworkFactory.newClient(zkProperties.getUrl(), retryPolicy);
	    }
	
	    /**
	     * 监视 ZK
	     *
	     * @return TreeCache
	     */
	    @Bean
	    public TreeCache getTreeCache() {
	        return new TreeCache(curatorFramework, zkProperties.getProjectName());
	    }
	```

 - 初始化

	```java
	  @Autowired
	    private CuratorFramework curatorFramework;
	    @Autowired
	    private TreeCache treeCache;
	    /**
	     * 本地配置
	     */
	    private final Properties properties = new Properties();
	
	    @PostConstruct
	    public void loadProperties() {
	        try {
	            log.info("================初始化配置=================");
	            curatorFramework.start();
	            treeCache.start();
	            // 从zk中获取配置放入本地配置中
	            Stat stat = curatorFramework.checkExists().forPath(zkProperties.getProjectName());
	            if (stat == null) {
	                curatorFramework.create().forPath(zkProperties.getProjectName());
	            }
	            List<String> configList = curatorFramework.getChildren().forPath(zkProperties.getProjectName());
	            log.info("配置参数：{}", JSON.toJSONString(configList));
	            for (String s : configList) {
	                byte[] value = curatorFramework.getData().forPath(zkProperties.getProjectName() + ZkConstant.SEPARATOR + s);
	                log.info("节点:【{}】   key：【{}】   Value:【{}】 ", zkProperties.getProjectName() + ZkConstant.SEPARATOR + s, s, new String(value));
	                properties.setProperty(s, new String(value));
	            }
	
	            // 监听属性值变更
	            treeCache.getListenable().addListener(new TreeCacheListener() {
	                @Override
	                public void childEvent(CuratorFramework curatorFramework, TreeCacheEvent treeCacheEvent) throws Exception {
	                    if (Objects.equals(treeCacheEvent.getType(), TreeCacheEvent.Type.NODE_ADDED) ||
	                            Objects.equals(treeCacheEvent.getType(), TreeCacheEvent.Type.NODE_UPDATED)) {
	                        String updateKey = treeCacheEvent.getData().getPath().replace(zkProperties.getProjectName() + ZkConstant.SEPARATOR, "");
	                        properties.setProperty(updateKey, new String(treeCacheEvent.getData().getData()));
	                        log.info("数据更新：更新类型【{}】，更新的key:【{}】,更新value:【{}】", treeCacheEvent.getType(), zkProperties.getProjectName() + ZkConstant.SEPARATOR + updateKey, new String(treeCacheEvent.getData().getData()));
	                    }
	                }
	            });
	
	        } catch (Exception e) {
	            log.error("zk配置初始化异常", e);
	        }
	    }
	```
### get/set方法

```java

    /**
     * 获取同步配置属性
     * </p>
     * zk的value保存在本地缓存
     *
     * @param key zk-Key
     * @return String
     */
    public String getProperties(String key) {
        return properties.getProperty(key);
    }

    /**
     * 获取zk的节点value
     * </>
     *
     * @return String
     */
    public String getZkValue(String key) {
        try {
            byte[] bytes = curatorFramework.getData().forPath(zkProperties.getProjectName() + ZkConstant.SEPARATOR + key);
            return new String(bytes);
        } catch (Exception e) {
            return null;
        }
    }

    /**
     * 设置zk的key value
     * <p>
     * 如果项目启动，会同步更新本地配置缓存
     *
     * @param key
     * @param value
     * @return
     */
    public Boolean setZkValue(String key, String value) {
        try {
            String propertiesKey = zkProperties.getProjectName() + ZkConstant.SEPARATOR + key;
            Stat stat = curatorFramework.checkExists().forPath(propertiesKey);
            if (stat == null) {
                curatorFramework.create().forPath(propertiesKey);
            }
            curatorFramework.setData().forPath(propertiesKey, value.getBytes());
            return Boolean.TRUE;
        } catch (Exception e) {
            return Boolean.FALSE;
        }
    }
```
### 分布式锁

```java
   /**
     * 分布式锁 对象
     * <p/>
     * 互斥锁
     *
     * @param path
     * @return
     */
    public InterProcessMutex getLock(String path) {
        InterProcessMutex lock = null;
        try {
            lock = new InterProcessMutex(curatorFramework, zkProperties.getProjectName() + ZkConstant.SEPARATOR + path);
            return lock;
        } catch (Exception e) {
            log.error("获取锁失败", e);
        }
        return null;
    }

```
### 实现结果

 - 项目启动初始化
![项目初始化](https://img-blog.csdnimg.cn/da1e996ac4964b30bc34e7541b5862cd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASVRkZnE=,size_20,color_FFFFFF,t_70,g_se,x_16)
 - 在zk中改个配置
![更改配置](https://img-blog.csdnimg.cn/16823ba4068041778b0ec54f3651c1e0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASVRkZnE=,size_20,color_FFFFFF,t_70,g_se,x_16)
 - 结果
 ![结果](https://img-blog.csdnimg.cn/806cb8755a2645f3af5a40676b0e1bd0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASVRkZnE=,size_20,color_FFFFFF,t_70,g_se,x_16)
 - 在线获取配置
 

```java
  @RequestMapping("/get/{key}")
    public String get(@PathVariable("key") String key) {
        log.info("请求参数：{}", key);
        return "结果：" + zkApi.getProperties(key);
    }
```

![获取配置](https://img-blog.csdnimg.cn/660739ce23b34291963b4a3ff8c37566.png)
