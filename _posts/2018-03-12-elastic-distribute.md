---
layout: post
title: Elasticsearch에서 분산모델(Distributed Model)
date: 2018-03-12
categories: elasticsearch
---



### 1. 노드(node)란?

엘라스틱에서 분산처리모델에서 알아보도록 하자. 분산처리에 앞아서 가장 먼저 알아야 할 것이 바로 노드(node)이다. 노드란 무엇인가? 노드의 특징은 크게 3가지로 정의할 수 있다.

- 노드는 엘라스틱 서치의 인스턴스(instance)이다.
- 노드는 자신만의 이름을 갖는다
- 노드는 가동할 때 자신의 유일키(UID)를 갖는다.

노드는 엘라스틱 서치의 인스턴스이다. 즉 jvm에서 자신만의 프로세스를 갖는 다는 말이다. 또한 노드는 elasticsearch.yml에서 `node.name` 을 통해서 이름을 정할 수 있고 실행시 이름을 부여할 수도 있다. 

``` bash 
/bin/elasticsearch -E node.name=master_01
```

위와 같이 실행시 옵션으로 노드명을 부여할 수 있다. 

### 2. 클러스터(Cluster)란?

엘라스틱에서의 모든 것을 클러스터 내에 생성된다. 따라서 모든 노드는 클러스트에 포함된다.  클러스트는 이름을 가지고 있다. 디폴트명은 `elasticsearch`이다.  클러스트도 역시 application.yml에서 `cluster.name`에서 설정하거나 실행시 부여할 수 있다. 

``` bash
./bin/elasticsearch -E node.name=master_01 -E cluster.name=main_cluster_1
```

모든 클러스트는 `cluster state` 라는 참조 정보를 가지고 있다. 이 cluster state는 각 노드안에 저장된다. 

![cluster](/assets/images/cluster1.png)

노드안에 있는 cluster state는 결코 수정이 불가능하다. 



### 3. 슬레이브 노드 추가

노드에는 마스터 노드(Master Node)라는 것이 있다. 노드 한개만 처음 실행할 때 그 노드는 마스터 노드가 된다(만약 이것을 막으려면 node.master:false 로 설정해야 한다)  노드는 마스터노드를 선출하여 결정한다. 만약 유일한 노드라면 그 노드가 마스터 노드가 된다.  

그리고 데이타 노드(Data Node)가 있다. 이것은 말 그대로 데이터를 저장하는 노드이다. 그럼 노드 하나면 실행하면 마스터노드이자 데이터 노드가 되는 것이다. 즉 여러개의 노드로 구성될 경우 한 노드는 마스터, 나머지는 데이터노드로 지정할 수 있다.

![cluster](/assets/images/cluster2.png)

그림과 같이 master_node_1은 마스터 노드이자 데이터 노드가 된 경우다. 우리고 PUT, DELETE등으로 인덱스를 추가하거나 삭제되면 데이터를 담고 있는 노드는 cluster state를 업데이트 하고 그 응답을 클라이언트에 돌려준다. 

만약 두번째 노드를 생성하면 어떻게 될 것인가? elasticsearch를 복사후에 새로운 노드로 실행시켜보자.

``` bash 
./bin/elasticsearch -E node.name=slave_01 -E cluster.name=main_cluster_1
```

cluster_name은 이전에 master_01에서 만든 main_cluster_1으로 했다. 실행되면 다음과 같은 에러가 반복되서 출력됨을 확인할 수 있다.

```bash 
{% raw %}
[2018-03-12T21:55:00,738][INFO ][o.e.d.z.ZenDiscovery     ] [slave_01] failed to send join request to master [{master_01}{IGp-s0MnRveYsX1q0xh_iA}{cV-g8WUJReahGervsb_1AQ}{127.0.0.1}{127.0.0.1:9300}], reason [RemoteTransportException[[master_01][127.0.0.1:9300][internal:discovery/zen/join]]; nested: IllegalArgumentException[can't add node {slave_01}{IGp-s0MnRveYsX1q0xh_iA}{A2QxhezXQPqWRRtwNQn_Uw}{127.0.0.1}{127.0.0.1:9301}, found existing node {master_01}{IGp-s0MnRveYsX1q0xh_iA}{cV-g8WUJReahGervsb_1AQ}{127.0.0.1}{127.0.0.1:9300} with the same id but is a different node instance]; ]
{% endraw %}
```

