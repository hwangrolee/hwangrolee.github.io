---
layout: post
title: Elasticsearch Simple Query String Query에 대해서 알아보기
date: 2019-05-09 22:45:00 +0900
description: Elasticsearch Simple Query String Query에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Simple-Query-String-Query에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

*query_string*과는 달리 *simple_query_string*쿼리는 예외를 throw하지 않으며 쿼리의 잘못된 부분을 삭제합니다.

*simple_query_string*은 특수문자를 통해 연산자를 선언할 수 있습니다.

**+** : AND operation

**\|** : OR operation

**-** : 싱글 토큰을 무시합니다.

**"** : 검색어 구문 그대로 검색하기 위해 사용합니다.

**\*** : 단어의 끝에 선언하는 prefix query

**(** **)** : 구문을 감쌀때 사용

#### example

```bash
$ curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "simple_query_string": {
      "fields": ["country"],
      "query": "kor* -south"
    }
  }
}'
```

kor로 시작하는 단어가 들어가며 south라는 단어는 빼고 검색합니다.

> 참고사이트
>
> 1. https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-simple-query-string-query.html
