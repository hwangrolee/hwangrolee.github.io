---
layout: post
title: logstash를 사용해서 CSV파일을 elastic search에 임포트하기
date: 2019-05-01
description: logstash를 사용해서 CSV파일을 elastic search에 임포트하기
categories: [Database]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /logstash를-사용해서-CSV파일을-elastic-search에-임포트하기/
author: hwangrolee
tags: [Logstash]
---

안녕하세요. Elastic Stack을 공부하고 있는 서버개발자입니다.

Elastic search를 공부하기에 앞서 대용량의 데이터가 elastic search에 저장되길 원했습니다.

일반적으로 csv 파일로 데이터 임포트를 많이 진행하기 때문에 저도 csv파일로 데이터를 미리 찾았습니다.

http://eforexcel.com/wp/downloads-18-sample-csv-files-data-sets-for-testing-sales/

위 사이트에는 판매내역 데이터이며 100개 ~ 150만개의 데이터를 다운받을 수 있습니다.

#### 도커로 Elasticsearch 7 설치하고 컨테이너 생성하기

```bash
# -p 9200:9200 포트 9200를 포트포워딩한다
# -p 9300:9300 포트 9200를 포트포워딩한다
# -e "discovery.type=single-node" 엘라스틱서치를 싱글노드로 설정한다.
# -d daemon형태로 동작한다(background)
# --name 컨테이너의 이름을 "es-sales-records" 로 지정한다
$ docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d --name es-sales-records docker.elastic.co/elasticsearch/elasticsearch:7.0.0 -Des.http.cors.enabled=true
670aae682079347cbf7ec2235436d9b0a85d6bdfcb19882513b03029189811f9
```

#### 생성된 컨테이너 확인하기

```
$ docker ps -a
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
670aae682079        docker.elastic.co/elasticsearch/elasticsearch:7.0.0   "/usr/local/bin/dock…"   10 seconds ago      Up 9 seconds        0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es-sales-records
```

#### 엘라스틱서치 확인하기

```bash
$ curl localhost:9200
{
  "name" : "670aae682079",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "pRxlVMULTZ6Ixooavg7Yng",
  "version" : {
    "number" : "7.0.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "b7e28a7",
    "build_date" : "2019-04-05T22:55:32.697037Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.7.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

#### 테스트할 데이터를 다운받아볼게요.

```bash
$ wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.zip
$ unzip 1500000\ Sales\ Records.zip
```

다운로드 하셨나요? csv파일에는 약 150만개의 데이터가 들어있습니다.

#### logstash를 통해 elastic search에 import할수있도록 conf 파일을 작성합니다.

```bash
# 개인적으로 vi를 많이 사용합니다.
# vi가 불편하다면 nano를 사용하세요.
$ vi sales-records.conf
```

```vim
input {
	file {
		path => "/config-dir/1500000 Sales Records.csv"
		start_position => "beginning"
	}
}

filter {
	csv {
		separator => ","
		columns => ["Region", "Country", "Item Type", "Sales Channel", "Order Priority", "Order Date", "Order ID", "Ship Date", "Units Sold", "Unit Price", "Unit Cost", "Total Revenue", "Total Cost", "Total Profit"]
	}

	date { match => ["Order Date", "dd/MM/yyyy"]}
	date { match => ["Ship Date", "dd/MM/yyyy"] }

	mutate { convert => ["Region", "string"] }
	mutate { rename => ["Region", "region"] }

	mutate { convert => ["Country", "string"] }
	mutate { rename => ["Country", "country"] }

	mutate { convert => ["Item Type", "string"] }
	mutate { rename => ["Item Type", "item_type"] }

	mutate { convert => ["Sales Channel", "string"] }
	mutate { rename => ["Sales Channel", "sales_channel"] }

	mutate { convert => ["Order Priority", "string"] }
	mutate { rename => ["Order Priority", "order_priority"] }

	mutate { rename => ["Order Date", "order_date"] }

	mutate { convert => ["Order ID", "integer"] }
	mutate { rename => ["Order ID", "order_id"] }

	mutate { rename => ["Ship Date", "ship_date"] }

	mutate { convert => ["Units Sold", "integer"] }
	mutate { rename => ["Units Sold", "units_sold"] }

	mutate { convert => ["Unit Price", "float"]  }
	mutate { rename => ["Unit Price", "unit_price"] }

	mutate { convert => ["Unit Cost", "float"]  }
	mutate { rename => ["Unit Cost", "unit_cost"] }

	mutate { convert => ["Total Revenue", "float"] }
	mutate { rename => ["Total Revenue", "total_revenue"] }

	mutate { convert => ["Total Cost", "float"] }
	mutate { rename => ["Total Cost", "total_cost"] }

	mutate { convert => ["Total Profit", "float"] }
	mutate { rename => ["Total Profit", "total_profit"] }

}

