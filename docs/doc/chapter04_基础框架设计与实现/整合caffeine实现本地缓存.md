# SpringBoot 整合caffeine实现本地缓存

| 版本   | 时间       | 修改点 | 修改人  |
| :----- | :--------- | :----- | :------ |
| v1.0.0 | 2022-05-25 | 编写   | LGX_TvT |
| -      | -          | -      | -       |





## 1、概述

本地缓存是指将数据存储在内存中，以避免频繁地从数据库或其他外部资源获取数据。减少应用程序的 I/O 操作，提高应用程序的性能和响应速度，并在需要时快速检索所需的数据。除了提高性能和响应速度之外，本地缓存还可以减少对外部资源的依赖性和网络延迟，使得应用程序更加稳定和可靠，并且更容易扩展和维护。在 Spring Boot 中，可以方便地整合多种本地缓存技术，例如 Ehcache、Guava Cache 和 Caffeine 等。



目前市面上有许多本地缓存框架可供选择，下面是一些常见的集中本地缓存框架的对比：

1. Ehcache：Ehcache 是一个功能强大的开源 Java 缓存框架，可以用于快速和灵活地增加缓存到现有应用程序。它提供了多种缓存策略和配置选项，并支持分布式缓存和缓存预热等高级功能。
2. Guava Cache：Guava Cache 是 Google 开发的一个简单而强大的本地缓存框架，具有高效、可靠和易于使用的特点。它支持多种缓存策略和配置选项，并提供了一组丰富的缓存 API。
3. Caffeine：Caffeine 是一个基于 Java 8 的高性能本地缓存框架，具有快速和灵活的特点。它采用了类似于 Guava Cache 的 API 设计，并提供了一组优化的数据结构和算法，在处理大量数据时能够获得更高的性能。
4. Redisson：Redisson 是一个基于 Redis 的分布式对象和服务框架，可以用于实现分布式缓存、分布式锁、分布式集合等多种分布式应用场景。它采用了高效的序列化和压缩技术，在处理大量数据时能够获得较好的性能和可靠性。
5. Hazelcast：Hazelcast 是一个功能强大的开源分布式缓存框架，可以用于实现分布式缓存、分布式锁、分布式队列等多种分布式应用场景。它采用了高效的内存管理和通信技术，并提供了一组丰富的分布式 API。

以上框架都有其各自的特点和优劣，选择一个合适的本地缓存框架需要根据具体应用场景和需求进行评估和比较。



| 框架名称    | 优点                                                         | 缺点                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Ehcache     | 支持多种缓存策略和配置选项，具有良好的灵活性；<br/>支持分布式缓存和缓存预热等高级功能；<br/>具有较好的可靠性和稳定性。 | 对于大规模高并发的场景可能性能不够优秀；<br/>依赖于 JVM 内存管理机制，无法扩展到多台服务器上。 |
| Guava Cache | 简单易用，具有良好的快速性；<br/>支持多种缓存策略和配置选项；<br/>具有较好的可靠性和稳定性。 | 不支持分布式缓存和缓存预热等高级功能；<br/>依赖于 JVM 内存管理机制，无法扩展到多台服务器上。 |
| Caffeine    | 高性能，适合处理大规模高并发的场景；<br/>支持多种缓存策略和配置选项；<br/>具有较低的内存占用和较好的可靠性。 | 不支持分布式缓存和缓存预热等高级功能；<br/>依赖于 JVM 内存管理机制，无法扩展到多台服务器上。 |
| Redisson    | 支持分布式缓存和缓存预热等高级功能；<br/>具有较好的可靠性和稳定性；<br/>支持多种数据结构和查询操作。 | 依赖于 Redis 服务器，需要额外的配置和运维工作；<br/>性能可能受到网络延迟和 Redis 服务器负载等因素的影响。 |
| Hazelcast   | 支持分布式缓存、锁、队列等多种分布式应用场景；<br/>具有良好的可扩展性和可靠性；<br/>支持多种数据结构和查询操作。 | 依赖于 Hazelcast 集群，需要额外的配置和运维工作；<br/>性能可能受到网络延迟和集群负载等因素的影响。 |



