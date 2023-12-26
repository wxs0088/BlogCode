---
title: Docker搭建ClickHouse+Zookeeper集群
date: 2023-04-14 13:58:45
img: https://wangxs020202.gitee.io/pbad/background/ClickHouse-Zookeeper.png
top: false
summary: Docker搭建ClickHouse+Zookeeper集群，是一种高效、灵活和可靠的解决方案，能够满足大规模数据处理和分析的需求。ClickHouse是一款开源的面向列存储的分布式数据库，可以快速地处理大范围的实时数据。而Zookeeper则是一个分布式的协调框架，为ClickHouse提供了多个节点之间协作运作的支持。
categories: Docker
tags:
  - db
  - Docker
  - ClickHouse
  - Ubuntu
---

### 一、前言
Docker搭建ClickHouse+Zookeeper集群，是一种高效、灵活和可靠的解决方案，能够满足大规模数据处理和分析的需求。ClickHouse是一款开源的面向列存储的分布式数据库，可以快速地处理大范围的实时数据。而Zookeeper则是一个分布式的协调框架，为ClickHouse提供了多个节点之间协作运作的支持。

该集群的优势在于，Docker将应用程序、库和依赖项打包成标准镜像并提供容器隔离，简化了 ClickHouse 和 Zookeeper 的配置和部署流程，同时带来更灵活的资源分配和使用。并且，在集群中添加或移除节点都可以实现弹性扩缩容，在保证服务的质量和稳定性的同时，节约了企业的资源成本。

在这篇博客中，我们将会介绍如何借助Docker技术，使用Docker-Compose将ClickHouse+Zookeeper集群快速搭建至本地环境，并对集群的扩展、管理以及数据备份等方面进行深入的探讨。无论是初学者还是资深的技术爱好者，都能从中获得有价值的经验和知识，进一步掌握 Docker和分布式数据库基础知识，提高工作效率和技术实力。

### 二、搭建
1. 需要创建一台`Ubuntu`系统的主机/虚拟机

2. 更换apt源

   ```shell
   # 备份信息
   sudo cp /etc/apt/sources.list /etc/apt/sources.list.bat
   # 打开配置文件进行修改
   sudo vi /etc/apt/sources.list
   # 删除所有数据换成一下信息
   ```

   ```reStructuredText
   deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
   deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
   # deb-src [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
   ```

   ```shell
   # 更新apt
   sudo apt-get update
   ```

3. 搭建docker环境

   ```shell
   # 卸载旧的docker环境
   sudo apt-get remove docker docker-engine docker-ce docker.io
   
   # 安装以下包以使apt可以通过HTTPS使用存储库（repository）：
   sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
   
   # 添加Docker官方的GPG密钥：
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   
   # 使用下面的命令来设置stable存储库：
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   
   # 安装最新版本的Docker CE：这个根据网络情况会比较慢
   sudo apt-get install -y docker-ce
   
   # 查看docker服务状态
   systemctl status docker
   
   # 如果没启动，则启动docker服务
   
   sudo systemctl start docker
   
   # 修改docker源
   
   # 进入阿里云 复制 加速器地址
   https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
   
   # 修改/创建daemon配置文件/etc/docker/daemon.json来使用加速器
   sudo vim /etc/docker/daemon.json
   
   # 复制一下信息
   {
     "registry-mirrors": ["https://xxxxxxxx.mirror.aliyuncs.com"]
   }
   ```

4. 拉取`yandex/clickhouse-server`、`zookeeper`镜像

   ```shell
   docker pull yandex/clickhouse-server
   docker pull zookeeper
   ```

5. 复制虚拟机

   使用VMware直接复制

6. 修改hosts

   ```shell
   # 三台服务器的ip分别是：117、105、103
   # 分别修改三台服务器的hosts文件
   vim /etc/hosts
   
   # 服务1 的ip
   192.168.3.117 server01
   # 服务2 的ip
   192.168.3.105 server02
   # 服务3 的ip
   192.168.3.103 server03
   ```

