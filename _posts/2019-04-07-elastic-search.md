---
layout: post
title:  "Elasticsearch Search APIs"
date:   2019-04-07
categories: elasticsearch
---

### Routing

엘라스틱에서 서치를 할때는 보통  여러개의 사드에서 검색이 이루어진다. 때문에 데이터가 많아지거나 샤드가 많을 수록 다소 지연될 수 있다. 이럴 때 Routing을 사용하면 특정 사드에 인덱싱이 되기 때문에 검색시 특정 샤드에서 검색만 되기 때문에 보다 빠르게 검색이 될 수 있다. 

``` json 
POST /test/_doc?routing=kim
{
    "user" : "kim1",
    "message" : "test kim1"
}

POST /test/_doc
{
    "user" : "kim2",
    "message" : "test kim2"
}
```

`kim1` 에는 `routing=kim`을 넣었고 `kim2` 에는 일반적인 방법으로 등록했다. 검색을 수행해보자.

``` json 
GET /test/_search
{
  "query": {
    "term": {
      "user": "kim1"
    }
  }
}
```

routing없이 검색을 하면 검색결과는 아래와 같다. 

``` json 
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "3nMC92kBVE5-ScJ0hJ7_",
        "_score" : 0.2876821,
        "_routing" : "kim",
        "_source" : {
          "user" : "kim1",
          "message" : "test kim1"
        }
      }
    ]
  }
}
```

_shards.total 부분을 보면 총 5개의 사드에서 검색됨을 알 수 있다. 이번에는 라우팅을 통해 검색해보자. 

``` json 
GET /test/_search?routing=kim
{
  "query": {
    "term": {
      "user": "kim1"
    }
  }
}
```

``` json 
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "3nMC92kBVE5-ScJ0hJ7_",
        "_score" : 0.2876821,
        "_routing" : "kim",
        "_source" : {
          "user" : "kim1",
          "message" : "test kim1"
        }
      }
    ]
  }
}
```

_shards에 total를 보면 하나의 샤드에서 검색이 이루어진다. 이처럼 routing을 통해서 자주 검색되는 것을 하나의 샤드에 넣어서 관리할 수 있다. 

### URI Search & Body Search

user 값이 kim5 인 것 검색하기 

``` json 
#한 개의 인덱스에서 검색
GET /test/_search?q=user:kim5
#여러 개의 인덱스에서 검색
GET /test,test2,test3/_search?q=user:kim5
#전체 인덱스에서 검색
GET /_all/_search?q=user:kim5
```

``` json 
GET /test/_search
{
  "query": {
    "term": {
      "user":"kim5"
    }
  }
}
```

### Script Fields 

아래와 같은 책에 대한 인덱스가 있다고 여기자, 그런데 price 에 세금을 포함한 가격으로 출력하고 싶을 때가 있다. 이렇게 특정피드에 어떤 스크립트 로직이 필요한 경우에 `script fields` 를 사용하면 된다. 

``` json 
POST /books/_doc
{
  "type": "novel",
  "title": "book1",
  "author" : "kim1",
  "price": 1000,
  "publisher": "YK"
}

POST /books/_doc
{
  "type": "novel",
  "title": "book2",
  "author" : "kim2",
  "price": 1100,
  "publisher": "YK"
}
```

``` json 
GET /books/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value + (doc['price'].value * params.tax)",
        "params": {
          "tax": 0.3
        }
      }
    }
  }
}
```

### Search Template

``` json 
GET /products/_search/template
{
  "source": {
    "query": {
      "match": {
        "{{my_field}}": "{{my_value}}"
      }
    },
    "size": "{{my_size}}"
  },
  "params": {
    "my_field":"name",
    "my_value":"Scampi",
    "my_size": 10
  }
}
```

Mustache templating 를 사용할 수 있다. 자주 사용되는 검색조건을 만든후에 파라미터만 변경되는 경우 사용하면 유용하다. 

### Suggester

``` json 
POST /products/_search
{
 "query" : {
    "match": {
      "name": "tring out Elasticsearch"
    }
  },
  "suggest" : {
    "my-suggestion" : {
      "text" : "Tail",
      "term" : {
        "field" : "name"
      }
    }
  }  
}
```

해당 검색어에 대한 유사어가 나타난다. 네이버나 구글검색을 떠오르면 된다. `Tail` 로 검색시 score 기준으로 표시해준다. 

``` json 
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 0,
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "my-suggestion" : [
      {
        "text" : "tail",
        "offset" : 0,
        "length" : 4,
        "options" : [
          {
            "text" : "trail",
            "score" : 0.75,
            "freq" : 2
          },
          {
            "text" : "thai",
            "score" : 0.5,
            "freq" : 2
          },
          {
            "text" : "table",
            "score" : 0.5,
            "freq" : 1
          },
          {
            "text" : "taro",
            "score" : 0.5,
            "freq" : 1
          },
          {
            "text" : "tart",
            "score" : 0.5,
            "freq" : 1
          }
        ]
      }
    ]
  }
}
```

