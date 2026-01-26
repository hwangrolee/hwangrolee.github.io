---
layout: post
title: Elasticsearch Fuzziness에 대해서 알아보기
date: 2019-05-04
description: Elasticsearch Fuzziness에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Fuzziness에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

Match Query를 공부 중 Fuzziness 개념을 알게 되었습니다.

일부쿼리 및 API는 _fuzziness_ 파라미터를 사용하여 정확하지 않은 퍼지매칭을 허용합니다.

**text**혹은 **keyword**쿼리시 **fuzziness**는 Levenshtein distance로 해석됩니다. _Levenshtein distance_ 단어, 혹은 문장과 같은 string 간의 형태적 유사성을 정의하는 방법입니다.
하지만 한글처럼 각글자가 초/중/종성으로 이루어진 언어를 고려한 방법이 아니라고 합니다. 참고하세요

**fuzziness**의 허용되는 최대 거리 Levenshtein Edit Distance (또는 편집 횟수)는
_0_,_1_,_2_ 입니다.

_AUTO_ 는 용어의 길이에 따라 편집거리를 생성합니다.

_Fuzziness_ 를 사용한 검색은 더 공부해봐야 겠지만 보통 *AUTO*만으로도 충분한 기대치를 얻을 수 있는 것 같습니다.

#### example1

> *AUTO*를 테스트해봅니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match": {
      "message": {
        "query":"cloth",
        "fuzziness":"AUTO"
      }
    }
  },
  "size": 2
}'
```

_AUTO_ 로 하니 결과가 나오질 않았네요

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

#### example2

> _cloth_ 만 입력후 *fuzziness*를 2로 입력하니 clothes에 대한 결과물을 받아왔습니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match": {
      "message": {
        "query":"cloth",
        "fuzziness":"2"
      }
    }
  },
  "size": 2
}'
```

```json
{
  "took": 13,
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
    "max_score": 2.6643744,
    "hits": [
      {
        "_index": "sales-records",
        "_type": "_doc",
        "_id": "Nn9aY2oBml718jk9tMLH",
        "_score": 2.6643744,
        "_source": {
          "total_profit": 566589.6,
          "sales_channel": "Offline",
          "order_id": 800003659,
          "units_sold": 7715,
          "tags": ["_dateparsefailure"],
          "total_revenue": 843095.2,
          "unit_cost": 35.84,
          "country": "Cote d'Ivoire",
          "order_priority": "C",
          "ship_date": "4/6/2016",
          "unit_price": 109.28,
          "total_cost": 276505.6,
          "item_type": "Clothes",
          "region": "Sub-Saharan Africa",
          "@version": "1",
          "host": "c5ceb9b17956",
          "order_date": "2/19/2016",
          "path": "/config-dir/1500000 Sales Records.csv",
          "message": "Sub-Saharan Africa,Cote d'Ivoire,Clothes,Offline,C,2/19/2016,800003659,4/6/2016,7715,109.28,35.84,843095.20,276505.60,566589.60\r",
          "@timestamp": "2016-06-04T00:00:00.000Z"
        }
      },
      {
        "_index": "sales-records",
        "_type": "_doc",
        "_id": "4n9aY2oBml718jk9v9SH",
        "_score": 2.6643744,
        "_source": {
          "total_profit": 621228.96,
          "sales_channel": "Offline",
          "order_id": 614078938,
          "units_sold": 8459,
          "tags": ["_dateparsefailure"],
          "total_revenue": 924399.52,
          "unit_cost": 35.84,
          "country": "Cote d'Ivoire",
          "order_priority": "H",
          "ship_date": "11/28/2011",
          "unit_price": 109.28,
          "total_cost": 303170.56,
          "item_type": "Clothes",
          "region": "Sub-Saharan Africa",
          "@version": "1",
          "host": "c5ceb9b17956",
          "order_date": "11/17/2011",
          "path": "/config-dir/1500000 Sales Records.csv",
          "message": "Sub-Saharan Africa,Cote d'Ivoire,Clothes,Offline,H,11/17/2011,614078938,11/28/2011,8459,109.28,35.84,924399.52,303170.56,621228.96\r",
          "@timestamp": "2019-04-28T09:50:52.295Z"
        }
      }
    ]
  }
}
```

상황에 따라 "AUTO" 혹은 0,1,2 를 선택하여 진행하시면 될 것 같습니다.
