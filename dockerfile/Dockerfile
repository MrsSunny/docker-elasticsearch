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
  apt-get install -y oracle-java8-installer && \
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
