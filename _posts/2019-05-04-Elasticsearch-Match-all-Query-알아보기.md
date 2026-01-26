---
layout: post
title: Elasticsearch Match all Query 알아보기
date: 2019-05-04 01:33:00 +0900
description: Elasticsearch Match all Query 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Match-all-Query-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

안녕하세요. Elastic Stack을 공부하고 있는 서버개발자입니다.

Match All Query에 대해서 공부를 했습니다.

**match_all**, **boost**, **match_none**에 대해서 알아보겠습니다.

실제로는 쓰이지 않을 쿼리이지만 Elastic search document에 있어서 공부했습니다.

#### example1

> 모든 문서와 일치하며 \_score에 1.0을 적용합니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match_all": {}
  },
  "size": 1
}'
```

```json
{
  "took": 1,
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
    "max_score": 1.0,
    "hits": [
      {
        "_index": "sales-records",
        "_type": "_doc",
        "_id": "K39aY2oBml718jk9UEiK",
        "_score": 1.0,
        "_source": {
          "total_profit": 442415.04,
          "sales_channel": "Online",
          "order_id": 255331122,
          "units_sold": 7008,
          "tags": ["_dateparsefailure"],
          "total_revenue": 1079652.48,
          "unit_cost": 90.93,
          "country": "Lesotho",
          "order_priority": "M",
          "ship_date": "11/27/2011",
          "unit_price": 154.06,
          "total_cost": 637237.44,
          "item_type": "Vegetables",
          "region": "Sub-Saharan Africa",
          "@version": "1",
          "host": "c5ceb9b17956",
          "order_date": "11/23/2011",
          "path": "/config-dir/1500000 Sales Records.csv",
          "message": "Sub-Saharan Africa,Lesotho,Vegetables,Online,M,11/23/2011,255331122,11/27/2011,7008,154.06,90.93,1079652.48,637237.44,442415.04\r",
          "@timestamp": "2019-04-28T09:50:20.507Z"
        }
      }
    ]
  }
}
```

#### example2

> *\_score*는 _boost_ 파라미터를 통해 변경할 수 있습니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match_all": {
      "boost": 1.2
    }
  },
  "size": 1
}'
```

```json
{
  "took": 2,
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
    "max_score": 1.2,
    "hits": [
      {
        "_index": "sales-records",
        "_type": "_doc",
        "_id": "K39aY2oBml718jk9UEiK",
        "_score": 1.2,
        "_source": {
          "total_profit": 442415.04,
          "sales_channel": "Online",
          "order_id": 255331122,
          "units_sold": 7008,
          "tags": ["_dateparsefailure"],
          "total_revenue": 1079652.48,
          "unit_cost": 90.93,
          "country": "Lesotho",
          "order_priority": "M",
          "ship_date": "11/27/2011",
          "unit_price": 154.06,
          "total_cost": 637237.44,
          "item_type": "Vegetables",
          "region": "Sub-Saharan Africa",
          "@version": "1",
          "host": "c5ceb9b17956",
          "order_date": "11/23/2011",
          "path": "/config-dir/1500000 Sales Records.csv",
          "message": "Sub-Saharan Africa,Lesotho,Vegetables,Online,M,11/23/2011,255331122,11/27/2011,7008,154.06,90.93,1079652.48,637237.44,442415.04\r",
          "@timestamp": "2019-04-28T09:50:20.507Z"
        }
      }
    ]
  }
}
```

#### example3

> *match_all*의 반대이며 아무 결과도 반환하지 않습니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match_none": {}
  }}'
```

```json
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```
