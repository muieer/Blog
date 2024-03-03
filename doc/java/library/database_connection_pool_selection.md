# 业务开发中的数据库连接池技术选型思考

## 一 业务背景

工作中有个新项目，我需要搭建一个中低复杂度的基于 Spring Boot 的 Web 后端服务，供部门内部使用。这个项目预计月活用户百人以内，单表不过十万行，热点数据反复查询的吞吐也不会太高。

我在配置数据库连接池的时候，第一时间想到的是阿里的 [Druid](https://github.com/alibaba/druid) 数据库连接池，因为之前校招时读了很多技术文章都说 Druid 数据库连接池功能强大和性能好。

我看了组内之前项目使用的数据库连接池是 [HikariCP](https://github.com/brettwooldridge/HikariCP)。这让我产生了困惑，到底哪个数据库连接池更好。

## 二 数据库连接池的作用

数据库连接开销很大。有网络传输开销，加载驱动开销，认证授权开销，会话建立开销，线程开销。

数据库连接池是一种池化思想，提前创建连接和复用连接，避免每次创建带来的性能开销，来提高程序性能。

## 三 Spring Boot 的选择

> We prefer HikariCP for its performance and concurrency.

[文档](https://docs.spring.io/spring-boot/docs/current/reference/html/data.html#data.sql.datasource.connection-pool)写到会优先使用 HikariCP 数据库连接池。

## 四 阿里 Druid 文档中的性能描述

[Druid Wiki](https://github.com/alibaba/druid/wiki/Druid%E8%BF%9E%E6%8E%A5%E6%B1%A0%E4%BB%8B%E7%BB%8D#12-%E7%AB%9E%E5%93%81%E5%AF%B9%E6%AF%94)中总结：Druid 连接池比竞品功能强大性能好。

Druid 连接池认为自己性能好的原因有三点。一是连接 LRU 方式复用。二是支持 PreparedStatementCache。三是生产环境中严苛的业务场景考验。

## 五 HikariCP 文档中的性能描述

> Using a statement cache at the pooling layer is an anti-pattern, and will negatively impact your application performance compared to driver-provided caches.

HikariCP 认为 JDBC 的常见驱动都有 StatementCache，而且是跨连接共享的，但是数据库连接池做不到跨连接共享，开销大。所以 HikariCP 不使用 PreparedStatementCache，认为这是一种反模式。

## 六 Druid vs HikariCP Benchmarks

两个数据库连接池官方文档都没有包含对方数据的 Benchmark 明细。HikariCP 的 [Issues](https://github.com/brettwooldridge/HikariCP/issues/232) 有彼此的性能对比，单看这次的数据，HikariCP 性能更好。

## 七 我的选择

我最终选择了使用 HikariCP 数据库连接池。原因如下：

1. SpringBoot 优先使用它，社区支持更好。
2. 轻量，配置简单。
3. 文档完善，对细节有更多的描述。 
4. 虽然 Druid 有更好的监控功能，但是我这里有公司级别的数据库监控看板，无需项目自建。