由于只考虑单体项目的场景，且需要尽可能的减少项目的复杂性，不考虑引入第三方缓存数据库等，综合权衡下采用了Caffeine来作为本地缓存的实现方式。Caffeine 是基于 Google Guava Cache 设计经验改进的结果。相较于 Guava Cache， Caffeine在性能和命中率上更具有效率。因其优秀的性能，SpringBoot在2.X 版本中已经放弃了Guava Cache，转而拥抱了Caffeine。



## 2、使用场景

对于以下场景，可以考虑使用Spring Boot 本地缓存缓存数据，减少系统I/O操作，提高相应接口的响应速度：

1. 频繁访问的数据：如果某些数据需要频繁地被访问，可以将其缓存在本地内存中，以减少对数据库或其他外部存储系统的访问次数。
2. 读取操作比写入操作多的情况：如果某些数据的读取操作远远多于写入操作，可以将其缓存在本地内存中，以加速读取操作。
3. 数据量较小的情况：如果某些数据量并不是很大，可以将其缓存在本地内存中，以节省外部存储系统的资源占用。
4. 数据一致性要求不高的情况：如果某些数据的一致性要求不是很高，可以将其缓存在本地内存中，以提高系统的响应速度。
5. 临时数据的处理：如果某些数据只是临时需要处理，并不需要长期保存，可以将其缓存在本地内存中，以方便快速处理。

需要注意的是，Spring Boot 本地缓存并不适用于以下场景：

1. 分布式环境下的缓存：如果需要在分布式环境下使用缓存，建议选择分布式缓存框架，如 Redis、Hazelcast 等。
2. 对数据一致性要求较高的情况：如果数据的一致性要求较高，建议选择分布式缓存框架或数据库等外部存储系统来管理数据。
3. 数据量较大的情况：如果数据量较大，建议选择外部存储系统来管理数据，避免内存溢出等问题。



## 3、整合Caffeine



### 3.1、 基础环境配置

- JDK版本：JDK11
- Caffeine 版本：3.0.4
- SpringBoot 版本：2.5.12



`pom.xml` 文件添加 maven依赖

```xml
<!--spring-boot-cache-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.5.12</version>
</dependency>

<!--caffeine缓存-->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.0.4</version>
</dependency>
```



### 3.2、创建本地缓存配置类



```java
@Configuration
public class CacheConfig extends CachingConfigurerSupport {

    // 初始的缓存空间大小 100条
    private static Integer DEFAULT_CACHE_INITIAL_CAPACITY = 100;
    // 最大缓存数量 1000条
    private static Integer DEFAULT_CACHE_MAX_SIZE = 1000;
    // 缓存过期时间 10s
    private static Integer DEFAULT_CACHE_EXPIRE_AFTER_WRITE = 10;


    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(caffeineCacheBuilder());
        return cacheManager;
    }

    private Caffeine<Object, Object> caffeineCacheBuilder() {
        return Caffeine.newBuilder()
                // 初始的缓存空间大小
                .initialCapacity(DEFAULT_CACHE_INITIAL_CAPACITY)
                // 缓存的最大条数
                .maximumSize(DEFAULT_CACHE_MAX_SIZE)
                // 设置最后一次写入或访问后10s后过期
                .expireAfterWrite(DEFAULT_CACHE_EXPIRE_AFTER_WRITE, TimeUnit.SECONDS);
    }
}
```



这段代码是配置 Spring Cache 模块，使用 Caffeine 作为缓存实现，并设置了缓存的一些基本信息。

该配置类主要实现：

