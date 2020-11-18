---
layout: post
title:  "Go 性能优化的方向"
date:   2020-11-16 17:10:00

categories: go
tags: performance
author: "Victor"
---

## 基本概念

### QPS

每秒查询率 QPS 是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。其用来衡量服务的性能，对应fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。

* 原理：每天80%的访问集中在20%的时间里，这20%时间叫做峰值时间。
* 公式：(总PV数 * 80%) / (每天秒数 * 20%) = 峰值时间每秒请求数(QPS) 。
* 机器：峰值时间每秒QPS / 单台机器的QPS = 需要的机器

问：每天300w PV 的在单台机器上，这台机器需要多少QPS？<br />
答：(3000000 * 0.8) / (86400 * 0.2) = 139 (QPS)。 一般需要达到139QPS，因为是峰值。

### 响应时间(RT)

响应时间是指系统对请求作出响应的时间。直观上看，这个指标与人对软件性能的主观感受是非常一致的，因为它完整地记录了整个计算机系统处理请求的时间。

由于一个系统通常会提供许多功能，而不同功能的处理逻辑也千差万别，因而不同功能的响应时间也不尽相同，甚至同一功能在不同输入数据的情况下响应时间也不相同。所以，在讨论一个系统的响应时间时，人们通常是指该系统所有功能的平均时间或者所有功能的最大响应时间。当然，往往也需要对每个或每组功能讨论其平均响应时间和最大响应时间。

对于单机的没有并发操作的应用系统而言，人们普遍认为响应时间是一个合理且准确的性能指标。

### 吞吐量(Throughput)

吞吐量是指系统在单位时间内处理请求的数量。对于无并发的应用系统而言，吞吐量与响应时间成严格的反比关系，实际上此时吞吐量就是响应时间的倒数。前面已经说过，对于单用户的系统，响应时间（或者系统响应时间和应用延迟时间）可以很好地度量系统的性能，但对于并发系统，通常需要用吞吐量作为性能指标。

对于一个多用户的系统，如果只有一个用户使用时系统的平均响应时间是t，当有你n个用户使用时，每个用户看到的响应时间通常并不是n×t，而往往比n×t小很多（当然，在某些特殊情况下也可能比n×t大，甚至大很多）。这是因为处理每个请求需要用到很多资源，由于每个请求的处理过程中有许多步骤难以并发执行，这导致在具体的一个时间点，所占资源往往并不多。也就是说在处理单个请求时，在每个时间点都可能有许多资源被闲置，当处理多个请求时，如果资源配置合理，**每个用户看到的平均响应时间并不随用户数的增加而线性增加。**

实际上，不同系统的平均响应时间随用户数增加而增长的速度也不大相同，这也是采用吞吐量来度量并发系统的性能的主要原因。一般而言，吞吐量是一个比较通用的指标，两个具有不同用户数和用户使用模式的系统，如果其最大吞吐量基本一致，则可以判断两个系统的处理能力基本一致。

### SLB 的活跃连接数和非活跃连接数

* 并发连接数：所有建立的 TCP 连接数量。
* 活跃连接数：当时所有 ESTABLISHED 状态的 TCP 连接，如果您采用的是长连接的情况，一个连接会同时传输多个文件请求。
* 非活跃连接数：指除 ESTABLISHED 状态的其他所有状态 TCP 连接数

如果服务端支持 Keep-Alive，当串联链路中放入了思考时间且时间比较大（大于 Keep-Alive 的 timeout），那么会出现 SLB 上的并发连接数特别是活跃连接数下降的情况。

## 高并发的四个角度

1. 首先是无状态前端机器不足以承载请求流量，需要进行水平扩展，一般QPS是千级；
2. 然后是关系型数据库无法承载读取或写入峰值，需要数据库横向扩展或引入nosql，一般是千到万级；
3. 之后是单机nosql无法承载，需要nosql横向扩展，一般是十万到百万QPS；
4. 最后是难以单纯横向扩展nosql，比如微博就引入多级缓存架构，这种架构一般可以应对百万到千万对nosql的访问QPS。

当然面向用户的接口请求一般到不了这个量级，QPS递增大多是由于读放大造成的压力，单也属于高并发架构考虑的范畴。

具体多少QPS跟业务强相关，只读接口读缓存，将压力给到缓存单机3000+没问题，写请求1000+也正常，也复杂些可能也就几百+QPS。所以QPS和业务场景和设计相关性很大，比如可以通过浏览器本地缓存，用缓存做热点数据查询，写事务MQ异步处理等方式提升QPS。

## 怎么优化 golang 性能

### cpu 耗时优化

* make时提前预估size
* 临时的map、slice采用sync.Pool
* 大于32Kb也可用sync.Pool
* 不滥用goroutine，减少gc压力
* 不滥用mutex，减少上下文切换
* []byte与string临时变量转换用unsafe
* 减少reflect、defer使用
* atomic无锁使用

### 网络io性能优化

* 批量接口支持
* http 长连接
* Redis pipeline
* db、redis连接池
* 增加缓存
* 大量数据压缩传输

## 相关阅读

* [吞吐量（TPS）、QPS、并发数、响应时间（RT）概念](https://www.cnblogs.com/longxiaojiangi/p/9259745.html)
* [PTS使用指引](https://help.aliyun.com/document_detail/145501.html?spm=a2c4g.11186623.6.541.4dabf9ecvYdB5j)
* [一直再说高并发，多少QPS才算高并发？](https://my.oschina.net/u/1000241/blog/3065185)
* [性能调测-压力并发-模拟生产环境数据](https://www.cnblogs.com/Yanqiqi/p/12690505.html)
* [怎么优化golang性能](https://m.yisu.com/zixun/126320.html)