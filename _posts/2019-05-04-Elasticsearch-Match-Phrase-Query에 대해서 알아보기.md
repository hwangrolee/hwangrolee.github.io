---
layout: post
title: Elasticsearch Match Phrase Query에 대해서 알아보기
date: 2019-05-04
description: Elasticsearch Match Phrase Query에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Match-Phrase-Query에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

오늘은 Match Phrase Query란 무엇인가에 대해서 알아보겠습니다.

*match_phrase query*는 text를 분석하고 분석된 텍스트를 *phrase query*로 만듭니다.

*match query*는 띄어쓰기가 포함된 경우 analyze 하면 띄어쓰기로 token을 생성하여 token 중 하나라도 일치하면 document에 포함한다.

반면, *match_phrase query*는 띄어쓰기를 포함한 정확한 검색을 하고 싶을때 사용하면 된다.

#### _match query_ 샘플

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match": {
      "message": {
        "query":"canada,snacks/online,c,9/13"      }
    }
  }}'
```

*match query*로 할 경우 결과는 10,000개로 나왔네요.

그 이유는 검색어를 *","*로 analyze하여 token을 생성하고 각 token이 포함된 document를 결과로 받아오기 때문입니다.

```json
{
  "took" : 35,
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
    "max_score" : 15.147114,
    "hits" : [
      ...
    ]
  }
}
```

#### _match_phrase query_ 샘플

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match_phrase": {
      "message": {
        "query":"canada,snacks/online,c,9/13"      }
    }
  }}'
```

*match_phrase query*는 *","*로 analyze하지 않아 _canada,snacks/online,c,9/13_ 토큰 그대로 검색을 합니다. 결과는 1개만 나왔습니다.

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 15.147113,
    "hits" : [
      ...
    ]
  }
}
```

검색어에 대해서 유사검색은 *match query*를 검색어가 정확하게 포함된 document를 원할 경우 *match_phrase query*를 사용하면 좋을 것 같습니다.
