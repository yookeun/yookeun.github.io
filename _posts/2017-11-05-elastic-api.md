---
layout: post
title: Elasticsearch와 spring 연동
date: 2017-11-05
categories: elasticsearch
---

Spring에서 Elasticsearch와 연동해보자. Elasticsearch는 기본적으로 http통신의 RestAPI이기 때문에 스프링에서 제공하는 RestTemplete를 이용해도 된다. 여기서는 Elasticsearch에서 제공하는 라이브러리를 이용해보도록 해보자. 

### 1. 엘라스틱서치 라이브러리 설치

엘라스틱서치에서 제공하는 [RestAPI](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html) 에는 공식적으로 두 가지 api가 존재한다.

- java Low Level REST Client
- java High Level REST Client

기본적인 PUT, POST, GET, DELETE 등은 `low level` 버전으로 충분히 커버가 된다. `high level` 버전의 가장 큰 특징은 비동기메소드를 제공한다는 것이다. 여기서는 `low level` 버전으로 시작해보겠다. 

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>5.6.3</version>
</dependency>
```

```groovy
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-client:5.6.3'
}
```

관련 라이브러리를 메이븐, 그래들 버전에 따라 받으면 된다. 

### 2. 구현

Spring boot를 기준으로 설명한다. 아래와 같이 클래스를 생성한다. 

``` java
@Component
public class ElasticApi {

    @Value("${elasticsearch.host}")
    private String host;

    @Value("${elasticsearch.port}")
    private int port;

    /**
     * 엘라스틱서치에서 제공하는 api를 이용한 전송메소드
     * @param method
     * @param url
     * @param obj
     * @param jsonData
     * @return
     */
    public Map<String, Object> callElasticApi(String method, String url, Object obj, String jsonData) {
        Map<String, Object> result = new HashMap<>();

        String jsonString;
        //json형태의 파라미터가 아니라면 gson으로 만들어주자.
        if (jsonData == null) {
            Gson gson = new Gson();
            jsonString = gson.toJson(obj);
        } else {
            jsonString = jsonData;
        }

        //엘라스틱서치에서 제공하는 restClient를 통해 엘라스틱서치에 접속한다
        try(RestClient restClient = RestClient.builder(new HttpHost(host, port)).build()) {
            Map<String, String> params =  Collections.singletonMap("pretty", "true");
            //엘라스틱서치에서 제공하는 response 객체
            Response response = null;

            //GET, DELETE 메소드는 HttpEntity가 필요없다
            if (method.equals("GET") || method.equals("DELETE")) {
                response = restClient.performRequest(method, url, params);
            } else {
                HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);
                response = restClient.performRequest(method, url, params, entity);

            }
            //앨라스틱서치에서 리턴되는 응답코드를 받는다
            int statusCode = response.getStatusLine().getStatusCode();
            //엘라스틱서치에서 리턴되는 응답메시지를 받는다
            String responseBody = EntityUtils.toString(response.getEntity());
            result.put("resultCode", statusCode);
            result.put("resultBody", responseBody);
        } catch (Exception e) {
            result.put("resultCode", -1);
            result.put("resultBody", e.toString());
        }
        return result;
    }
}
```

 주석에 설명이 있는 것처럼 `callElasticApi` 메소드를 만들고 엘라스틱서치 api에서 제공하는 RestClient를 통해서 연동하면 간단히 해결된다. 단 다음과 같은 객체 형식으로 넘길 경우 객체를 json으로 변환시켜야 하는데 간단하게 gson으로 처리하도록 했다.  그리고 엘라스틱서치에 연동 host, port등은 `application.properties`에 등록하면 편리하다. 

이제 실행하는 메소드를 작성하자.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ElasticSpringExampleApplicationTests {

	@Autowired
	ElasticApi elasticApi;

	private final String ELASTIC_INDEX = "test_index";
	private final String ELASTIC_TYPE = "test_type";

	@Test
	public void 엘라스틱서치_POST_전송() {
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE;
		Weather weather = new Weather();
		weather.setCity("Seoul");
		weather.setTemperature(10.2);
		weather.setSeason("Winter");

		Map<String, Object> result = elasticApi.callElasticApi("POST", url, weather, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}


	@Test
	public void 엘라스틱서치_PUT_전송() {
		String id = "122345";
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE+"/"+id;
		Weather weather = new Weather();
		weather.setCity("Tokyo");
		weather.setTemperature(12.3);
		weather.setSeason("Winter");

		Map<String, Object> result = elasticApi.callElasticApi("PUT", url, weather, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}


	@Test
	public void 앨라스틱서치_GET_전송() {
		String id = "122345";
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE+"/"+id;
		Map<String, Object> result = elasticApi.callElasticApi("GET", url, null, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}


	@Test
	public void 앨라스틱서치_DELETE_전송() {
		String id = "122345";
		String url = ELASTIC_INDEX + "/" + ELASTIC_TYPE+"/"+id;
		Map<String, Object> result = elasticApi.callElasticApi("DELETE", url, null, null);
		System.out.println(result.get("resultCode"));
		System.out.println(result.get("resultBody"));
	}
```

