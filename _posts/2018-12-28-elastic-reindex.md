---
layout: post
title:  "Elasticsearch에서 Reindex 살펴보기"
date:   2018-12-28
categories: elasticsearch
---

Elasticsearch에서 reindex를 사용하기 위해서는 두가지 사항을 알고 있어야 한다.

1. _source 는 enabled 상태인 index만 reindex가 가능하다. 
2. reindex는 복사가 아니다. 즉 reindex를 하려는 index의 mapping정보도 똑같이 카피되는 것이 아니라. 데이터만 넣기 때문에 이때 디폴트 매핑으로 인덱스가 생성되기 때문에 반드시 reindex 할 곳에 mapping작업도 원본과  동일하게 해주어야 한다. 

문법은 아래와 같다. 

```json
POST _reindex
{
    "source": {
        "index": "가져올 인덱스명"
    },
    "dest": {
        "index": "저장할 인덱스명"
    }
}
```

remote 된 곳이 인덱스도 가져올 수 있다.

```json 
POST _reindex
{
    "source": {
        "remote": {
            "host": "http://remotehost:9200",
            "username": "user",
            "password": "pass"
        },
        "index": "old_index"
    },
    "dest": {
        "index": "new_index"
    }
}
```

원한다면 쿼리를 주어서 가져올 수도 있다. 

```json 
POST _reindex
{
    "source": {
        "remote": {
            "host": "https://remotehost:9200",
            "username": "user",
            "password": "pass"
        },
        "index": "old_index",
        "query": {
            "match": {
                "field_name": "field_value"
            }
        }
    },
    "dest": {
        "index": "new_index"
    }
}
```

`username`, `password` 는 옵션이다. 만약 https 설정이 안되어 있다면 필요없다. 그런데 원격서버에서 인덱스를 가져와서 reindex를 하려면 원격서버의 elasticsearch에 elasticsearch.yml 에 다음과 같이 접근가능한 elasticsearch 주소를 기재해야 한다. 

> ```yaml
> reindex.remote.whitelist: "otherhost:9200, another:9200, 127.0.10.*:9200, localhost:*"
> ```



reindex를 할 때 몇가지 재미있는(?) 옵션들이 있다. 바로 `version_type` 이다. 우리가 같은 index에 대한 reindex를 여러번 할때마다 document version 필드가 업데이트가 된다. 기존꺼를 지우고 다시 인서트된다는 의미이다. version_type=internal 일 경우이다. 

```json 
POST /_reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  }
}
```

그런데 만약 version_type='external' 로 처리한다면 아래와 같은 버전 충돌 메시지가 나타난다.

```json 
{
  "took": 2,
  "timed_out": false,
  "total": 8,
  "updated": 0,
  "created": 0,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 8,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": [
    {
      "index": "new_twitter",
      "type": "_doc",
      "id": "5",
      "cause": {
        "type": "version_conflict_engine_exception",
        "reason": "[_doc][5]: version conflict, current version [7] is higher or equal to the one provided [1]",
        "index_uuid": "D8jYKopoQjS4bVTZH7MqlQ",
        "shard": "1",
        "index": "new_twitter"
      },
      "status": 409
    },
    .....중략......
```

old_index에 document가 추가/수정되면  version_type='external' 처리하면 이미 등록된 id는 버전충돌이 되서 업데이트가 되지 않고 **새로 추가/수정된 document만 입력되는 것이다.** 만약 저 에러나는 메시지가 보고 싫고 그냥 수정이나 추가된 document만 제대로 반영되기만 하면 된다면 아래와 같이 옵션(conflicts)을 추가하면 된다. 

```json 
POST /_reindex
{
  "conflicts": "proceed", 
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  }
}
```

새로 document가 추가 된 후에 위에 reindex를 실행하면 아래와 같이 나온다.

```json 
{
  "took" : 9,
  "timed_out" : false,
  "total" : 9,
  "updated" : 0,
  "created" : 1,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 8,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

total: 9 이고, version_conflicts: 8개 났다는 메시지가 보인다. 

그 다음 재미있는 옵션은 `op_type` 이다. op_type의 값은 create 밖에 없다. 

```json 
POST /_reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
```

`op_type: create` 는 만약 reindex를 한 후에 다시 reindex를 할 경우에 원본 index에 이미 reindex된 document가 수정되어도 reidex에 생성된 index에서는 반영되지 않고 오직 새로 생성된(create) document 만 적용된다는 것이다. 이것도 역시 수행시 같은 document가 있다면 version conflict 에러가 나고 그 메시지를 피하기 위해서 `conflicts: proceed` 를 사용하면 된다. 

아래와 같이 reindex를 한 후에 원본에 doucment을 업데이트해보자. 

```json 
PUT /twitter/_doc/9
{
  "counter":6,
  "tag": "8899aaaa2222"
}
```

reindex된 new_twitter만들고 나서 원본에 기존 존재하던 id=9를 업데이트 한후에 다시 op_type=create를 주고 reindex하고 `GET /new_twitter/_doc/9`을 조회 하면 위에처럼 갱신되지 않고 과거에 데이터만 나온다

```json 
{
  "_index" : "new_twitter",
  "_type" : "_doc",
  "_id" : "9",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "counter" : 6,
    "tag" : "8899aaaa"
  }
}
```

**새로 추가된 document만  reindex시 반영된다.** 실제 운영시 version_type, op_type를 용도에 맞게 적절하게 사용하면 원하는 결과를 얻을 수 있을 것이다. 

기억해야할 마지막 부분은 reindex는 scroll batch로 돌아간다는 것이다. 즉 default size=1000 개로 나누어서 가져온다. 이 사이즈는 아래와 같이 조정이 가능하다.

```json 
POST /_reindex
{
  "source": {
    "index": "twitter",
    "size": 100
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

그외 자세한 옵션은 elasticsearch의 공식 레퍼런스를 이용하도록 하자. 

[Elasticsearch Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)