7. Zookeeper集群搭建

   ```shell
   # 创建zk配置信息存放文件
   sudo mkdir /usr/soft
   sudo mkdir /usr/soft/zookeeper
   ```

   **server01执行：**

   ```shell
   docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper_node --restart always \
   -v /usr/soft/zookeeper/data:/data \
   -v /usr/soft/zookeeper/datalog:/datalog \
   -v /usr/soft/zookeeper/logs:/logs \
   -v /usr/soft/zookeeper/conf:/conf \
   --network host  \
   -e ZOO_MY_ID=1  zookeeper
   ```

   **server02执行：**

   ```shell
   docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper_node --restart always \
   -v /usr/soft/zookeeper/data:/data \
   -v /usr/soft/zookeeper/datalog:/datalog \
   -v /usr/soft/zookeeper/logs:/logs \
   -v /usr/soft/zookeeper/conf:/conf \
   --network host  \
   -e ZOO_MY_ID=2  zookeeper
   ```

   **server03执行：**

   ```shell
   docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper_node --restart always \
   -v /usr/soft/zookeeper/data:/data \
   -v /usr/soft/zookeeper/datalog:/datalog \
   -v /usr/soft/zookeeper/logs:/logs \
   -v /usr/soft/zookeeper/conf:/conf \
   --network host  \
   -e ZOO_MY_ID=3  zookeeper
   ```

   唯一的差别是： `-e ZOO_MY_ID=*` 而已。

8. 修改zookeeper配置文件

   `/usr/soft/zookeeper/conf/zoo.cfg`

   ```shell
   dataDir=/data
   dataLogDir=/datalog
   tickTime=2000
   initLimit=5
   syncLimit=2
   clientPort=2181
   autopurge.snapRetainCount=3
   autopurge.purgeInterval=0
   maxClientCnxns=60
   
   # 服务1 的ip
   server.1=192.168.3.117:2888:3888
   # 服务2 的ip
   server.2=192.168.3.105:2888:3888
   # 服务3 的ip
   server.3=192.168.3.103:2888:3888
   ```

9. 验证zookeeper是否配置成功

   ```shell
   docker exec -it zookeeper_node /bin/bash
   ./bin/zkServer.sh status
   ```

   **成功结果**

   ```shell
   ZooKeeper JMX enabled by default
   Using config: /conf/zoo.cfg
   Client port found: 2181. Client address: localhost. Client SSL: false.
   Mode: follower
   ```

