---
layout: post
title: "在 Windows 中连接 Ubuntu 内 Redis"
categories: Java
tags: Redis
author: ABei
---

由于在 Ubuntu 中部署 Redis，想要在 Windows 中用 Java 代码连接测试。



在获取 VM 中的 IP 地址之后，写了一个简单的 PingTest。

```java
import redis.clients.jedis.Jedis;

public class ConnectRedis {

	public static void main(String[] args) {
		Jedis jedis = new Jedis("192.168.56.142", 6379);
					
		System.out.println(jedis.ping());
	}
}

```

结果抛出异常，链接失败。

```java
Exception in thread "main" redis.clients.jedis.exceptions.JedisConnectionException: Failed connecting to host 192.168.56.142:6379
    at redis.clients.jedis.Connection.connect(Connection.java:207)
    at redis.clients.jedis.BinaryClient.connect(BinaryClient.java:97)
    at redis.clients.jedis.Connection.sendCommand(Connection.java:126)
    at redis.clients.jedis.BinaryClient.keys(BinaryClient.java:160)
    at redis.clients.jedis.Client.keys(Client.java:102)
    at redis.clients.jedis.Jedis.keys(Jedis.java:283)
    at com.redis.test.redisTest.main(redisTest.java:13)
Caused by: java.net.ConnectException: Connection refused: connect
    at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
    at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:75)
    at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:339)
    at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:200)
    at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:182)
    at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:157)
    at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:391)
    at java.net.Socket.connect(Socket.java:579)
    at redis.clients.jedis.Connection.connect(Connection.java:184)
    ... 6 more
```

在确认虚拟机 IP 地址无误之后，我再在 Dom 窗口 ping 了一次 IP 是无误的。

![](http://cdn.51leif.com/2019-3-3-connect-redis.png)

查阅相关文档之后，发现问题出现在 `redis.conf` 文件中。

在 Network 配置选项中有这么一段文字：

```txt
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

bind 127.0.0.1
```

原来 Redis 默认绑定的 IP 地址是 127.0.0.1。也就是说，除了本机之外的访问都将被阻挡。由于 Redis 是安装在虚拟机中，所以这里在 Windows 端是不能够访问。

所以，这里为了简单方便，我就将 `bind 127.0.0.1` 注释掉，意思就是允许所有的 IP 地址去连接这个 Redis 数据库。

并不是这样做了之后就能连接上了，最后还需要将 `protected-mode` 设置为 `no`。

设置好上面 2 种配置之后，重启 Redis Server 才会生效。