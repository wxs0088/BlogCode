---
title: 容器中的洞察：使用 Docker-Compose 部署 Elasticsearch 和 Kibana
date: 2024-01-05 14:09:40
img: https://wangxs020202.gitee.io/pbad/background/es_kibana_docker.png
top: false
summary: 本文将介绍如何使用Docker-Compose来部署Elasticsearch和Kibana。
categories: Elasticsearch
tags:
  - Elasticsearch
  - Kibana
  - Docker
  - Docker-Compose
  - 容器
  - 搜索引擎
  - 智能客服
---

### 一、写在前面

在大语言模型的时代，向量数据库无疑是最火的数据库之一。`milvus`作为向量数据库赛道的领先者，自2019 年正式开源以来，已经成长为全球最大、最活跃的向量数据库开源项目与开发者社区。但是，身为
老款数据库的`Elasticsearch`也不甘示弱，作为全球下载量最多的向量数据库，我们可以很方便地利用它来帮我们进行计算向量之间的相似性。。本文将介绍如何使用`Docker-Compose`来部署`Elasticsearch`和`Kibana`。

### 二、Docker-Compose的安装

#### 2.1 安装Docker

```bash
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```

#### 2.2 安装Docker-Compose

```bash
sudo apt-get install docker-compose
```

### 三、构建容器编排脚本

#### 3.1 创建目录

```bash
mkdir es_kibana
cd es_kibana
```

#### 3.2 创建docker-compose.yml文件

```bash
vim docker-compose.yml
```

#### 3.3 编写docker-compose.yml文件

```yml
version: "3.1"
# 服务配置
services:
  elasticsearch:
    container_name: elasticsearch-8.8.1
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.1
    # 用来给容器root权限（不安全）可移除
    privileged: true
    # 在linux里ulimit命令可以对shell生成的进程的资源进行限制
    ulimits:
      memlock:
        soft: -1
        hard: -1
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "http.host=0.0.0.0"
      - "node.name=elastic01"
      - "cluster.name=cluster_elasticsearch"
      - "discovery.type=single-node"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      #- ./elasticsearch/config:/usr/share/elasticsearch/config
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/plugin:/usr/share/elasticsearch/plugins
    networks:
      - elastic_net
  kibana:
    container_name: kibana-8.8.1
    image: docker.elastic.co/kibana/kibana:8.8.1
    ports:
      - "5601:5601"
    #volumes:
    #  - ./kibana/config:/usr/share/kibana/config
    networks:
      - elastic_net
# 网络配置
networks:
  elastic_net:
    driver: bridge
```

### 四、构建相关目录

#### 4.1 创建elasticsearch相关目录

```bash
mkdir elasticsearch
mkdir elasticsearch/data
mkdir elasticsearch/plugin
```

#### 4.2 创建kibana相关目录

```bash
mkdir kibana
```

#### 4.3 更改目录权限

```bash
sudo chmod -R 777 elasticsearch
sudo chmod -R 777 kibana
```

### 五、启动容器

```bash
sudo docker-compose up -d
```

**错误排除**

在拉取镜像的时候，可能会出现如下错误：

```bash
failed to register layer: exit status 22: unpigz: abort: zlib version less than 1.2.3
```

这是因为`docker`的`zlib`版本过低导致的，可以通过如下命令进行升级：

```bash
# 升级zlib
cd /usr/local/
wget https://github.com/madler/pigz/archive/refs/tags/v2.8.tar.gz
tar -zxf v2.8.tar.gz
cd pigz-2.8
make
# 备份原来的zlib
which pigz
# /usr/bin/pigz

which unpigz
# /usr/bin/unpigz

mv /usr/bin/pigz /usr/bin/pigz.bak
mv /usr/bin/unpigz /usr/bin/unpigz.bak
# 将新编译的pigz和unpigz拷贝到/usr/bin目录下
cd /usr/local/pigz-2.8
cp pigz /usr/bin/
cp unpigz /usr/bin/
```

然后重新拉取镜像即可。

### 六、映射目录

#### 6.1 elasticsearch

