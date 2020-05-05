---
layout: post
title:  "Logstash에서 Kafka Input 사용"
date:   2019-02-17
categories: elasticsearch
---

Kafka INPUT를 사용하기 위해서는 플러그인을 먼저 인스톨 해야 한다. 

```
./bin/logstash-plugin install logstash-input-kafka
```

이제 conf 파일을 작성해보자.

```
input {
    kafka {
        bootstrap_servers =>  "192.168.56.4:9092,192.168.57.3:9092,192.168.58.3:9092"
        group_id => "logstash"
        topics => ["ykkim-topic"]
        consumer_threads => 1
    }
}

filter {
}

output {
    stdout {
        codec => rubydebug
    }    
    elasticsearch {
        hosts => "http://localhost:9200"
        index => "kafka-test-%{+YYYY-MM-dd}"
        document_type => "_doc"
    }
}
```

`bootstrap_servers` 에 카프카 서버리스트를 작성해주고 group_id를 설정해준다. 그리고 가져올 토픽을 지정하면 된다. consumer_threads는 사양에 맞게 적절하게 세팅하면 된다. 디폴트는 1이다. 

이제 로그스테스를 가동한 후에 카프카에서 producer를 이용해 메시지(Hello Logstash1~3)를 전송하고 이를 로그스테시에서 잘 받는지 확인해보자. 

```
>./bin/kafka-console-producer.sh --broker-list 192.168.56.4:9092,192.168.57.3:9092,192.168.58.3:9092 --topic ykkim-topic

>Hello Logstash1
>Hello Logstash2
>Hello Logstash3
```

총 3번의 메시지를 전송하고 로그스테시 화면에서 보면 아래와 같이 출력됨을 확인할 수 있다. 

```
Consumer clientId=logstash-0, groupId=logstash] Setting newly assigned partitions [ykkim-topic-0]
{
       "message" => "Hello Logstash1",
    "@timestamp" => 2019-02-17T11:40:00.208Z,
      "@version" => "1"
}
{
       "message" => "Hello Logstash2",
    "@timestamp" => 2019-02-17T11:40:09.514Z,
      "@version" => "1"
}
{
       "message" => "Hello Logstash3",
    "@timestamp" => 2019-02-17T11:40:16.496Z,
      "@version" => "1"
}
```

Elasticsearch에서도 잘 저장되고 있음을 확인할 수 있다. 

>  GET kafka-test-2019-02-17/_search

```json 
{
  "took" : 33,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "kafka-test-2019-02-17",
        "_type" : "_doc",
        "_id" : "7IFB-2gBFeMvUP3hhPeT",
        "_score" : 1.0,
        "_source" : {
          "message" : "Hello Logstash1",
          "@timestamp" : "2019-02-17T11:40:00.208Z",
          "@version" : "1"
        }
      },
      {
        "_index" : "kafka-test-2019-02-17",
        "_type" : "_doc",
        "_id" : "7YFB-2gBFeMvUP3hpPc4",
        "_score" : 1.0,
        "_source" : {
          "message" : "Hello Logstash2",
          "@timestamp" : "2019-02-17T11:40:09.514Z",
          "@version" : "1"
        }
      },
      {
        "_index" : "kafka-test-2019-02-17",
        "_type" : "_doc",
        "_id" : "7oFB-2gBFeMvUP3hv_dg",
        "_score" : 1.0,
        "_source" : {
          "message" : "Hello Logstash3",
          "@timestamp" : "2019-02-17T11:40:16.496Z",
          "@version" : "1"
        }
      }
    ]
  }
}
```

