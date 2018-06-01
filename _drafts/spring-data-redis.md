### 安装Redis
```sh
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
tar xzf redis-4.0.9.tar.gz
cd redis-4.0.9/
# sudo apt-get install build-essential tcl
sudo make
sudo make test
sudo make install
# sudo ./utils/install_server.sh
sudo vim /etc/redis/6379.conf

# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# bind 127.0.0.1

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
# protected-mode no

# 密码设置
# requirepass foobared
```
### Spring Data Redis

v1.8 Spring Framework 4.0以上
```java

@Bean
public CacheManager cacheManager() {
    return new RedisCacheManager(redisTemplate());
}

@Bean
public JedisConnectionFactory jedisConnectionFactory() {
    JedisConnectionFactory connectionFactory = new JedisConnectionFactory();
    connectionFactory.setHostName(${redisHostName});
    connectionFactory.setPassword(${redisPassWd});
    connectionFactory.setUsePool(true);
    connectionFactory.setPort(${redisPort});
    connectionFactory.setPoolConfig(jedisPoolConfig());

    return connectionFactory;
}

@Bean
public JedisPoolConfig jedisPoolConfig() {
    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
    jedisPoolConfig.setMinEvictableIdleTimeMillis(43200000);
    jedisPoolConfig.setSoftMinEvictableIdleTimeMillis(43200000);

    return jedisPoolConfig;
}

@Bean
RedisTemplate<Object, Object> redisTemplate() {
    RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(jedisConnectionFactory());
    /// use Jackson to serialize the bean
    /// redisTemplate.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
    return redisTemplate;
}
```
v2, Spring Framework 5.0 以上
```java
@Bean
public CacheManager cacheManager() {
    return RedisCacheManager.create(jedisConnectionFactory());
}

@Bean
public JedisConnectionFactory jedisConnectionFactory() {
    JedisConnectionFactory connectionFactory = new JedisConnectionFactory();
    connectionFactory.getStandaloneConfiguration().setHostName("");
    connectionFactory.getStandaloneConfiguration().setPassword(RedisPassword.of(""));

    return connectionFactory;
}
```
