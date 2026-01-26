---
layout: post
title: Elasticsearch Multi Match Query에 대해서 알아보기
date: 2019-05-08 19:35:00 +0900
description: Elasticsearch Multi Match Query에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Multi-Match-Query에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

쿼리를 시도할때 검색어를 하나의 필드가 아닌 여러개의 필드를 통해 검색을 하고 싶다면 **Multi Match Query**를 사용하세요.

단, 한번에 검색할 수 있는 필드 수에는 제한이 있으며 기본값은 1024개이며 _indices.query.bool.max_clause_count_ 검색설정에 정의되어있습니다.

만약 검색하려는 필드가 비어있다면 모든 필드를 검색합니다. 단, 매핑타입에 맞는 필드로 한정하는 것 같습니다. 하지만 검색하려는 필드가 비어있을 일은 거의 없겠죠?

#### example1

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "multi_match": {
      "query": "south",
      "fields": ["country", "message"]
    }
  }
}'
```

*"south"*라는 검색어를 _country_, *message*필드에서 검색합니다.

```json
{
  "took" : 23,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 4.295763,
    "hits" : [
      ...
    ]
  }
}
```

위 예제는 기본적인 Multi Match Query입니다.

이외에도 유형을 정의하여 검색을 할 수 있습니다.

### best_fields

*best_fields*는 동일한 필드에서 여러 단어를 검색할때 유용합니다. 그리고 정확도가 높은 필드의 *score*를 사용하지만 *tie_breaker*가 지정되 있다면 정확도가 높은 필드의 _\_score_ + 다른 모든 필드의 (_tie_breaker_ \* _\_score_)로 score를 계산합니다.

*best_fields*의 동작방식은 각 필드에 대해 *match query*를 생성하고 _dis_max_ 쿼리에서 래핑하여 가장 일치하는 단일 필드를 찾습니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "multi_match": {
      "query": "south",
      "fields": ["country", "*ssage"],
      "type": "best_fields",
      "tie_breaker":0.3
    }
  }
}'
```

위와 같은 쿼리는 다음과 같습니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "country": "south" }},
        { "match": { "message": "south" }}
      ],
      "tie_breaker": 0.3
    }
  }
}'
```

### most_fields

*most_fields*는 다른 방식으로 같은 텍스트를 analyze 해서 여러 필드를 쿼리할때 유용합니다.

예를들어 필드가 다른 형태소 분석기를 쓰는 경우입니다.

### cross_fields

*cross_fields*는 모든 필드를 하나의 필드 처럼 보고 검색을 진행합니다.

```bash
$ curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "multi_match": {
      "query": "asia korea",
      "type": "cross_fields",
      "operator": "and",
      "fields": ["region", "country"]
    }
  }
}'

```

### phrase

각 필드에서 _match_phrase_ 쿼리를 실행하고 최상의 필드에서 *\_score*를 사용합니다.

### phrase_prefix

각 필드에서 _match_phrase_prefix_ 쿼리를 실행하고 각 필드의 *\_score*를 사용합니다.
