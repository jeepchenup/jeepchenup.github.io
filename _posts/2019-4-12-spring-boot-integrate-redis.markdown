---
layout: post
title: "记录踩坑 - Spring Boot 2.0 整合 Redis 出现序列化"
author: ABei
categories : Java
tags: SpringBoot Redis
---
* content
{:toc}

> Spring Boot Version ：2.1.4.RELEASE




Spring Boot 在整合 Redis 的时候，序列化对象出现乱码情况。在 research 之后，发现网上现有的解决方案都是针对 `Spring Boot 1.x`。看来只能自己看看如何解决。

## 切入点 - RedisCacheConfiguration 类

```java
@Configuration
@ConditionalOnClass(RedisConnectionFactory.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisConnectionFactory.class)
// 注意：在创建的 RedisCacheConfiguration 的前提是，当前 IoC 容器中还不存在 CacheManager 对象
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {

    @Bean
	public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory,
			ResourceLoader resourceLoader) {
		RedisCacheManagerBuilder builder = RedisCacheManager
				.builder(redisConnectionFactory)
                // determineConfiguration 这个方法设置了 Redis 默认的配置
				.cacheDefaults(determineConfiguration(resourceLoader.getClassLoader()));
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		return this.customizerInvoker.customize(builder.build());
	}

    private org.springframework.data.redis.cache.RedisCacheConfiguration determineConfiguration(
			ClassLoader classLoader) {
		if (this.redisCacheConfiguration != null) {
			return this.redisCacheConfiguration;
		}
		Redis redisProperties = this.cacheProperties.getRedis();
		org.springframework.data.redis.cache.RedisCacheConfiguration config = org.springframework.data.redis.cache.RedisCacheConfiguration
				.defaultCacheConfig();
        // 设置 JdkSerializationRedisSerializer 为默认的序列化器
		config = config.serializeValuesWith(SerializationPair
				.fromSerializer(new JdkSerializationRedisSerializer(classLoader)));
		if (redisProperties.getTimeToLive() != null) {
			config = config.entryTtl(redisProperties.getTimeToLive());
		}
		if (redisProperties.getKeyPrefix() != null) {
			config = config.prefixKeysWith(redisProperties.getKeyPrefix());
		}
		if (!redisProperties.isCacheNullValues()) {
			config = config.disableCachingNullValues();
		}
		if (!redisProperties.isUseKeyPrefix()) {
			config = config.disableKeyPrefix();
		}
		return config;
	}

}
```

在 `RedisCacheConfiguration` 类中有 3 个重要的点需要注意：

1.  `@ConditionalOnMissingBean(CacheManager.class)`
    RedisCacheConfiguration 创建的前提是，当前 IoC 容器中还不存在 CacheManager 对象。

1.  在 RedisCacheConfiguration 中创建出了 CacheManager Bean。如果，我们想要自定义 CacheManager 的序列化方式，只能覆盖 RedisCacheConfiguration 默认的 CacheManager。在覆盖之后，因为上面一条的原因，默认的 RedisCacheConfiguration 就不会起作用了。

1.  Spring Data 为 Redis 提供默认的序列化器是 `JdkSerializationRedisSerializer`。
    除了 JdkSerializationRedisSerializer，Spring Data 还为下面几个序列化器：

    ![](http://cdn.51leif.com/2019-4-12-spring-boot-integrate-redis.png)

## 实现自定义 CacheManager

```java
@Configuration
public class CustomRedisConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory, ResourceLoader resourceLoader) {

        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new Jackson2JsonRedisSerializer<Employee>(Employee.class)));

        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
    }
}
```

这里要注意的是 `serializeValuesWith` 方法，它返回的是一个新的 RedisCacheConfiguration 对象。之前没有注意到，我的写法是下面这样的：

```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory, ResourceLoader resourceLoader) {

    RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();

    redisCacheConfiguration.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new Jackson2JsonRedisSerializer<Employee>(Employee.class)));

    return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
}
```

上述写法，一直未生效，在打断点之后，我发现 CacheManager 中依然使用的是 `JdkSerializationRedisSerializer`。在跟进 serializeValuesWith 方法后，发现：

```java
public RedisCacheConfiguration serializeValuesWith(SerializationPair<?> valueSerializationPair) {

    Assert.notNull(valueSerializationPair, "ValueSerializationPair must not be null!");

    // 返回的是一个新的 RedisCacheConfiguration 对象。
    return new RedisCacheConfiguration(ttl, cacheNullValues, usePrefix, keyPrefix, keySerializationPair,
            valueSerializationPair, conversionService);
}
```

自定义的内容由于没有应用进去，所以导致一直是乱码的状态。