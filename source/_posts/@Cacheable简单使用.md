---
title: '@Cacheable简单使用'
top: false
cover: false
toc: true
mathjax: true
date: 2022-09-06 16:24:39
summary: 本文主要介绍@Cacheable的简单使用,介绍下SpringBoot环境下如何使用缓存提高效率
tags:
- 原创
- 缓存
categories:
- java
---
### @Cacheable学习笔记

---
#### 注解
1. @CachePut：既调用方法，又更新缓存数据；同步更新缓存
2. @Cacheable：查找缓存 - 有就返回 -没有就执行方法体 - 将结果缓存起来；
    * cacheNames/value ：用来指定缓存组件的名字<br/>
    * key ：缓存数据时使用的 key，可以用它来指定。默认是使用方法参数的值。（这个 key 你可以使用 spEL 表达式来编写）<br/>
    * keyGenerator ：key 的生成器。 key 和 keyGenerator 二选一使用<br/>
    * cacheManager ：可以用来指定缓存管理器。从哪个缓存管理器里面获取缓存。<br/>
    * condition ：可以用来指定符合条件的情况下才缓存<br/>
    * unless ：否定缓存。当 unless 指定的条件为 true ，方法的返回值就不会被缓存。当然你也可以获取到结果进行判断。（通过 #result 获取方法结果）<br/>
    * sync ：是否使用异步模式。
3. @CacheEvict 就是一个触发器，在每次调用被它注解的方法时，就会触发删除它指定的缓存的动作。
    * allEntries 是 @CacheEvict 特有的一个属性，意为是否删除整个缓存（value 或 cacheNames 指定的），默认为 false
    * beforeInvocation 是 @CacheEvict 中特有的一个属性，意为是否在执行对应方法之前删除缓存，默认 false（即执行方法之后再删除缓存）。
#### 注意事项
1. @CachePut和@Cacheable 同时使用的时候，返回值必须一样，否则@CachePut更细会异常
#### 自定义的cacheManager

```java
package com.itdfq.springboot.redis.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

/**
 * @author duanfangqin 2022/7/11 10:29
 * @implNote
 */
@EnableCaching  //开启缓存
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
   @Bean
   public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
      RedisTemplate<String, Object> template = new RedisTemplate<>();
      RedisSerializer<String> redisSerializer = new StringRedisSerializer();
      Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
      ObjectMapper om = new ObjectMapper();
      om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
      jackson2JsonRedisSerializer.setObjectMapper(om);
      template.setConnectionFactory(factory);
      //key序列化方式
      template.setKeySerializer(redisSerializer);
      //value序列化
      template.setValueSerializer(jackson2JsonRedisSerializer);
      //value hashmap序列化
      template.setHashValueSerializer(jackson2JsonRedisSerializer);
      return template;
   }

   @Bean
   public CacheManager cacheManager(RedisConnectionFactory factory) {
      RedisSerializer<String> redisSerializer = new StringRedisSerializer();
      Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
      //解决查询缓存转换异常的问题
      ObjectMapper om = new ObjectMapper();
      om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
      jackson2JsonRedisSerializer.setObjectMapper(om);
      // 配置序列化（解决乱码的问题）,过期时间600秒
      RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
              .entryTtl(Duration.ofSeconds(600))
              .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
              .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
              .disableCachingNullValues();
      RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
              .cacheDefaults(config)
              .build();
      return cacheManager;
   }

   @Bean("cacheManagerDemo")
   @Primary //优先使用这个
   public CacheManager cacheManager1(RedisConnectionFactory factory) {
      // 生成一个默认配置，通过config对象即可对缓存进行自定义配置
      RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
      // 设置缓存的默认过期时间，也是使用Duration设置
      config = config.entryTtl(Duration.ofMinutes(2))
              // 不缓存空值
              .disableCachingNullValues();
      // 设置一个初始化的缓存空间set集合
      Set<String> cacheNames = new HashSet<>();
      cacheNames.add("catalog_test_id");
      cacheNames.add("catalog_test_name");

      // 对每个缓存空间应用不同的配置
      Map<String, RedisCacheConfiguration> configMap = new HashMap<>();
      configMap.put("catalog_test_id", config);
      configMap.put("catalog_test_name", config.entryTtl(Duration.ofMinutes(5)));
      // 使用自定义的缓存配置初始化一个cacheManager
      RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
              // 注意这两句的调用顺序，一定要先调用该方法设置初始化的缓存名，再初始化相关的配置
              .initialCacheNames(cacheNames)
              .withInitialCacheConfigurations(configMap)
              .build();
      return cacheManager;
   }

}

```