이런 에러가 발생한 것은 우리가 기존에 elasticsearch를 그냥 복사해서 사용해서 그렇다. 그 안에 이미 data 디렉토리가 슬레이드도 같아서 생긴 문제다. 이럴때는 슬레이브에 있는 data 디렉토리를 삭제해주어야 한다. 

다음 세컨드 노드에서는 포트를 9201로 변경하도록 하겠다.

```groovy
http.port: 9201
```

이제 세컨드 노드를 재시작하면 정상가동되고 `127.0.0.1:9200` 은 마스터 `127.0.0.1:9201`은 슬레이브로 구성되어 있다.

```bash 
{% raw %}
[2018-03-23T22:46:48,196][INFO ][o.e.c.s.ClusterService   ] [slave_01] detected_master {master_01}{YQLO20xvSjGDIBLKuHg4Lg}{Ka9fgfy5RRmkk9f9Yv5PzA}{127.0.0.1}{127.0.0.1:9300}, added {{master_01}{YQLO20xvSjGDIBLKuHg4Lg}{Ka9fgfy5RRmkk9f9Yv5PzA}{127.0.0.1}{127.0.0.1:9300},}, reason: zen-disco-receive(from master [master {master_01}{YQLO20xvSjGDIBLKuHg4Lg}{Ka9fgfy5RRmkk9f9Yv5PzA}{127.0.0.1}{127.0.0.1:9300} committed version [19]])
{% endraw %}
```

정상적으로 붙으면 `detected_master` 라고 보인다. 이제 실제 잘 되는지 테스트해보자 

### 4. 마스터 - 슬레이브 테스트 

위에서 우리는 마스터는 `localhost:9200` 슬레이브는 `localhost:9201` 으로 설정했다. 각각 브라우저로 접속하면 각각 정보가 잘 보일것이다. 

그러면 마스터에 인덱스를 추가해보도록 하자. 마스터에 추가하고 슬레이브에서 그 인덱스가 나오면 성공한 것이다. 

```bash 
curl -XPUT "http://localhost:9200/my_index/doc/1" -H "Content-Type: application/json" -d '
{
    "username": "harrison",
    "comment": "Mt favorite movie is Star Wars!"
}'
```

마스터에 정상적으로 들어가면 이제 위에 인덱스를 슬레이브에서 조회해보자.

```bash 
curl -XGET "http://localhost:9201/my_index/_search" -H "Content-Type: application/json"
```

```json 
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "my_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "harrison",
          "comment": "Mt favorite movie is Star Wars!"
        }
      }
    ]
  }
}
```

정상적으로 슬레이브에서 잘 보인다. 마스터 슬레이브 구성이 성공한 것이다. 슬레이브에서 추가,수정,삭제해도 마스터에 바로 반영이 된다. 그러면 슬레이브를 죽인 상태에서 마스터에 추가하고 슬레이브를 살리면 슬레이브에서 가져오는지 확인해보자. 

```bash 
curl -XPUT "http://localhost:9200/my_index/doc/2" -H "Content-Type: application/json" -d '
{
    "username": "richard",
    "comment": "Mt favorite movie is Golira!"
}'
```

마스터에 추가하고 조회하면 아래에 같이 2건이 나온다. 

```json 
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "my_index",
        "_type": "doc",
        "_id": "2",
        "_score": 1,
        "_source": {
          "username": "richard",
          "comment": "Mt favorite movie is Golira!"
        }
      },
      {
        "_index": "my_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "harrison",
          "comment": "Mt favorite movie is Star Wars!"
        }
      }
    ]
  }
}
```

이제 슬레이브를 가동하고 조회해보자. 실제로 슬레이브를 조회해도 동일한 결과가 나오는 것을 확인할 수 있다. 

```bash 
curl -XGET "http://localhost:9201/my_index/_search"  
```

간단히 엘라스틱서치에서 분산처리를 해보았다. 생각보다 어렵지 않고 쉽게 구현이 되었다. 그런데 실제 분리된 서버에서 적용하려면 슬레이브 elasticsearch.yml에는 `discovery.zen.ping.unicast.hosts:` 에 마스터 ip 정보를 세팅해야 한다. 

