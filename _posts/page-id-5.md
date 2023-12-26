---
title: "掌握Redis集群：使用Docker搭建Redis高可用Cluster集群环境"
date: 2023-06-14 09:49:44
img: https://wangxs020202.gitee.io/pbad/background/page_5.png
top: false
summary: 本文将介绍Redis的三种集群模式，以及使用Docker搭建Redis高可用Cluster集群环境。
categories: Redis
tags:
  - Redis
  - Docker
  - 集群
  - 高可用
---

### 序幕

在生产环境中，为了保证Redis的高可用性，我们通常会使用Redis集群来搭建Redis高可用环境。

### Redis集群模式

Redis集群有三种模式，分别是主从模式、哨兵模式和Cluster模式。

一般情况下都会以主从的模式进行搭建，我们知道主从的原理是从服务器获取rdb文件的全量复制+写操作的增量复制来共同保证数据的一致性，因此我们在配置从服务器时，需要指定主服务器的ip和端口，这样从服务器才能够连接到主服务器进行数据的复制。
下图是一个简单的主从模式的Redis集群示意图：

![Redis_2023-06-14_10-04-52](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-04-52.png)

但是主从模式也有一些缺点，比如主服务器宕机后，从服务器无法自动切换为主服务器，需要手动进行切换，这样就会造成一定的停机时间，因此我们需要哨兵模式来解决这个问题。

哨兵模式，就是用于在一主多从的集群环境下，如果主服务器宕机了，它会自动的将从服务器中的一台设为新的master，并且将其余的slave的配置文件自动修改，这样就切换出一套新的主从服务，不需要人工干预，且不会影响服务的使用。

这里就不再赘述哨兵模式的原理了，后期会单独写一篇文章来介绍哨兵模式的原理和搭建。

**Cluster模式**，是Redis3.0之后推出的一种集群模式，它的特点是：**数据分片**，**高可用**，**自动分配**，**自动迁移**，**自动复制
**，**自动扩容**，**自动容错**，也是目前Redis官方推荐的集群模式。

这里引用下[官方](https://redis.io/docs/reference/cluster-spec/)的解释：

```text
Redis Cluster 实现了非分布式 Redis 版本中可用的所有单键命令。执行复杂的多键操作（如集合并集和交集）的命令是针对操作中涉及的所有键散列到同一个槽的情况实现的。

Redis Cluster 实现了一个称为散列标签的概念，可用于强制将某些键存储在同一个散列槽中。但是，在手动重分片过程中，多键操作可能会在一段时间内不可用，而单键操作始终可用。

Redis Cluster 不像 Redis 的独立版本那样支持多数据库。我们只支持数据库0；该SELECT命令是不允许的。这是因为在 Redis Cluster 中，数据库是通过槽而不是通过数据库号码来选择的。
```

### Redis Cluster集群搭建

一般在生产环境中，会使用不同的物理机来搭建Redis集群，但是在开发环境中，我们可以使用Docker来搭建Redis集群，这样可以节省一些成本。

#### 1. 环境准备

系统环境：CentOS Stream release 9
Docker版本：Docker version 24.0.1, build 6802122

#### 2. 拉取Redis镜像

```shell
docker pull redis
```

![Redis_2023-06-14_10-26-57](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-26-57.png)

#### 3. 创建挂载目录

```shell
mkdir -p /mnt/redis/{6381,6382,6383,6384,6385,6386}/data
```

#### 4. 创建节点容器

```shell
docker create --name redis-6381 --restart=always --net host  -v /mnt/redis/6381/data:/data redis --requirepass 123456 --masterauth 123456 --port 6381 --appendonly yes --cluster-enabled yes --cluster-announce-ip 127.0.0.1
```

... ...

```shell
docker create --name redis-6386 --restart=always --net host  -v /mnt/redis/6386/data:/data redis --requirepass 123456 --masterauth 123456 --port 6386 --appendonly yes --cluster-enabled yes --cluster-announce-ip 127.0.0.1
```

![Redis_2023-06-14_10-32-29](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-32-29.png)

- requirepass：外面服务、客户端来连接 redis 的密码
- masterauth：redis从去连接 redis 主使用的密码。这个意思是说，如果你在主上设置了requirepass 参数，你就需要再从上设置 masterauth 参数，并和主密码指定成一样的。这样从才能继续去同步主的数据
- appendonly：是否开启持久化
- cluster-announce-ip：对外提供访问的IP，如果集群机器不在局域网内的服务器需填写外网域名或IP

#### 5. 启动节点容器

```shell
docker start redis-6381 redis-6382 redis-6383 redis-6384 redis-6385 redis-6386
```

![Redis_2023-06-14_10-37-37](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-37-37.png)


#### 6. 创建集群

```shell
docker exec -it redis-6381 redis-cli -a 123456 --cluster create 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1
```

![Redis_2023-06-14_10-41-29](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-41-29.png)
![Redis_2023-06-14_10-42-24](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-42-24.png)

- --cluster-replicas 1：表示每个主节点对应一个从节点
- --cluster-replicas 0：表示每个主节点没有从节点

至此，Redis集群搭建完成。接下来我们就可以使用Redis集群了。

### Redis Cluster集群使用

#### 1. 连接集群

```shell
docker exec -it redis-6381 redis-cli -h 127.0.0.1 -p 6382 -a 123456 -c
```

![Redis_2023-06-14_10-45-47](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-45-47.png)

#### 2. 查看集群信息

```shell
cluster info
```

![Redis_2023-06-14_10-47-16](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-47-16.png)

#### 3. 查看集群节点信息

```shell
cluster nodes
```

![Redis_2023-06-14_10-48-06](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-48-06.png)

#### 4. 查看集群槽信息

```shell
cluster slots
```

![Redis_2023-06-14_10-49-04](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-49-04.png)

#### 5. 创建键值对

```shell
set name wangxs
```

#### 6. 查看键值对

```shell
get name
```

![Redis_2023-06-14_10-51-47](https://wangxs020202.gitee.io/pbad/new/redis/Redis_2023-06-14_10-51-47.png)


### 结语

对于主从模式，我们可以使用Redis Sentinel来实现高可用，但是对于Redis Cluster，我们可以使用Redis Cluster自身来实现高可用，这样就不需要再使用Redis Sentinel了。

希望本篇文章对你有所帮助，如果有什么问题，欢迎联系我。