output {
	elasticsearch {
		action => "index"
		hosts => ["172.17.0.2:9200"]
		index => "sales-records"
	}
	stdout { codec => rubydebug{} }
}
```

#### 임포트 작업을 하기전에 index를 생성해야합니다.

```bash
$ curl -XPUT localhost:9200/sales-records
```

#### 그럼 이제 logstash를 통해 임포트를 진행하겠습니다.

```bash
$ docker run --rm -it -e "xpack.monitoring.elasticsearch.hosts=172.17.0.2:9200" -v "$PWD":/config-dir docker.elastic.co/logstash/logstash:7.0.0 -f /config-dir/sales-records.conf
```

임포트 하는 시간이 꽤 걸릴 것입니다.

#### 간단하게 2개의 데이터만 보도록하겠습니다.

```bash
$ curl -XGET "localhost:9200/sales-records/_search?pretty" -d '{ "size": 2 }' -H "Content-Type: application/json"
{
  "took" : 2153,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "sales-records",
        "_type" : "_doc",
        "_id" : "12hYYGoBml718jk9b2If",
        "_score" : 1.0,
        "_source" : {
          "total_cost" : 3870530,
          "order_date" : "4/20/2015",
          "order_id" : 530370843,
          "country" : "Romania",
          "@version" : "1",
          "@timestamp" : "2019-04-27T19:49:26.962Z",
          "order_priority" : "L",
          "units_sold" : 7373,
          "unit_cost" : 524,
          "total_revenue" : 4801371,
          "path" : "/config-dir/1500000 Sales Records.csv",
          "sales_channel" : "Online",
          "item_type" : "Office Supplies",
          "host" : "fb86e440330a",
          "message" : "Europe,Romania,Office Supplies,Online,L,4/20/2015,530370843,5/26/2015,7373,651.21,524.96,4801371.33,3870530.08,930841.25\r",
          "tags" : [
            "_dateparsefailure"
          ],
          "unit_price" : 651,
          "region" : "Europe",
          "total_profit" : 930841,
          "ship_date" : "5/26/2015"
        }
      },
      {
        "_index" : "sales-records",
        "_type" : "_doc",
        "_id" : "pmhYYGoBml718jk9b2IV",
        "_score" : 1.0,
        "_source" : {
          "total_cost" : 881752,
          "order_date" : "5/11/2016",
          "order_id" : 839094388,
          "country" : "Tonga",
          "@version" : "1",
          "@timestamp" : "2016-11-05T00:00:00.000Z",
          "order_priority" : "L",
          "units_sold" : 5531,
          "unit_cost" : 159,
          "total_revenue" : 1411953,
          "path" : "/config-dir/1500000 Sales Records.csv",
          "sales_channel" : "Online",
          "item_type" : "Baby Food",
          "host" : "fb86e440330a",
          "message" : "Australia and Oceania,Tonga,Baby Food,Online,L,5/11/2016,839094388,5/31/2016,5531,255.28,159.42,1411953.68,881752.02,530201.66\r",
          "tags" : [
            "_dateparsefailure"
          ],
          "unit_price" : 255,
          "region" : "Australia and Oceania",
          "total_profit" : 530201,
          "ship_date" : "5/31/2016"
        }
      }
    ]
  }
}
```

#### 매핑정보를 확인해서 각 필드가 어떤 유형인지 확인해볼게요

```bash
$ curl -XGET "localhost:9200/sales-records/_mappings?pretty"
{
  "sales-records" : {
    "mappings" : {
      "properties" : {
        "@timestamp" : {
          "type" : "date"
        },
        "@version" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "country" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "host" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "item_type" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "message" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "order_date" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "order_id" : {
          "type" : "long"
        },
        "order_priority" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "path" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "region" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "sales_channel" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "ship_date" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "tags" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "total_cost" : {
          "type" : "float"
        },
        "total_profit" : {
          "type" : "float"
        },
        "total_revenue" : {
          "type" : "float"
        },
        "unit_cost" : {
          "type" : "float"
        },
        "unit_price" : {
          "type" : "float"
        },
        "units_sold" : {
          "type" : "long"
        }
      }
    }
  }
}
```