1. 定义了三个默认参数：初始的缓存空间大小、最大缓存数量和缓存过期时间，分别对应 `DEFAULT_CACHE_INITIAL_CAPACITY`、`DEFAULT_CACHE_MAX_SIZE` 和 `DEFAULT_CACHE_EXPIRE_AFTER_WRITE`。
2. 创建了一个 `CacheManager` 实例，使用 Caffeine 作为缓存实现，并将 Caffeine 的构建器注入到 `CaffeineCacheManager` 中。
3. 在 Caffeine 构建器中设置了缓存的一些基本信息，即初始容量、最大容量和过期时间。

这样配置之后，就可以在需要缓存的方法上添加 `@Cacheable` 注解，指定缓存名称，Spring 会自动根据名称从 `CacheManager` 中获取 Caffeine 缓存实例，并按照配置好的参数进行缓存。



注：在使用之前还需要再启动类或者相应配置类上添加`@EnableCaching`开启缓存功能

```java
@SpringBootApplication
@EnableCaching // 启用缓存
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



**下表是Caffeine的一些常用配置：**

| 配置              | 值类型   | 描述说明                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| expireAfterAccess | duration | 最后一次写入或访问后，指定经过多长的时间过期<br>`expireAfterWrite` 和 `expireAfterAccess`同时存在时，以`expireAfterWrite`为准 |
| expireAfterWrite  | duration | 最后一次写入后，指定经过多长的时间缓存过期                   |
| initialCapacity   | integer  | 初始的缓存空间大小                                           |
| maximumSize       | long     | 缓存的最大条数<br>`maximumSize`和`maximumWeight`不可以同时使用 |
| maximumWeight     | long     | 缓存的最大权重                                               |
| refreshAfterWrite | duration | 创建缓存或者最近一次更新缓存后，经过指定的时间间隔后刷新缓存 |



### 3.3、使用方式

使用缓存的方式非常简单，主要有两种使用方式：一种是注解方式,另一种是代码方式。

#### 3.3.1、注解方式的使用

在需要缓存的方法上添加`@Cacheable`注解即可实现该方法返回结果的缓存功能。

例如：

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Cacheable(value = "userCache", key = "#id")
    @Override
    public User getUserById(Integer id) {
        // 从数据库中获取用户信息
        return userDao.getUserById(id);
    }
}
```

上述代码在 `getUserById()` 方法上添加了 `@Cacheable` 注解，指定了缓存名称为 `userCache`，并设置了缓存键为方法参数 `id`。当第一次调用该方法时，Spring 会将返回结果缓存起来；当下一次再次调用该方法时，Spring 会先从缓存中根据缓存键获取结果，如果缓存中有结果，直接返回缓存值，否则调用方法获取结果，并将结果缓存起来。



需要注意的是：在使用缓存时，为了保证缓存的可靠性和正确性，缓存键应该足够唯一、具有代表性，避免出现缓存击穿、缓存雪崩等问题。



> 缓存Key的指定

key 可以使用spEL表达式，例如：

```java
@Cacheable(value = "userCache:getUser", key = "#user.name + ':' + #user.code")
public Map<String, Object> getUser(User user)
```

key 还有一些其他写法，比如获取方法名等

| **属性名称** | **描述**                    | **示例**              |
| ------------ | --------------------------- | --------------------- |
| methodName   | 当前方法名                  | \#root.methodName     |
| method       | 当前方法                    | \#root.method.name    |
| target       | 当前被调用的对象            | \#root.target         |
| targetClass  | 当前被调用的对象的class     | \#root.targetClass    |
| args         | 当前方法参数组成的数组      | \#root.args[0]        |
| caches       | 当前被调用的方法使用的Cache | \#root.caches[0].name |



除了使用 `@Cacheable` 注解之外，Spring Cache 还提供了其他几种注解来实现缓存功能：

1. `@CachePut`：更新缓存中的数据，同时也会执行方法。

2. `@CacheEvict`：从缓存中删除指定的键值对。

3. `@Caching`：组合多个注解，可以在一个方法上同时定义多个缓存操作。

   

