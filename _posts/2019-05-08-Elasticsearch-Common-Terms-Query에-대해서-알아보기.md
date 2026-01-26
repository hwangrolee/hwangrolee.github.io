---
layout: post
title: Elasticsearch Common Terms Query에 대해서 알아보기
date: 2019-05-09
description: Elasticsearch Common Terms Query에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Common-Terms-Query에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

쿼리의 모든 term에는 비용이 있습니다.

만약 "The brown fox"를 검색하려면 "the", "brown", "fox"로 검색을 하게 되며 "the"의 경우 많은 문서와 일치하므로 다른 두 용어보다 영향력이 적습니다.

이러한 경우에 예전에는 빈도가 높은 용어를 무시하는 것이었습니다. "the"를 *stopword*로 처리하여 인덱스크기를 줄이고 실행하는 term 쿼리의 수를 줄이는 것입니다.

이 방법의 문제점 *stopwords*가 관련성에는 미치는 영향이 적지만 중요한 부분도 존재합니다. *stopword*를 제거할 경우 정확도는 떨어지게됩니다.

예를 들어 "happy" 와 "not happy"를 구분할 수 없습니다.

*common terms query*는 *query terms*를 더 중요한 그룹, 덜 중요한 그룹으로 나뉩니다.

1. 중요한 용어와 일치하는 문서를 검색합니다. 이는 검색으로 나온 결과 문서가 적을 수록 관련성에 더 큰 영향력을 준다는 말이 됩니다.

2. 덜 중요한 용어(빈번하게 존재하며 낮은 관련성을 띄는 단어)에 대한 쿼리를 실행합니다. 그러나 모든 문서에 대한 관련성 점수를 계산하는 대신 첫번째 쿼리에서 이미 일치하는 문서의 **\_score** 만 계산합니다.

이러한 방식으로 _high frequency terms_(많이 존재하는 단어?)에 대한 성능 저하 비용을 지불하지 않고도 관련도 계산을 향상시킬 수 있습니다.

만약 쿼리에 *high frequency terms*로만 이루어져 있다면 _and_ 쿼리로 실행합니다.

Terms는 _relative frequency_(0.0 .. 1.0) 혹은 _absolute frequency_(>=1)로 지정할 수 있는 **cutoff_frequency**에 따라 높거나 낮은 빈도 그룹으로 할당됩니다.

### example

```bash
$ curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "common": {
      "country": {
        "query": "Republic of",
        "cutoff_frequency": 0.001,
        "low_freq_operator": "and"
      }
    }
  }
}'
```
