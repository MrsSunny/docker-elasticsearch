# docker-elasticsearch部署

docker部署Elasticsearch Cluster


## 安装 docker ##

安装docker 查看官方文档：针对每个系统版本进行安装。

```

https://docs.docker.com/installation/#installation

```
## 启动 docker ##

```
$ sudo service docker start

```

##编写Dockerfile##

```

FROM     debian:jessie
MAINTAINER Sunny "liuzhenx@hotmail.com"

RUN apt-get update

RUN apt-get install -y vim wget curl
ENV Elasticsearch_version elasticsearch-1.7.0
RUN echo "root:root" |chpasswd


RUN \
  echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list && \
  echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list && \
  apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886 && \
  apt-get update

RUN \
  echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  apt-get install -y oracle-java8-installer &&\
  apt-get clean

RUN echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> /etc/environment

RUN mkdir /usr/es
WORKDIR /usr/es

RUN wget https://download.elastic.co/elasticsearch/elasticsearch/$Elasticsearch_version.tar.gz

RUN tar -zxf $Elasticsearch_version.tar.gz
WORKDIR $Elasticsearch_version/config

RUN mv elasticsearch.yml elasticsearch.yml.bak

WORKDIR /usr/es

EXPOSE 9200 9300

```


## 根据Dockerfile制作 images##

Dockerfile文件路径：docker/

```
sudo docker build -t debian/elasticsearch .

编译完成后查看镜像
$ sudo docker images

REPOSITORY                   TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
debian/elasticsearch         latest              5f0668a6b9k0        About an hour ago   899.1 MB

```

##安装weave##

weave安装请看我的 https://github.com/MrsSunny/docker-weave这个项目

##配置Elasticsearch##
elasticsearch.yml 内容如下

```

cluster.name: aaaaaaa-name
node.name: nade_01
index.number_of_shards: 8
index.number_of_replicas: 2
network.host: 10.0.0.1
transport.tcp.port: 9300
transport.tcp.compress: true
path.data: /usr/es/data
path.logs: /usr/es/logs
http.port: 9200
http.enabled: true
node.master: true
node.data: true
index.store.type: mmapfs
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping.timeout: 90s
indices.fielddata.cache.size: 40%
bootstrap.mlockall: true
gateway.expected_nodes: 4
indices.recovery.max_size_per_sec: 100mb
index.cache.field.type: soft
node.rack_id: 101150
cluster.routing.allocation.awareness.attributes: rack_id

```

配置事项：

1.indices.fielddata.cache.size 这个值如果不用doc_values的话最好设置一下，默认是无上限的。
不过最好启用doc_values ，Elasticsearch官方也说明在以后的版本中doc_values会成为默认值。

2.因为只有两台物理机，并且配置也不一样所以设置了一下分片规则。

##关闭swap##
由于swap会影响Elasticsearch的性能，所以应该关闭swap

请看Es官方提供的方法：
https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration.html

注意：官网上说的方法不能在容器里面执行，因为swap是系统功能，应该在宿主机上禁用swap。
docker 容器里面的root的用户如果不是特权容器是没有办法修改这些参数的。还有就是容器最好不要用特权启动，那样容器里面的root和外面的root权限就一样了，很危险。

##修改fd 参数##

和swap修改的方式一样，需要在宿主机上修改，
sysctl -w vm.max_map_count=262144
这个事官网修改的值（没有测试），要想彻底修改这个值需要修改/etc/sysctl.conf这个文件，在里面加上vm.max_map_count=262144
##启动容器##

因为要用weave实现cluster，所以容器用weave启动


```

sudo weave run 10.0.0.1/24 --name "dataNode1" -m 9g -v /opt/conf/data_node_1:/usr/es/elasticsearch-1.7.0/config -v /opt/data/data_node_1:/usr/es/data -v /opt/logs/data_node_1:/usr/es/logs debian/elasticsearch /usr/es/elasticsearch-1.7.0/bin/elasticsearch -D

```

注意：要把es三个目录挂在外面，配置文件目录、数据目录、日志目录。

这样用docker结合weave 就能实现跨主机的cluster。
