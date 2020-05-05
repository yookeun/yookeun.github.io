---
layout: post
title: Elasticsearch에서 매핑(mapping) 하기
date: 2018-03-09
categories: elasticsearch
---

Elastic에서 index를 생성할 때 가장 중요한 것인 바로 매핑이 아닌가 싶다. 매핑은 RDBMS에서는 스키마같은 것이다.

엘라스틱에서 처음 인덱스를 생성하면 자동으로 매핑이 만들어진다. 하지만 실제로 그렇게 사용하는 경우는 드물다. 키바나와 효과적으로 연동하기 위해서 효율적인 매핑작업이  반드시 필요하다. 본 문서는 Elasticsearch 5.5를 기준으로 설명한다. 

### 1. Mapping Best Practices

엘라스틱에서 권장하는 방침은 아래와 같다.

- 직접 정의된 매핑을 사용하라 : 자동 매핑은 적절하지 않은 매핑을 만들어낸다. 

- Index Template 을 사용하라 : Index를 추가할 때 기본적으로 적용될 템플릿을 만들어라.

- 모든 신규 템플릿에 적용될 디폴트 템플릿을 만들어라 : 즉 글로벌 세팅을 통해 신규로 추가되는 템플릿에 반드시 적용되도록 하라. 

- 커스텀 anlyzer를 만들어 사용하라

- 싱슬 인덱스에 여러개의 타입(type)를 만들지 마라. 


### 2. Mapping 만들기

우리가 직접 매핑을 만드는 방법은 처음부터 바로 만드는 방법이 있고 일단 임시 인덱스를 만들고 그 자동 생성된 매핑을 가져와서 우리가 원하는 매핑을 만드는 방법이 있다. 후자는 매핑명령어가 익숙지 않은 경우에 사용하면 좋다.

일단 아래 필드로 구성된 인덱스를 만든다고 가정하자.

| 필드값   | 설명   |
| -------- | ------ |
| name     | 이름   |
| age      | 나이   |
| gender   | 성별   |
| memo     | 메모   |
| reg_date | 등록일 |

그러면 임시로 아래와 같이 임시 인텍스를 만들고 위에 필드를 적용한 인덱스를 만든다.

```json
PUT imsi_my_test/person/1 
{
  "name": "kim",
  "age": 40,
  "gender": "male",
  "memo": "This is good man!",
  "reg_date": "2018-03-01"
}
```

성공적으로 잘 만들졌으면 자동으로 생성된 매핑을 가져오자. `GET imsi_my_test/_mapping` 

```json
{
  "imsi_my_test": {
    "mappings": {
      "person": {
        "properties": {
          "age": {
            "type": "long"
          },
          "gender": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "memo": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "reg_date": {
            "type": "date"
          }
        }
      }
    }
  }
}
```

이제 위에 정보를 이용하여 매핑을 만들고 그것을 우리가 만들 인덱스에 생성해주면 된다. 위에서 보면 name, gender등이 `text` 으로 되어 있는데 이름하고 성별은 `keyword` 으로 하는 것이 좋다. 그래야 키바나를 통해 쉽게 분류할 수 있기 때문이다. 그리고 age도 `integer`로 하고 나머지 정보는 그대로 따르면 되겠다. 

```json
PUT my_test/
{
    "mappings": {
      "person": {
        "properties": {
          "age": {
            "type": "integer"
          },
          "gender": {
            "type": "keyword"
          },
          "memo": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "name": {
            "type": "keyword"
          },
          "reg_date": {
            "type": "date"
          }
        }
      }
    }
}
```

우리가 사용할 `my_idex`를  만들고 `mappings` 를 통해 `person` 이라는 `type` 를 만들고 각각의 `properties`를 만들면 된다.

`GET my_test/_mapping` 를 확인하면 제대로 매핑됨을 확인할 수 있다. 

>  Mapping 을 수정하고 싶은가? 

엘라스틱 서치에서는 이미 생성된 매핑을 수정할 수 없다.  데이터의 유무에 상관없이 매핑수정은 불가능하다. 그 이유는 만약 데이터가 있다면 수정시 타입이 변경이 안되기 때문이다. 하지만 몇가지 조건하(dynamic=true(default))에 필드 추가는 가능하다고 한다.  

```json
PUT my_test/_mapping/person
{
  "properties": {
    "location" : {
      "type": "geo_point"
    }
  }
}
```

위와 같이 location이라는 필드를 매핑에 추가할 수 있다(데이터 이미 존재해도..)

### 3. Index Template 만들기

인덱스 템플릿은 인덱스를 새로 생성할때마다 미리 작성된 매핑으로 적용하는 것이다. 기존에 공통적인 매핑정보를 만들고 추가되는 인덱스에서 별도로 매핑을 만들경우 아래와 같이 구성된다.

> 공통된 매핑 + 추가된 매핑 = 새 인덱스 

Index Template은 여러개를 만들수 있고 우선순위(order)를 부여함으로써 순차적으로 적용할 수 있다. 모든 인덱스에 해당되도록 하는 가장 최상위 기본 템플릿을 만들어보자. 

```json
PUT _template/default_my_template
{
  "template": "*",
  "order": 1,
  "mappings": {
    "my_type" :{
      "properties": {
        "reg_date": {
          "type": "date"
        }
      }
    }
  }
}
```

`template:*` 를 주목하라. 모든 템플릿에 적용한다는 것이다. order=1은 가장 최상위라는 의미다. 매핑정보를 보면 my_type이라는 type를 만들때 프로퍼티가 reg_date는 date type으로 한다는 것이다. 따라서 이 후에 생성되는 모든 매핑에는 reg_date는 기본으로 date가 된다는 의미이다. 

다른 인덱스를 추가해보자. 

```json

PUT my_test1/
{
    "mappings": {
      "person": {
        "properties": {
          "age": {
            "type": "integer"
          },
          "gender": {
            "type": "keyword"
          },
          "memo": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "name": {
            "type": "keyword"
          }
        }
      }
    }
}
```

reg_date라는 필드는 없지만 새로 값을 넣으면 date 타입으로 만들어진다. 만약 새로 추가된 인덱스에서 reg_date를 keyword로 등록하려면 바로 에러가 나서 등록이 되지 않는다. 물론 추가되는 필드매핑은 그대로 적용된다.  따라서 매우 유용하게 사용할 수 있다. 

`"template": "*",` 에 `template": "my_*` 식으로 해서 조건을 지정해서 적용할 수 있다. 엘라스틱서치에서는 적극적으로 Index Template를 사용하길 권장한다. 

등록된 템플릿을 모두 보거나 삭제하는 것은 아래처럼 하면 된다

``` json
GET _template

DELETE _template/my_template
```





