---
layout: post
title: Elasticsearch Query String Query에 대해서 알아보기
date: 2019-05-09
description: Elasticsearch Query String Query에 대해서 알아보기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Elasticsearch-Query-String-Query에-대해서-알아보기/
author: hwangrolee
tags: [ElasticSearch]
---

*query string*쿼리는 연산자를 중심으로 텍스트를 분할하여 쿼리를 분석합니다.

#### example

```bash
$ curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "query_string": {
      "default_field": "country",
      "query": "(south africa) OR (south korea)"
    }
  }
}'
```

"south africa"와 "south korea"로 나뉘며 각 부분은 분석기에 의해서 독립적으로 분석됩니다.

공백은 연산자로 간주되지 않아 공백 상태 그대로 분석기에 전달됩니다. 만약 각 단어를 개별적으로 쿼리하길 원한다면 단어에 연산자를 추가해야합니다.

예를 들어, "south africa"을 검색하시려면 (south AND africa)으로 입력하시면 되며 "(south africa) OR (south korea)"는 "(south AND africa) OR (south AND korea)" 로 하시면 됩니다.

여러 필드를 통해 검색을 하고 싶다면 _type_ 속성을 사용하시면 됩니다. 기본값은 *"best_fields"*입니다. 그리고 *fields*를 검색할 필드명을 명시할 수 있습니다.

#### example

```bash
$ curl "localhost:9200/sales-records/_search?pretty" -H "Content-Type:application/json" -d '
{
  "query": {
    "query_string": {
      "query": "south AND asia",
      "fields": ["country", "message"]
    }
  }
}'
```

> 참고사이트
>
> 1. https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html