엘라스틱 인덱스명은 `test_index`로 하고 타입은 `test_type`으로 실행했다. 실제 수행되면 콘솔에 아래와 같이 출력이 됨을 확인할 수 있다.

```json
[post]
201
{
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "AV-Mj6uo54tuIMbWUlfS",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
[put]
201
{
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "122345",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
[get]
200
{
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "122345",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "city" : "Tokyo",
    "temperature" : 12.3,
    "season" : "Winter"
  }
}
[delete]
200
{
  "found" : true,
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "122345",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  }
}
```

[post] - [put] - [get] - [delete]식으로 잘 출력이 됨을 확인할 수 있다. 그런데 실무에서는 아마도 `x-pack`를 설치해서 허가된 요청만 처리할 있게 할 것이다. 그런 경우에는 x-pack에서 별도의 계정을 생성한 후에 해당 계정과 패스워드를 application.properties등에 등록하고 api를 호출할때 헤더에 포함시켜 주면 된다. 

### 3. API에 인증(Authorization) 추가 

상단의 소스에서 `callElasticApi`메소드에 인증부분을 추가한 `callElasticApiAuth`를 구성했다.

```java
public Map<String, Object> callElasticApiAuth(String method, String url, Object obj, String jsonData) {
        Map<String, Object> result = new HashMap<>();

        String jsonString;
        //json형태의 파라미터가 아니라면 gson으로 만들어주자.
        if (jsonData == null) {
            Gson gson = new Gson();
            jsonString = gson.toJson(obj);
        } else {
            jsonString = jsonData;
        }

        //엘라스틱서치에서 제공하는 restClient를 통해 엘라스틱서치에 접속한다
        try(RestClient restClient = RestClient.builder(new HttpHost(host, port)).build()) {
            String auth = user+":"+password;
            String basicAuth  = "Basic "+ Base64.getEncoder().encodeToString(auth.getBytes());
            Header[] headers = {
                    new BasicHeader("Authorization", basicAuth)
            };
            Map<String, String> params =  Collections.singletonMap("pretty", "true");
            //엘라스틱서치에서 제공하는 response 객체
            Response response = null;

            //GET, DELETE 메소드는 HttpEntity가 필요없다
            if (method.equals("GET") || method.equals("DELETE")) {
                response = restClient.performRequest(method, url, params, headers);
            } else {
                HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);
                response = restClient.performRequest(method, url, params, entity, headers);

            }
            //앨라스틱서치에서 리턴되는 응답코드를 받는다
            int statusCode = response.getStatusLine().getStatusCode();
            //엘라스틱서치에서 리턴되는 응답메시지를 받는다
            String responseBody = EntityUtils.toString(response.getEntity());
            result.put("resultCode", statusCode);
            result.put("resultBody", responseBody);
        } catch (Exception e) {
            result.put("resultCode", -1);
            result.put("resultBody", e.toString());
        }
        return result;
    }
```

 추가된 부분은 바로 헤더처리 부분이다. 엘라스틱에서 `Authorization` 부분은 `<user_name>:<password>` 로 구성되고 이것을 `base64`로 인코딩해주면 된다. 그런 다음에 생성된 Headers[] headers 를 `performRequest` 메소드에 파라미터를 추가해주면 된다.  

이렇게 엘라스틱에서 제공하는 client api를 통해 간단히 연동을 할 수 있다. 전체 소스는 아래에서 확인할 수 있다.

- [elastic-spring-example](https://github.com/yookeun/elasic-spring-example)