10. Clickhouse集群部署

    - 拷贝出临时镜像配置

      ```shell
      # 运行一个临时容器，目的是为了将配置、数据、日志等信息存储到宿主机上：
      docker run --rm -d --name=temp-ch yandex/clickhouse-server
      # 拷贝容器内的文件：
      docker cp temp-ch:/etc/clickhouse-server/ /etc/
      ```

    - 修改配置文件`/etc/clickhouse-server/config.xml`

      ```xml
      <listen_host>0.0.0.0</listen_host>
      <timezone>Asia/Shanghai</timezone>
      <remote_servers incl="clickhouse_remote_servers" />
      <include_from>/etc/clickhouse-server/metrika.xml</include_from>
      <zookeeper incl="zookeeper-servers" optional="true" />
      <macros incl="macros" optional="true" />
      ```

    - 拷贝配置文件到挂载文件下

      ```shell
      # 分别在server01 server02 server03 执行创建指令
      # 创建挂载文件 mian
      sudo mkdir /usr/soft/clickhouse-server
      sudo mkdir /usr/soft/clickhouse-server/main
      sudo mkdir /usr/soft/clickhouse-server/main/conf
      # 创建挂载文件 sub
      sudo mkdir /usr/soft/clickhouse-server/sub
      sudo mkdir /usr/soft/clickhouse-server/sub/conf
      # 拷贝配置文件
      cp -rf /etc/clickhouse-server/ /usr/soft/clickhouse-server/main/conf
      cp -rf /etc/clickhouse-server/ /usr/soft/clickhouse-server/sub/conf
      ```

    - 修改每台服务器的scp配置

      ```shell
      vim /etc/ssh/sshd_config
      # 修改
      PermitRootLogin yes
      # 重启服务
      systemctl restart sshd 
      ```



    - 分发到其他服务器

      ```shell
      # 拷贝配置到server02上
      scp -r /usr/soft/clickhouse-server/main/conf/ server02:/usr/soft/clickhouse-server/main/
      scp -r /usr/soft/clickhouse-server/sub/conf/ server02:/usr/soft/clickhouse-server/sub/
      # 拷贝配置到server03上
      scp -r /usr/soft/clickhouse-server/main/conf/ server03:/usr/soft/clickhouse-server/main/
      scp -r /usr/soft/clickhouse-server/sub/conf/ server03:/usr/soft/clickhouse-server/sub/
      ```

    - 删除掉临时容器

      ```shell
      docker rm -f temp-ch
      ```

    - 进入`server01`修改`/usr/soft/clickhouse-server/sub/conf/config.xml`为了和主分片 **main**的配置区分开来

      ```XML
      原：
      <http_port>8123</http_port>
      <tcp_port>9000</tcp_port>
      <mysql_port>9004</mysql_port>
      <postgresql_port>9005</postgresql_port>
      <interserver_http_port>9009</interserver_http_port>
      
      修改为：
      <http_port>8124</http_port>
      <tcp_port>9001</tcp_port>
      <mysql_port>9005</mysql_port>
      <!--<postgresql_port>9005</postgresql_port>-->
      <interserver_http_port>9010</interserver_http_port>
      
      ```

      **server02**和**server03**如此修改或scp命令进行分发

    - `server01`新增集群配置文件`/usr/soft/clickhouse-server/main/conf/metrika.xml`

      ```xml
      <yandex>
          <!-- CH集群配置,所有服务器都一样 -->
          <clickhouse_remote_servers>
              <cluster_3s_1r>
                  <!-- 数据分片1  -->
                  <shard>
                      <internal_replication>true</internal_replication>
                      <replica>
                          <host>server01</host>
                          <port>9000</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                      <replica>
                          <host>server03</host>
                          <port>9001</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                  </shard>
                  <!-- 数据分片2  -->
                  <shard>
                      <internal_replication>true</internal_replication>
                      <replica>
                          <host>server02</host>
                          <port>9000</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                      <replica>
                          <host>server01</host>
                          <port>9001</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                  </shard>
                  <!-- 数据分片3  -->
                  <shard>
                      <internal_replication>true</internal_replication>
                      <replica>
                          <host>server03</host>
                          <port>9000</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                      <replica>
                          <host>server02</host>
                          <port>9001</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                  </shard>
              </cluster_3s_1r>
          </clickhouse_remote_servers>
      
          <!-- zookeeper_servers所有实例配置都一样 -->
          <zookeeper-servers>
              <node index="1">
                  <host>192.168.3.117</host>
                  <port>2181</port>
              </node>
              <node index="2">
                  <host>192.168.3.105</host>
                  <port>2181</port>
              </node>
              <node index="3">
                  <host>192.168.3.103</host>
                  <port>2181</port>
              </node>
          </zookeeper-servers>
      
          <!-- marcos每个实例配置不一样 -->
          <macros>
              <layer>01</layer>
              <shard>01</shard>
              <replica>cluster01-01-1</replica>
          </macros>
          <networks>
              <ip>::/0</ip>
          </networks>
      
          <!-- 数据压缩算法  -->
          <clickhouse_compression>
              <case>
                  <min_part_size>10000000000</min_part_size>
                  <min_part_size_ratio>0.01</min_part_size_ratio>
                  <method>lz4</method>
              </case>
          </clickhouse_compression>
      </yandex>
      ```

    - `server01`新增集群配置文件`/usr/soft/clickhouse-server/sub/conf/metrika.xml`

      ```xml
      <yandex>
          <!-- CH集群配置,所有服务器都一样 -->
          <clickhouse_remote_servers>
              <cluster_3s_1r>
                  <!-- 数据分片1  -->
                  <shard>
                      <internal_replication>true</internal_replication>
                      <replica>
                          <host>server01</host>
                          <port>9000</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                      <replica>
                          <host>server03</host>
                          <port>9001</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                  </shard>
                  <!-- 数据分片2  -->
                  <shard>
                      <internal_replication>true</internal_replication>
                      <replica>
                          <host>server02</host>
                          <port>9000</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                      <replica>
                          <host>server01</host>
                          <port>9001</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                  </shard>
                  <!-- 数据分片3  -->
                  <shard>
                      <internal_replication>true</internal_replication>
                      <replica>
                          <host>server03</host>
                          <port>9000</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                      <replica>
                          <host>server02</host>
                          <port>9001</port>
                          <user>default</user>
                          <password></password>
                      </replica>
                  </shard>
              </cluster_3s_1r>
          </clickhouse_remote_servers>
      
          <!-- zookeeper_servers所有实例配置都一样 -->
          <zookeeper-servers>
              <node index="1">
                  <host>192.168.3.117</host>
                  <port>2181</port>
              </node>
              <node index="2">
                  <host>192.168.3.105</host>
                  <port>2181</port>
              </node>
              <node index="3">
                  <host>192.168.3.103</host>
                  <port>2181</port>
              </node>
          </zookeeper-servers>
      
          <!-- marcos每个实例配置不一样 -->
          <macros>
              <layer>01</layer>
              <shard>02</shard>
              <replica>cluster01-02-2</replica>
          </macros>
          <networks>
              <ip>::/0</ip>
          </networks>
      
          <!-- 数据压缩算法  -->
          <clickhouse_compression>
              <case>
                  <min_part_size>10000000000</min_part_size>
                  <min_part_size_ratio>0.01</min_part_size_ratio>
                  <method>lz4</method>
              </case>
          </clickhouse_compression>
      </yandex>
      ```

    - 将`server01`新增的两个`metrika.xml`文件分发到`server02`,`server03`

      ```shell
      # server02
      scp -r /usr/soft/clickhouse-server/main/conf/metrika.xml server02:/usr/soft/clickhouse-server/main/conf
      scp -r /usr/soft/clickhouse-server/sub/conf/metrika.xml server02:/usr/soft/clickhouse-server/sub/conf
      # server03
      scp -r /usr/soft/clickhouse-server/main/conf/metrika.xml server03:/usr/soft/clickhouse-server/main/conf
      scp -r /usr/soft/clickhouse-server/sub/conf/metrika.xml server03:/usr/soft/clickhouse-server/sub/conf
      ```

    - 修改`server02`,`server03`的`metrika.xml`文件

      ```xml
      # server02 main
      <macros>
          <layer>01</layer>
          <shard>02</shard>
          <replica>cluster01-02-1</replica>
      </macros>
      # server02 sub
      <macros>
          <layer>01</layer>
          <shard>03</shard>
          <replica>cluster01-03-2</replica>
      </macros>
      # server03 main
      <macros>
          <layer>01</layer>
          <shard>03</shard>
          <replica>cluster01-03-1</replica>
      </macros>
      # server03 sub
      <macros>
          <layer>01</layer>
          <shard>02</shard>
          <replica>cluster01-01-2</replica>
      </macros>
      ```

      **至此，已经完成全部配置，其他的比如密码等配置，可以按需增加。**

