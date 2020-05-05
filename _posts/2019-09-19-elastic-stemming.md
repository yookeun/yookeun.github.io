---
layout: post
title: "Elasticsearch Lovins, Porter, Porter2 비교"
date: 2019-09-19
categories: elasticsearch
---

Elasticsearch에서 stemming Algorithms 에 대해 비교분석해보자.  보통 Lovins Stemmer와 Porter Stemmer로 나누어지는데 장단점은 아래와 같다. 

출처) <https://pdfs.semanticscholar.org/1c0c/0fa35d4ff8a2f925eb955e48d655494bd167.pdf>

위 출처에서 장단점 표를 그대로 옮기면 다음과 같다. 

### Lovins Stemmer 의 장단점

|Advantage|Limitations|
|------------------------------------------------------------ | ------------------------------------------------------------ |
| Fast – single pass algorithm.| Time consuming.|
| Handles removal of double letters in words like ‘getting’ being transformed to ‘get’. | Not all suffixes available.|
| Handles many irregular plurals like – mouse and mice etc.| Not very reliable and frequently fails to form words from the stems. |
|                                                              | Dependent on the technical vocabulary being used by the author. |

### Porter Stemmer 의 장단점

| Advantage | Limitations|
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Produces the best output as compared to otherstemmers.       | The stems produced are not always real words.                |
| Less error rate.| It has at least five steps and sixty rules and hence is time consuming.|
| Compared to Lovins it’s a light stemmer.                     |                                                              |
| The Snowball stemmer framework designed by Porter is language independent approach to stemming.|                            |

### Porter2 Stemmer는 Porter 업그레이드 버전이다.

> Porter: Most commonly used stemmer without a doubt, also one of the most gentle stemmers. One of the few stemmers that actually has Java support which is a plus, though it is also the most computationally intensive of the algorithms(Granted not by a very significant margin). It is also the oldest stemming algorithm by a large margin.

> Porter2: Nearly universally regarded as an improvement over porter, and for good reason. Porter himself in fact admits that it is better than his original algorithm. Slightly faster computation time than porter, with a fairly large community around it.

출처) <https://stackoverflow.com/questions/10554052/what-are-the-major-differences-and-benefits-of-porter-and-lancaster-stemming-alg>

Elasticsearch에서는 filter에서  Lovins, Porter, Porter2 를 모두 지원한다. 해당 stemmer를 통해서 어간 분석이 각각 어떻게 다른지 테스트를 해보자. 

먼저 아래와 같은 인덱스를 생성한다. 해당 인덱스에는 lovins, porter, porter2가 filter로 등록된 analyzer가 있다. 이것을 가지고 실제 분석을 해본다. 

### Stemmer Test Index 생성

```json 
PUT stemmer_test
{
    "settings": {
        "analysis": {
            "analyzer": {
                "lovins_analyzer": {
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "lovins"
                    ]
                },
                "porter_analyzer": {
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "porter"
                    ]
                },
                "porter2_analyzer": {
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "porter2"
                    ]
                }
            },
            "filter": {
                "lovins": {
                    "type": "stemmer",
                    "name": "lovins"
                },
                "porter": {
                    "type": "stemmer",
                    "name": "porter"
                },
                "porter2": {
                    "type": "stemmer",
                    "name": "porter2"
                }
            }
        }
    }
}
```

### Lovins 테스트

```json 
POST stemmer_test/_analyze
{
	"analyzer": "lovins_analyzer",
	"text": "dying"
}
```

#### 결과 

```json 
{
    "tokens": [
        {
            "token": "dying",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
        }
    ]
}
```

Lovins  사용시 검색어와 동일한 `dying` 으로  토큰 처리되어 전혀 어간으로 분류되지 않는다. 

### Porter 테스트 

```json 
POST stemmer_test/_analyze
{
	"analyzer": "porter_analyzer",
	"text": "dying"
}
```

#### 결과 

```json 
{
    "tokens": [
        {
            "token": "dy",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
        }
    ]
}
```

Porter 사용시 `dy` 처럼 전혀 엉뚱한 어간으로 토큰처리된다. 

### Porter2 테스트 

```json 
POST stemmer_test/_analyze
{
	"analyzer": "porter2_analyzer",
	"text": "dying"
}
```

#### 결과 

```json 
{
    "tokens": [
        {
            "token": "die",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
        }
    ]
}
```

Porter2에서는 `die`  라는 제대로 된 어간으로 토큰 처리되어 진다.  