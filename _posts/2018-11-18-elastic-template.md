---
layout: post
title:  "Elasticsearch에서 Template 사용하기"
date:   2018-11-18
categories: elasticsearch
---

엘라스틱서치에서 우리는 인덱스에 대한 매핑을 작성하고 데이터를 수집한다. 그런데 만약 로그성 데이터를 받을 경우 예를 들어서 일자별이나 월별로 인덱스가 생성되는 경우는 어떻게 할까? 미리 미리 매핑을 만들 수는 없을까? 이럴때 바로 Template를 이용하는 것이다. 

예를 들어 다음과 같은 로그데이트를 수집한다고 가정하다.

- 회원로그인데이터 (member_login_2018-11-10)
- 회원구매데이터(member_buy_2018-11-10)

각각의 데이터를 일자별로 수집할 경우 해당 인덱스의 매핑정보를 미리 설정하는 방법이 바로 템플릿이다. 템플릿을 만들면 일자별, 월별로 인덱스를 만들때 정의된 매핑으로 구성할 수 있게 된다. 

### 1. 공통 템플릿

위에 같은 경우에 회원에 대한 공통정보가 있을 것이다. 아이디, 이름등이 이에 해당될 것이다. 이를 별도로 공통 상위템플릿을 만들자.

```json 
PUT /_template/member_basic_template
{
    "index_patterns": ["member*"],
    "order": 0,
    "settings": {
        "number_of_shards": 5,
        "number_of_replicas" : 0
    },
    "mappings": {
         "_doc": {
            "_source": {
                "enabled": false
            },
            "properties": {
                "userId": {
                    "type": "keyword"     
                },
                "userName": {
                    "type": "keyword"                    
                }
            }
        }
    }
}
```

가장 상위를 뜻하는 `order` 를 0으로 지정한다. 그리고 settings를 통해 샤드에 대한 정보를 세팅하자. 다음은 회원 로그인데이터에 대한 템플릿을 만들어보자

### 2. 개별 탬플릿 

```json 
PUT /_template/member_login_template
{
    "index_patterns": ["member_login*"],
    "orders": 1,
    "aliases": {
        "member_login": {}
    },
    "mappings": {
        "_doc": {
            "_source": {
                "enabled": true
            },
            "properties": {
                "loginTime": {
                    "type": "date"
                },
                "loginMenue": {
                    "type": "keyword"
                }
            }
        }
    }
}
```

위에서 같이 `order` 부분을 1로 지정하면 member_login_2018-11-18.log 가 발생할 경우 member로 시작되니까 0으로 세팅되고 `member_login*` 으로 시작되니 1번이 적용된다.  공통되는 필드는 0으로 지정하고 나머지는 1로 지정해서 사용하면 편리하게 세팅할 수 있다. 

다만 orders가 0이 아닌 템플릿에서는 `aliases` 를 지정해야 `인덱스명_yyyy-mm-dd`식으로 생성된 모든 인덱스를 하나의 얼라이스명으로 묶을수 있다. 

### 3. 로그스테시 설정

로그스테시에서 로그를 생성할 경우 인덱스_날짜별로 세팅해주어야 한다. 

```ruby 
output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        user => "elastic"
        password => "1234"
        index => "member_login-%{+YYYY-MM-dd}"
        document_type => "_doc"
    }
}
```

이렇게 하면 일자별, 월별로 생성되는 로그데이터의 인덱스 매핑을 템플릿으로 적용할 수 있게 된다. 