11. 集群运行与测试

    在每一台服务器上依次运行实例，zookeeper前面已经提前运行，没有则需先运行zk集群

    **在每台服务器执行命令，唯一不同的参数是hostname**

    - 运行main实例

      ```shell
      docker run -d --name=ch-main -p 8123:8123 -p 9000:9000 -p 9009:9009 --ulimit nofile=262144:262144 \-v /usr/soft/clickhouse-server/main/data:/var/lib/clickhouse:rw \-v /usr/soft/clickhouse-server/main/conf:/etc/clickhouse-server:rw \-v /usr/soft/clickhouse-server/main/log:/var/log/clickhouse-server:rw \
      --add-host server01:192.168.3.117 \
      --add-host server02:192.168.3.105 \
      --add-host server03:192.168.3.103 \
      --hostname server01 \
      --network host \
      --restart=always \
       yandex/clickhouse-server
      ```

    - 运行sub实例

      ```shel
      docker run -d --name=ch-sub -p 8124:8124 -p 9001:9001 -p 9010:9010 --ulimit nofile=262144:262144 \
      -v /usr/soft/clickhouse-server/sub/data:/var/lib/clickhouse:rw \
      -v /usr/soft/clickhouse-server/sub/conf:/etc/clickhouse-server:rw \
      -v /usr/soft/clickhouse-server/sub/log:/var/log/clickhouse-server:rw \
      --add-host server01:192.168.3.117 \
      --add-host server02:192.168.3.105 \
      --add-host server03:192.168.3.103 \
      --hostname server01 \
      --network host \
      --restart=always \
       yandex/clickhouse-server
      ```

    - 在每台服务器的实例都启动之后，这里使用正版DataGrip来打开

    - 执行 `select * from system.clusters` 查询集群

    - 在任一实例上新建一个查询

      ```sql
      create table T_UserTest on cluster cluster_3s_1r
      (
          ts  DateTime,
          uid String,
          biz String
      )
          engine = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/T_UserTest', '{replica}')
              PARTITION BY toYYYYMMDD(ts)
              ORDER BY ts
              SETTINGS index_granularity = 8192;
      ```

      **cluster_3s_1r是前面配置的集群名称，需一一对应上， /clickhouse/tables/ 是固定的前缀，相关语法可以查看官方文档了。**

      刷新每个实例，即可看到全部实例中都有这张T_UserTest表，因为已经搭建zookeeper，很容易实现分布式DDL。

    - 新建Distributed分布式表

      ```sql
      CREATE TABLE T_UserTest_All ON CLUSTER cluster_3s_1r AS T_UserTest ENGINE = Distributed(cluster_3s_1r, default,  T_UserTest, rand())
      ```

      每个主分片分别插入相关信息：

      ```sql
      --server01
      insert into  T_UserTest values ('2021-08-16 17:00:00',1,1)
      --server02
      insert into  T_UserTest values ('2021-08-16 17:00:00',2,1)
      --server03
      insert into  T_UserTest values ('2021-08-16 17:00:00',3,1)
      ```

      **查询对应的副本表或者关闭其中一台服务器的docker实例，查询也是不受影响，时间关系不在测试**

### 三、结语
本篇博客介绍了如何借助Docker技术，通过使用Docker-Compose快速搭建ClickHouse+Zookeeper集群，并实现扩展节点、数据备份等资料处理方案，最终实现了高可靠性、高性能和高灵活性的大数据处理和分析。这种基于容器化、分布式技术的解决方案与传统中心式单体架构相比，具有更好的故障处理、增强效率、运维便利和易于管理等多方面的优越性。

在搭建过程中，我们深入探究了ClickHouse 和 Zookeeper 的特性及用途，在容器环境下运行它们并配置好了每个组件的网络和访问方法。我们同时关注到容器资源的调配策略、数据备份的独立存储、服务治理的容错性等问题，帮助读者全面掌握了 Docker 集群管理和优化的基本方法。

总之，本篇博客旨在提供一种端到端指导，使您更好地了解使用 Docker 搭建 ClickHouse+Zookeeper 集群的过程，可以在企业级大数据应用中发挥出强大的作用，并极大地促进业务发展。希望在今后的工作中为读者提供启示和指导，让技术更好的转化为实际价值。
