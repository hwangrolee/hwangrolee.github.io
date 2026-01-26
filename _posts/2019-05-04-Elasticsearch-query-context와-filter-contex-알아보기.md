---
layout: post
title: Elasticsearch query context와 filter context 알아보기
date: 2019-05-04
description: Elasticsearch query context와 filter context 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-query-context와-filter-contex-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

안녕하세요. Elastic Stack을 공부하고 있는 서버개발자입니다.

logstash를 통해 elastic search에 데이터를 넣은 후 검색에 대한 공부를 진행하였습니다.

오늘은 query context와 filter context가 어떻게 다른지를 알아보았습니다.

우선, 이 포스팅을 보시는 분들은 이미 elasticsearch를 설치하였고 공부를 시작했다고 가정하겠습니다.

만약 elsticsearch를 설치하지 않았으며 설치를 했지만 데이터가 없다면 제가 작성한 **[logstash를 사용해서 CSV파일을 elastic search에 임포트하기](https://hwangrolee.github.io/blog/logstash를-사용해서-CSV파일을-elastic-search에-임포트하기/)**편을 참고해보세요.

---

Elastic search를 통해 검색을 할 경우 **쿼리 콘텍스트(query context)**와 **필터 콘텍스트(filter context)**에 따라 달라집니다.

#### Query context

Query context는 쿼리 절이 "쿼리가 문서와 일치 하는가?"라는 질문에 답하며 \_score를 통해 문서가 다른 문서와 비교하여 쿼리와 얼마나 일치하는지를 계산합니다.

#### Filter Context

Filter Context는 쿼리절이 "쿼리가 문서와 일치하는가?"라는 질문에 답하지만 \_score는 계산하지 않습니다.

대부분 필터링을 위해 사용합니다.

#### example

> 예제를 통해 무엇이 다른지 확인해 볼게요

- _message_ 필드에 "africa"가 포함된 것.
- _item_type_ 필드가 "clothes" 인 것.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "bool": {
      "must": [
        {"match": {"message": "africa"}}
      ],
      "filter": [
        {"term": {"item_type": "clothes"}}
      ]}
    },
    "size": 2
}'
```

*"query"*는 query context에서 동작합니다. _"bool"_ 과 *"must"*는 query context에서 사용됩니다. 얼마나 일치하는 지를 평가하는데 사용됩니다.

*"filter*는 filter context에서 동작하며 *"term"*은 filter context에서 사용됩니다. 일치하는 문서는 걸러내지만 점수에는 영향을 주지 않습니다.

```json
{
  "took": 17,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": 1.2925828,
    "hits": [
      {
        "_index": "sales-records",
        "_type": "_doc",
        "_id": "Dn9aY2oBml718jk9tcNd",
        "_score": 1.2925828,
        "_source": {
          "total_profit": 37674.72,
          "sales_channel": "Online",
          "order_id": 653758466,
          "units_sold": 513,
          "tags": ["_dateparsefailure"],
          "total_revenue": 56060.64,
          "unit_cost": 35.84,
          "country": "South Africa",
          "order_priority": "M",
          "ship_date": "3/16/2013",
          "unit_price": 109.28,
          "total_cost": 18385.92,
          "item_type": "Clothes",
          "region": "Sub-Saharan Africa",
          "@version": "1",
          "host": "c5ceb9b17956",
          "order_date": "2/21/2013",
          "path": "/config-dir/1500000 Sales Records.csv",
          "message": "Sub-Saharan Africa,South Africa,Clothes,Online,M,2/21/2013,653758466,3/16/2013,513,109.28,35.84,56060.64,18385.92,37674.72\r",
          "@timestamp": "2019-04-28T09:50:49.700Z"
        }
      },
      {
        "_index": "sales-records",
        "_type": "_doc",
        "_id": "nn9aY2oBml718jk9tcOa",
        "_score": 1.2925828,
        "_source": {
          "total_profit": 28054.08,
          "sales_channel": "Online",
          "order_id": 669640576,
          "units_sold": 382,
          "tags": ["_dateparsefailure"],
          "total_revenue": 41744.96,
          "unit_cost": 35.84,
          "country": "South Africa",
          "order_priority": "C",
          "ship_date": "11/21/2015",
          "unit_price": 109.28,
          "total_cost": 13690.88,
          "item_type": "Clothes",
          "region": "Sub-Saharan Africa",
          "@version": "1",
          "host": "c5ceb9b17956",
          "order_date": "11/17/2015",
          "path": "/config-dir/1500000 Sales Records.csv",
          "message": "Sub-Saharan Africa,South Africa,Clothes,Online,C,11/17/2015,669640576,11/21/2015,382,109.28,35.84,41744.96,13690.88,28054.08\r",
          "@timestamp": "2019-04-28T09:50:49.789Z"
        }
      }
    ]
  }
}
```

정확한 검색이 아닌 유사검색(문자 검색)과 같은 것은 query context를 사용하고 정확한 검색을 원할때는 filter context를 사용하면 될 것 같습니다.
