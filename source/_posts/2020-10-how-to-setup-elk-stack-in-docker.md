---
title: How to setup elk stack in docker
cover: /gallery/media/fsqs-tips-tricks-notes.png
date: 2020-10-16T10:08:22.921Z
categories:
  - ELK
tags:
  - elasticsearch
  - kibana
  - logstash
  - docker
---
The Elastic Stack (also known as the ELK Stack) is used across a variety of use cases — from observability to security, from enterprise search to business analytics.

<!-- more -->

> ELK is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana.

On this blog post, we will go through necessary steps to run elk using docker.

## Using docker

- Create a Docker network to enable communication between containers via container name.

```shell
 docker network create elk
```

- Run elasticsearch docker container.

```shell
docker run -d --name elasticsearch --net elk -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.9.2
```

- Create logstash configuration file `logstash.conf`.

```shell
cat <<EOF> ~/logstash.conf
input {
  tcp {
    port => 5044
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "example-%{appname}-%{env}"
  }
}
EOF
```

- Run logstash docker container.

```shell
docker run -d --name logstash --net elk -p 5044:5044 -v ~/logstash.conf:/usr/share/logstash/pipeline/logstash.conf logstash:7.9.2
```

- Run kibana docker container.
```shell
docker run -d --name kibana --net elk -e "ELASTICSEARCH_URL=http://elasticsearch:9200" -p 5601:5601 kibana:7.9.2
```

## Using docker-compose

- Create logstash configuration file `logstash.conf`.

```shell
cat <<EOF> ~/logstash.conf
input {
  tcp {
    port => 5044
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "example-%{appname}-%{env}"
  }
}
EOF
```

`docker-compose.yaml` configuration for elk stack:

```yaml
version: "3.1"
services:
  elasticsearch:
    image: elasticsearch:7.9.2
    environment:
      - discovery.type=single-node
    ports:
      - 9200
  logstash:
    image: logstash:7.9.2
    volumes:
      - ~/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    ports:
      - 5000:5000
    links:
      - elasticsearch
  kibana:
    image: kibana:7.9.2
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    ports:
      - 5601:5601
    links:
      - elasticsearch
```

Command to start elk:

```shell
docker-compose up -d
```

## References
- [https://www.elastic.co/what-is/elk-stack](https://www.elastic.co/what-is/elk-stack)