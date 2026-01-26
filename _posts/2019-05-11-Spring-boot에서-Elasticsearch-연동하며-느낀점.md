---
layout: post
title: Spring boot에서 Elastic search 연동하며 느낀점
date: 2019-05-11 22:05:00 +0900
description: Spring boot에서 Elasticsearch를 ORM으로 시도했다가 Rest Client로 바꾼 이유에 대해서 설명할게요.
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Spring-boot에서-Elasticsearch-연동하며-느낀점/
author: hwangrolee
tags: [ElasticSearch]
---

Spring boot에서 Elasticsearch를 ORM으로 시도했다가 Rest Client로 바꾼 이유에 대해서 설명할게요.

#### 시작은 ORM으로 개발하기

elasticsearch7 을 docker로 설치하여 curl로 테스트를 하다 spring boot로 테스트를 하고 싶어 spring boot 2.1.4 기반으로 개발을 해 보았습니다.

spring boot에서 elasticsearch와 연동할때 [spring-boot-starter-data-elasticsearch](https://mvnrepository.com/artifact/org.springframework.data/spring-data-elasticsearch/3.1.6.RELEASE)을 사용했는데 아직까진 elasticsearch 7에는 연결이 안되더군요...

처음 시도하는거라 설정이 잘못된건가? spring에서 docker로 설치한 elasticsearch에 접근하지 못한건가? 싶어 많은 시간 삽질을 했습니다.

그 결과, spring-data-elasticsearch와 elasticsearch는 버전에 큰 영향을 받기 때문에 연동시 문제가 될 수 있다고 합니다.

그래서 제가 선택한 것은 ORM을 사용해보고 싶은 마음에 elasticsearch를 6.5로 downgrade 했고 spring-boot-starter-data-elasticsearch를 사용해 보았습니다.

#### ORM에서 Rest Client로 바꾼 이유

결론은 저의 코딩스타일을 살리기 힘들어서 이 방법은 포기하고 Rest Client로 공부를 시작했습니다.

> 저는 코딩시 [**CamelCase**](https://en.wikipedia.org/wiki/Camel_case)를 선호하고 저장소(database, elasticsearch, redis, ETC.)는 [**SnakeCase**](https://en.wikipedia.org/wiki/Snake_case)를 선호합니다.

> spring boot에서 **@JsonProperty**로 Elasticsearch field를 지정할 수 있었지만 elasticsearch 6.2.2 이후에는 지원하지 않아 ORM을 통해 Elasticsearch를 관리하고 싶다면 코딩을 **SnakeCase**로 바꾸거나 Elasticsearch field명을 **CamelCase**로 바꾸어야 하지만 그러긴 싫었습니다.

#### 장점

- rest client로 연동할 경우 elasticsearch가 업데이트 될 경우 spring-data-elasticsearch가 새로운 elasticsearch를 지원할때까지 기다리지 않고 사용할 수 있습니다.
- elasticsearch의 다양한 쿼리는 직접 만들어볼 수 있습니다. 신나겠다.
- 직접 쿼리를 만들어볼 수 있는 만큼 튜닝도 가능하니 성능을 향상시키기 위해 많은 공부를 할 수 있을거에요.

#### 단점

- 직접 쿼리를 만들어야하니 귀찮습니다.
