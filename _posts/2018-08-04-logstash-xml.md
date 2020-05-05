---
layout: post
title: "Logstash에서 xml 파일 파싱작업"
date: 2018-08-04
categories: elasticsearch
---

logstash에서 xml파일을 읽고 처리하는 방법을 처리해본다.

### 1. xml 파일 읽고 처리  

아래와 같은 일반적인 xml 구조로 테스트해본다.

```xml 
<?xml version="1.0" encoding="utf-8"?>
<simpleData>
    <simpleInfo>
        <userName>kim</userName>
        <age>40</age>
        <gender>male</gender>
        <regDate>2018-08-01 10:00:00</regDate>
    </simpleInfo>
    <simpleInfo>
        <userName>lee</userName>
        <age>41</age>
        <gender>female</gender>
        <regDate>2018-08-02 11:00:00</regDate>
    </simpleInfo>
    <simpleInfo>
        <userName>park</userName>
        <age>42</age>
        <gender>female</gender>
        <regDate>2018-08-03 12:00:00</regDate>
    </simpleInfo>    
</simpleData>
```

`simpleInfo` 엘리먼트에 데이터가 저장된 xml이다. 이제 해당 xml을 담을 매핑 인덱스를 만들어보자

``` json 
PUT simple-xml 
{
    "mappings": {
        "simplexml": {
            "properties": {
                "simpleInfo": {
                    "properties": {
                        "userName": {
                            "type": "keyword"
                        },
                        "age": {
                            "type": "integer"
                        },
                        "gender": {
                            "type": "keyword"
                        },
                        "regDate": {
                            "type": "date",
                            "format": "yyyy-MM-dd HH:mm:ss"                          
                        }
                    }
                }
            }
        }
    }
}
```

`simplxml` 라는 인덱스를 생성했다.  `regDate`  타입을 `date`로 설정했고, `format` 실제 데이터에 맞게 처리했다. kibana에서는 date 필드를 UTC로 처리하기 때문에 위에 regDate의 날짜를 kibana에선 UTC로 읽어들인다. 

### 2. logstash 설정 

이제 xml를 읽여들여 처리하는 logstash conf파일을 만들어보자.  먼저 `input`  부분이다.

```
input {
    file {
        path => "/Users/ykkim/workspace/logstash-parsing-example/xml/data/simple.xml"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => multiline {
            pattern => "<simpleInfo>|</simpleData>"
            negate => "true"   
            what => "previous"
            auto_flush_interval => 1   
        }
    }
}
```

실제 xml 파일을 읽여들여 처리하는 부분이다. `file` 통해 읽으면 되고 `start_position => beginning` 으로 시작부분을 설정해준다. `sincedb_path => /dev/null` 부분은 실제 한번 읽은 파일은 logstash에서 별도로 저장해서 파일내용이 추가되면 감지를 해서 처리하는데 이 부분을 /dev/null로 해주면 매번 처음부터 파일을 읽어들인다. 이것은 개발테스트용으로 설정한 것이기 때문에 운영에서는 제거해주면 된다. 

그다음 `codec => mulitline` 를 통해서 반복적으로 구현되는 부분을 처리한다. `pattern` 부분에 반복시작/종료부분을 설정해준다. `negate` 는 정규식을 사용여부 옵션이고 `what` 은 항상 파일 마지막부터 읽어들이니까 previous로 설정한다. `auto_flush_interval` 부분도 설정하도록 하자.  자세한 옵션설명은 logstash 사이트를 참조하자. 

다음은 `filter`부분이다.

```
filter { 
    if [message] =~ "^<\?xml .*" {
        drop {}
    }
    xml {
        source => "message"
        remove_namespaces => "true"
        force_array => "false"
        target => "simpleInfo"
        store_xml => "true"                
    }

    mutate {        
        remove_field => ["@version", "beat", "count", "fields", "input_type","offset","source","type","host","tags","path"]
    }
}
```

`<xml>` 로 시작되는 부분은 필요없으므료 제거하도록 한다. `force_array` 는 말 그대로 배열사용여부인데 우리는 그냥 row데이터로 처리할 것이나 반드시 false로 둔다. `target` 매핑때 정의한 properties로 하면 된다. store_xml은 반드시 true로 해준다. 그리고 `mutate` 을 사용해서 `remove_filed` 를 통해 불필요한 정보는 제거하자. 

마지막으로 output은 아래처럼 처리하면 된다. 

```
output {
    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts => ["http://127.0.0.1:9200"]
        index => "simple-xml"
        document_type => "simplexml"
    }
}
```

이렇게 해서 logstash를 실행하면 정상적으로 처리됨을 확인할 수 있다. kibana에서 date 필드를 `@timestamp` 대신에 위에서 정의한 `simpleInfo.regDate`로 하면 처리된 날짜로 통계를 만들 수 있다.