```bash
sudo docker cp elasticsearch-8.8.1:/usr/share/elasticsearch/config ./elasticsearch/config
```

#### 6.2 kibana

```bash
sudo docker cp kibana-8.8.1:/usr/share/kibana/config ./kibana/config
```

### 七、配置elasticsearch

#### 7.1 修改elasticsearch.yml

```yml
# 集群节点名称
node.name: "es01"
# 设置集群名称为elasticsearch
cluster.name: "cluster_es"
# 网络访问限制
network.host: 0.0.0.0
# 以单一节点模式启动
discovery.type: single-node


# 是否支持跨域
http.cors.enabled: true
# 表示支持所有域名
http.cors.allow-origin: "*"
# 内存交换的选项，官网建议为true
bootstrap.memory_lock: true


# 修改安全配置 关闭 证书校验
xpack.security.http.ssl:
  enabled: false
xpack.security.transport.ssl:
  enabled: false
```

### 八、配置kibana

#### 8.1 修改kibana.yml

```yml
#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
# 国家化配置中文
i18n.locale: zh-CN
server.host: "0.0.0.0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
server.port: 5601
```

### 九、修改脚本，重启容器

在 `docker-compose.yml` 文件中，我们将之前注释的部分取消注释，然后重启容器即可。

```yml
version: "3.1"
# 服务配置
services:
  elasticsearch:
    container_name: elasticsearch-8.8.1
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.1
    # 用来给容器root权限（不安全）可移除
    privileged: true
    # 在linux里ulimit命令可以对shell生成的进程的资源进行限制
    ulimits:
      memlock:
        soft: -1
        hard: -1
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "http.host=0.0.0.0"
      - "node.name=elastic01"
      - "cluster.name=cluster_elasticsearch"
      - "discovery.type=single-node"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - ./elasticsearch/config:/usr/share/elasticsearch/config
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/plugin:/usr/share/elasticsearch/plugins
    networks:
      - elastic_net
  kibana:
    container_name: kibana-8.8.1
    image: docker.elastic.co/kibana/kibana:8.8.1
    ports:
      - "5601:5601"
    volumes:
      - ./kibana/config:/usr/share/kibana/config
    networks:
      - elastic_net
# 网络配置
networks:
  elastic_net:
    driver: bridge
```

```bash
sudo docker-compose up -d
```

### 十、设置密码

#### 10.1 重置 elastic 用户密码

```bash
docker exec -it elasticsearch-8.8.1 /usr/share/elasticsearch/bin/elasticsearch-reset-password -uelastic
```

输出如下：

```bash
This tool will reset the password of the [elastic] user to an autogenerated value.
The password will be printed in the console.
Please confirm that you would like to continue [y/N]y


Password for the [elastic] user successfully reset.
New value: xxxxxxxxxxxxxxxxxxx
```

#### 10.2 重置 kibana_system 用户密码

```bash
docker exec -it elasticsearch-8.8.1 /usr/share/elasticsearch/bin/elasticsearch-reset-password -ukibana_system
```

输出如下：

```bash
This tool will reset the password of the [kibana_system] user to an autogenerated value.
The password will be printed in the console.
Please confirm that you would like to continue [y/N]y


Password for the [kibana_system] user successfully reset.
New value: 
```

### 十一、配置kibana，并登录kibana

#### 11.1 配置kibana

```bash
vim kibana/config/kibana.yml
```

```yml
elasticsearch.username: kibana_system
elasticsearch.password: xxxxxxxxxxxxxxxxxxx
```

xxxxxxxxxxxxxxxxxxx 为上一步中重置 kibana_system 用户密码后的值。

#### 11.2 重启容器

```bash
sudo docker-compose up -d
```

#### 11.3 登录kibana

在浏览器中输入`http://ip:5601`，进入kibana登录界面，输入`elastic`用户对应的密码即可登录。


### 十二、总结

使用docker-compose部署Elasticsearch和Kibana非常简单，只需要几步就可以完成。相比于传统的部署方式，使用docker-compose部署可以大大减少部署的时间，提高部署的效率。
此时，我们已经可以使用Elasticsearch和Kibana来进行向量的相似性计算了。



