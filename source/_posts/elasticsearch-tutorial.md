---
title: elasticsearch 튜토리얼
date: 2019-12-24 15:09:45
categories:
- elasticsearch
tags:
- elasticsearch
- logstash
- kibana
---

elasticsearch란 아파치 Lucene 기반으로 개발한 오픈소스 검색엔진으로 많은 양의 데이터를 보관하고 실시간으로 분석할 수 있게 해준다. JSON 기반의 비정형 데이터 분산 검색과 분석을 지원하며, 다양한 기능을 플러그인 형태로 구현하여 적용할 수 있는 특징을 가진다. 본 문서에서는 설치 및 사용법(spring boot 연동)을 다룬다.
<!--more-->  

## **Table of Contents**

- [elasticsearch 란](#elasticsearch-란)
- [Installation](#Installation)
- [기본 사용법](#기본-사용법)
- [Spring boot 연동](#Spring-boot-연동)
- [Reference](#Reference)
- [Contributors](#Contributors)


## Elasticsearch 란

Elasticsearch는 모든 유형의 데이터에 대한 실시간 검색 및 분석을 제공하는 오픈소스 검색 엔진으로 구조화되거나 구조화되지 않은 텍스트, 숫자 데이터 또는 지리 공간 데이터에 관계없이 Elasticsearch는 빠른 검색을 지원하는 방식으로 효율적으로 저장하고 색인을 생성 할 수 있다. 

단순한 데이터 검색을 넘어 정보를 집계하여 데이터의 추세와 패턴을 발견 할 수 있고 데이터 및 쿼리 볼륨이 증가함에 따라 Elasticsearch의 분산 특성으로 인해 배포가 원활하게 확장 될 수 있습니다.

Elasticsearch는 Elastic Stack의 중심에있는 분산 검색 및 분석 엔진이다. Logstash and Beats를 사용하면 데이터를 수집, 집계 및 보강하고 Elasticsearch에 저장할 수 있고, Kibana를 사용하면 대화식으로 데이터를 탐색, 시각화, 공유, 데이터를 관리 등이 가능하다.



##### elasticsearch와 관계형 DB 비교

| Elasticsearch | 관계형 DB |
| ------------- | --------- |
| Index         | Database  |
| Type          | Table     |
| Document      | Row       |
| Field         | Column    |
| Mapping       | Schema    |

| Elasticsearch | 관계형 DB | CRUD   |
| ------------- | --------- | ------ |
| POST          | INSERT    | CREATE |
| GET           | SELECT    | READ   |
| PUT           | UPDATE    | UPDATE |
| DELETE        | DELETE    | DELETE |



## Installation

##### 튜토리얼 환경

ubuntu 18.04,  openjdk 11.0.5



##### Elasticsearch 설치

```bash
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.1-amd64.deb
$ dpkg -i elasticsearch-7.5.1-amd64.deb
$ sudo systemctl enable elasticsearch.service
```



##### Elasticsearch 실행 및 동작 테스트

```bash
$ sudo service elasticsearch start   
$ curl -X GET 'localhost:9200'
```



##### Elasticsearch 종료

```bash
$ sudo service elasticsearch stop
```



## 기본 사용법

Elasticsearch는 REST API를 통해 CRUD가 가능. URL의 계층에 따라 /index/type/document/field로 구분하여 사용.

##### Index CRD

```bash
$ curl -X PUT http://localhost:9200/'class name'
$ curl -X GET http://localhost:9200/'class name'?pretty
$ curl -X DELETE http://localhost:9200/'class name'
```



##### Document CRUD

document는 index가 있을때 만들어도 되고, index가 없을때도 index명과 type명을 명시해주면 바로 document 생성이 가능.

```bash
$ curl -X POST http://localhost:9200/'class name'/'type name'/'document name' -H 'Content-type:application/json' -d '{"title":"algorithm","professor":"john"}'
$ curl -X POST http://localhost:9200/'class name'/'type name'/'document name' -H 'Content-type:application/json' -d @data.json

$ curl -X GET http://localhost:9200/'class name'/'type name'/'document name'

$ curl -X DELETE http://localhost:9200/'class name'/'type name'/'document name'
```

필드에 대한 업데이트 방법은 다음과 같다.

```bash
$ curl -X POST http://localhost:9200/'class name'/'type name'/'document name'/_update -H 'Content-type:application/json' -d '{"doc":{"unit":1}}'
$ curl -X POST http://localhost:9200/'class name'/'type name'/'document name'/_update -H 'Content-type:application/json' -d '{"script":"ctx._source.unit += 5"}'
```

데이터를 bulk로 저장하려면 json 파일에 index, type, id를 지정하여 데이터를 입력하여 저장한 후 다음 명령어를 통해 저장.

```json
bulk.json

{"index":{ "_index":"classes","_type":"class","_id":"1"}}
{"title":"algorithm","Professor":"john"}
```

```bash
curl -X POST http://localhost:9200/_bulk --data-binary -H 'Content-type:application/json' -d @bulk.json
```



##### Mapping

매핑없이 elastic search에 데이터를 넣을 수 있지만 매우 위험하다. 이유는 타입형에서도 혼선이 올 수 있으며,  Kibana로 시각화 할때 적절한 출력이 안될 수 있는 등 다양한 위험 요소가 존재 힌다. 따라서 데이터를 입력하기 전에 Mapping을 통해 자료형을 저장한 후 데이터를 입력.

데이터 관리시에는 매핑을 먼저 추가하고, 데이터가 이미 있을때는 매핑을 추후에 추가하여, 분석이나 시각화할때 도움이 될 수 있음.

Mapping 방법은 다음과 같다.

```json
mapping.json

{
	"class" : {
		"properties" : {
			"title" : {
				"type" : "keyword"
			},
			"professor" : {
				"type" : "keyword"
			}
		}
	}
}
```

```bash
$ curl -X PUT http://localhost:9200/'class name'/'type name'/_mapping?include_type_name=true&pretty -H 'Content-type:application/json' -d @mapping.json
```

include_type_name=true가 없으면 illegal_argument_exception 에러가 발생하여 추가.



##### Search

전체 조회는 검색조건 없이 조회하면 되고, 검색 조건을 통한 방법은 url에서 query 파라미터를 추가 하는 방법과 request body를 추가하는 방법 존재.

```bash
$ curl -X GET http://localhost:9200/'class name'/'type name'/_search
$ curl -X GET http://localhost:9200/'class name'/'type name'/_search?&q=points:30 
$ curl -X GET http://localhost:9200/'class name'/'type name'/_search -d '{"query":{"term":{"points":30}}}' -H 'Content-type:application/json'
```

request body에는 여러가지 옵션이 존재함. 자세히 살펴보려면 [문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)를 확인.



##### Aggregation

 document 안에서 조합을 통해서 어떠한 값을 도출할때 쓰이는 방법을 Aggregation 이라한다. aggregation의 종류는 Bucketing, Metric, Matrix, Pipeline이 있으며, 구조는 다음과 같다.

```bash
"aggregations" : {
	"<aggregation_name>" : {
		"<aggregation_type>" : {
		}
		[,"meta":{[<meta_data_body>]}]?
		[,"aggregations":{[<sub_aggregation>]+}]}]?
	}
	[, "<aggregation_name_2>":{...}]*
}

```

- Bucketing

  버킷을 작성하는 Aggregation으로, 각 버킷은 키 및 문서 기준과 연결.  Aggregation이 실행될 때 모든 버킷 기준이 컨텍스트의 모든 document에서 평가되고 기준이 일치하면 문서가 관련 버킷에 "fall in"한 것으로 간주. Aggregation  프로세스가 끝날 때마다 버킷 목록이 생김. 각 버킷에는 "belong"된 문서 세트 존재.

- Metric

  일련의 문서에서 메트릭을 추적하고 계산하는 Aggregation.

- Matrix

  여러 필드에서 작동하고 요청 된 문서 필드에서 추출 된 값을 기반으로 매트릭스 결과를 생성하는 Aggregation.

- Pipeline

  다른 집계 및 관련 메트릭의 결과를 집계하는 Aggregation 

  

하위 예제에서 사용할 document

```bash
 basketball_mapping.json 
 
{
	"record" : {
		"properties" : {
			"team" : {
				"type" : "text",
				"fielddata" : true
			},
			"name" : {
				"type" : "text",
				"fielddata" : true
			},
			"points" : {
				"type" : "long"
			},
			"rebounds" : {
				"type" : "long"
			},
			"assists" : {
				"type" : "long"
			},
			"blocks" : {
				"type" : "long"
			},
			"submit_date" : {
				"type" : "date",
				"format" : "yyyy-MM-dd"
			}
		}
	}
}

```

```bash
twoteam_basketball.json  

 "index" : { "_index" : "basketball", "_type" : "record", "_id" : "1" } }
{"team" : "Chicago","name" : "Michael Jordan", "points" : 30,"rebounds" : 3,"assists" : 4, "blocks" : 3, "submit_date" : "1996-10-11"}
{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "2" } }
{"team" : "Chicago","name" : "Michael Jordan","points" : 20,"rebounds" : 5,"assists" : 8, "blocks" : 4, "submit_date" : "1996-10-13"}
{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "3" } }
{"team" : "LA","name" : "Kobe Bryant","points" : 30,"rebounds" : 2,"assists" : 8, "blocks" : 5, "submit_date" : "2014-10-13"}
{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "4" } }
{"team" : "LA","name" : "Kobe Bryant","points" : 40,"rebounds" : 4,"assists" : 8, "blocks" : 6, "submit_date" : "2014-11-13"}

```



##### Metric Aggregation

 document 안에서 조합을 통해서 어떠한 값을 도출할때 쓰이는 방법을 Aggregation 이라하고 그 중 최댓값, 최솟값, 평균값 등 산술관련한 연산이 필요할 경우 metric aggregations을 사용한다. 간단한 예로 최대, 최소, 평균값을 구하면 다음과 같다.

```bash
avg_metric.json

{
	"size" : 0,
	"aggs" : {"avg_score" : { "avg" : { "field" : "points"}}}
}
```

```bash
$ curl -X GET http://localhost:9200/_search\?pretty -H 'content-type:application/json' --data-binary @avg_metric.json
```

```bash
min_metric.json

{
	"size" : 0,
	"aggs" : {"avg_score" : { "min" : { "field" : "points"}}}
}
```

```bash
$ curl -X GET http://localhost:9200/_search\?pretty -H 'content-type:application/json' --data-binary @avg_metric.json
```

```bash
max_metric.json

{
	"size" : 0,
	"aggs" : {"avg_score" : { "max" : { "field" : "points"}}}
}
```

```bash
$ curl -X GET http://localhost:9200/_search\?pretty -H 'content-type:application/json' --data-binary @maxcat st	_metric.json
```

```bash
stats_metric.json

{
	"size" : 0,
	"aggs" : {"avg_score" : { "stats" : { "field" : "points"}}}
}
```

```bash
$ curl -X GET http://localhost:9200/_search\?pretty -H 'content-type:application/json' --data-binary @stats_metric.json
```

metrics aggregation의 옵션들을 자세히 살펴보려면 [문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html)를 확인.



##### Bucket Aggregation

bucket aggregation은 group by의 개념과 흡사하다. 각 필드에서 값을 계산하지는 않지만 각 bucket은 키 및 document criterion과 연결하여 document bucket을 생성한다.

```bash
terms_aggs.json 

{
	"size" : 0,
	"aggs" : {"players" : {"terms" : {"field" : "team"}}}
}
```

```bash
$ curl -XGET http://localhost:9200/_search\?pretty -H 'content-type:application/json' --data-binary @terms_aggs.json
```

bucket과 metric을 조합하면 특정 필드별 계산된 값을 얻을 수 있다.

```bash
stats_by_team.json

{
	"size" : 0,
	"aggs" : {
		"team_stats" : {
		    "terms" : {"field" : "team"},
			"aggs" : {
			    "stats_score" : {"stats" : {"field" : "points"}}
			}
		}
	}
}
```

```bash
$ curl -XGET http://localhost:9200/_search\?pretty -H 'content-type:application/json' --data-binary @stats_by_team.json 
```

bucket aggregation의 옵션들을 자세히 살펴보려면 [문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html)를 확인.





## Spring boot 연동

블라블라



## Reference

- [Elastic Stack and Product Documentation](https://www.elastic.co/guide/index.html)
- https://github.com/minsuk-heo/BigData

## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)