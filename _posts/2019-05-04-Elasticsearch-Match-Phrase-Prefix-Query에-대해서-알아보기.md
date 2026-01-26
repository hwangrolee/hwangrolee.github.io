---
layout: post
title: Elasticsearch Match Phrase Prefix Query에 대해서 알아보기
date: 2019-05-04 20:30:00 +0900
description: Elasticsearch Match Phrase Prefix Query에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Match-Phrase-Prefix-Query에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

**Match Phrase Query** 와 같지만 약간 다른 **Match Phrase Prefix Query**에 대해서 알아보겠습니다.

*Match Phrase Query*와 동일하게 동작하지만 마지막 term을 접두어로 일치하는 Document를 검색합니다.

#### example

*max_expansions*는 prefix를 확장할 최대값을 지정합니다.

```bash
curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "match_phrase_prefix": {
      "country": {
        "query":"south ko", "max_expansions": 10      }
    }
  }}'
```

*match_phrase_prefix*로 "south ko"를 검색하면 "south"는 필수로 들어가며 "ko"는 prefix로서 "south ko"이 일치하는 document를 검색합니다.

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 8104,
      "relation" : "eq"
    },
    "max_score" : 11.504026,
    "hits" : [
      ...
    ]
  }
}
```