下面分别介绍这三种注解的用法：

- @CachePut

`@CachePut` 注解可以用于更新缓存中的数据。当我们需要更新某个对象时，可以调用该对象的更新方法，同时使用 `@CachePut` 将更新后的对象重新加入到缓存中。

例如：

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Cacheable(value = "userCache", key = "#id")
    @Override
    public User getUserById(Integer id) {
        // 从数据库中获取用户信息
        return userDao.getUserById(id);
    }

    @CachePut(value = "userCache", key = "#user.id")
    @Override
    public void updateUser(User user) {
        // 更新数据库中的用户信息
        userDao.updateUser(user);
    }
}
```

这里我们在 `updateUser()` 方法上添加了 `@CachePut` 注解，指定了缓存名称为 `userCache`，并设置了缓存键为方法参数 `user.id`。当调用该方法时，Spring 会先执行方法，然后将方法返回值（即更新后的对象）重新加入到缓存中。



- @CacheEvict

`@CacheEvict` 注解可以用于删除缓存中的数据。当我们需要删除某个对象时，可以调用该对象的删除方法，同时使用 `@CacheEvict` 将该对象从缓存中删除。

例如：

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Cacheable(value = "userCache", key = "#id")
    @Override
    public User getUserById(Integer id) {
        // 从数据库中获取用户信息
        return userDao.getUserById(id);
    }

    @CacheEvict(value = "userCache", key = "#id")
    @Override
    public void deleteUserById(Integer id) {
        // 删除数据库中的用户信息
        userDao.deleteUserById(id);
    }
}
```

这里我们在 `deleteUserById()` 方法上添加了 `@CacheEvict` 注解，指定了缓存名称为 `userCache`，并设置了缓存键为方法参数 `id`。当调用该方法时，Spring 会先执行方法，然后将缓存中对应的键值对删除。



- @Caching

`@Caching` 注解可以组合多个注解，可以在一个方法上同时定义多个缓存操作。

例如：

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Caching(
        cacheable = {
            @Cacheable(value = "userCache", key = "#id")
        },
        put = {
            @CachePut(value = "userCache", key = "#user.id")
        }
    )
    @Override
    public User updateUserById(Integer id, User user) {
        // 更新数据库中的用户信息
        userDao.updateUserById(id, user);
        // 返回更新后的用户信息
        return user;
    }
}
```

这里我们在 `updateUserById()` 方法上添加了 `@Caching` 注解，组合了 `@Cacheable` 和 `@CachePut` 两个注解，并分别指定了不同的缓存名称和缓存键。当调用该方法时，Spring 会先执行方法，然后将方法返回值（即更新后的对象）加入到缓存中，并根据方法参数 `id` 将查询结果也加入到缓存中。



#### 3.3.2、代码方式的使用

如果不想使用注解来实现缓存功能，可以通过代码的方式来使用 Spring Cache。

在需要使用缓存的方法中使用 `CacheManager` 对象来获取缓存并进行操作。

例如：

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Autowired
    private CacheManager cacheManager;

    @Override
    public User getUserById(Integer id) {
        // 从缓存中获取用户信息
        Cache userCache = cacheManager.getCache("userCache"); // 获取缓存对象
        ValueWrapper valueWrapper = userCache.get(id); // 根据键获取值
        if (valueWrapper != null) {
            // 缓存中有结果，直接返回缓存值
            return (User) valueWrapper.get();
        }
        // 缓存中没有结果，从数据库中获取用户信息
        User user = userDao.getUserById(id);
        // 将查询结果加入到缓存中
        userCache.put(id, user);
        return user;
    }
}
```

上述代码在 `getUserById()` 方法中使用了 `CacheManager` 对象来获取名为 `userCache` 的缓存对象，并根据方法参数 `id` 获取缓存中的值。如果缓存中有值，直接返回缓存值；否则从数据库中获取用户信息，并将查询结果加入到缓存中。

