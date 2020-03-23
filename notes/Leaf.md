# Leaf

## Description

A Distributed ID Generate Service.

It has three kinds of IG generators, those are:
+ Snowflake based generator
+ Segment based generator
+ Zero generator.

## Snowflake based generator

Follow the idea of snowflake algorithm provided by Twitter.

The main idea:
+ [相对时间差] [worker序号] [毫秒内自增序号] 构成id
+ 使用zk的永久有序节点，来生成和维持worker序号
+ 使用本地文件存储worker序号，在zk不可用的时候，不影响启动
+ 通过定时线程，上报节点的状态
+ 维持内部时间戳，防止服务器时间回退导致的发号重复

## Segment based generator

Meitu propose this algorithm, database based id generator, with two segment buffer.

The main idea:
+ 使用数据库自增类似的id生成方法
+ id号段提前分配给服务，存储在内存中进行缓存和发号
+ 每个实例，维持2个号段，以提高可用性
+ 定时将数据库中的业务新增和删除在实例内部进行同步
+ 如果发号器剩余可用低于阈值，就开始填充后备号段，以应对切换
+ 当前号段耗尽，就切换到后备号段
+ 通过读写锁，来反正大量请求下，切换的一致性
+ 根据拉取号段的时间间隔，动态调整每次拉取的号段大小


## Zero generator

It always return id with value is ZERO. It cannot be use in real application, is just a backup for incorrect use of the Leaf.