需要注意的是，在使用 `CacheManager` 获取缓存对象时，如果缓存名称不存在，CaffeineCacheManager会自动按照缓存名称创建一个缓存对象，源码如下：

```java
	@Override
	@Nullable
	public Cache getCache(String name) {
		return this.cacheMap.computeIfAbsent(name, cacheName ->
				this.dynamic ? createCaffeineCache(cacheName) : null);
	}
```



**CacheManager和Cache的常见用法：**

- CacheManager

`CacheManager` 接口提供了一些常用的缓存操作方法，下面是其中几个常用方法的介绍：

1. `Cache getCache(String name)`：根据指定名称获取对应的缓存对象。如果不存在该名称对应的缓存，则返回 `null`。
2. `Collection<String> getCacheNames()`：获取所有缓存对象的名称集合。



- Cache

`Cache` 接口也提供了一些常用的缓存操作方法，下面是其中几个常用方法的介绍：

1. `ValueWrapper get(Object key)`：根据指定键获取对应的缓存数据。如果不存在该键对应的数据，则返回 `null`。
2. `void put(Object key, Object value)`：向缓存中添加指定键和值的数据。如果该键已经存在，则会覆盖原有的数据。
3. `void evict(Object key)`：删除指定键对应的缓存数据。
4. `void clear()`：清空缓存中所有数据。

在实际使用中，通过`CacheManager`的 `getCache()` 方法获取到指定名称的缓存对象后，可以使用 `get()` 方法获取到指定键的缓存数据，然后进行相关的处理。而使用 `put()` 则可以将新的数据添加到缓存中，而使用 `evict()` 则可以删除指定键的缓存数据。最后，使用 `clear()` 方法则可以清空整个缓存中的数据。



## 4、实际使用



在网站首页初始化时候有些基础配置数据需要通过接口获取（例如：背景图片、自定义元素、网站名称、提示信息等）。由于这些基础配置比较固定，数据量不大，且读取频率远大于写入频率，故而通过引入本地缓存的方式优化基础配置数据接口的响应速度。

![image-20230601100501679](%E6%95%B4%E5%90%88caffeine%E5%AE%9E%E7%8E%B0%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98.assets/image-20230601100501679-16855851053053.png)

配置数据如下（部分）：

```json
{
    "code": 200,
    "success": true,
    "msg": "操作成功",
    "data": [
        {
            "id": "100",
            "name": "网站名称",
            "code": "sys_site_name",
            "value": "🍀 JianYi Admin",
            "type": "base_field",
            "description": "网站名称"
        },
        {
            "id": "101",
            "name": "网站LOGO",
            "code": "sys_logo",
            "value": "",
            "type": "base_field",
            "description": "网站LOGO"
        },
		// ...
    ]
}
```



在基础配置查询接口方法上添加相应的`@Cacheable`缓存注解

```java
    @ApiOperation(value = "列表查询系统基础配置", notes = "")
    @GetMapping("/list")
    @Cacheable(value = "BasicSettingController:doQueryList", key = "#root.methodName")
    public Result<List<BasicSetting>> doQueryList() {
        // ...
        return Result.ok(list);
    }
```



对接口在添加缓存接口与不添加缓存的情况，在相同环境分别进行压测：

压测结果如下：

- 未添加缓存

![QQ截图20230601093445](%E6%95%B4%E5%90%88caffeine%E5%AE%9E%E7%8E%B0%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98.assets/QQ%E6%88%AA%E5%9B%BE20230601093445-16855838044891.jpg)



- 添加缓存

![QQ截图20230601092916](%E6%95%B4%E5%90%88caffeine%E5%AE%9E%E7%8E%B0%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98.assets/QQ%E6%88%AA%E5%9B%BE20230601092916-16855838260732.jpg)

从上述结果可以看出平均响应时间大幅度降低，且吞吐量大幅度提升。
