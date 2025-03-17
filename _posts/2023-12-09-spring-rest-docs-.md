---
layout: single
title: "Spring Rest docs 적용해 보기"
date: 2023-12-09
categories: [java]
tags: [java, spring]
---

Spring Rest Docs를 Gradle에 추가한다.

plugins, configurations, dependencies에 각각 작성한다.

```groovy
plugins {
        ...
    id 'org.asciidoctor.jvm.convert' version '3.3.2'
}

...

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    asciidoctorExt
}

...

dependencies {
    //rest docs
    asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}
```

그리고 asciidoctor를 사용하기 위한 task를 작성한다. asciidoctor은 문서 모델로 구문을 분석해서 HTML형식으로 출력해주는 일종의 텍스트 처리기이다.

```groovy
//rest docs
ext {
    set('snippetsDir', file("build/generated-snippets"))
}

asciidoctor {
    dependsOn test
    inputs.dir snippetsDir
}


tasks.named('test') {
    outputs.dir snippetsDir
    useJUnitPlatform()
}

tasks.named('asciidoctor') {
    inputs.dir snippetsDir
    dependsOn test
}


test {
    outputs.dir snippetsDir
}

asciidoctor {
    configurations 'asciidoctorExt'
    baseDirFollowsSourceFile()
    inputs.dir snippetsDir
    dependsOn test
}

asciidoctor.doFirst {
    delete file('src/main/resources/static/docs')
}

task copyDocument(type: Copy) {
    dependsOn asciidoctor
    from file("build/docs/asciidoc")
    into file("src/main/resources/static/docs")
}

build {
    dependsOn copyDocument
}
```

이것으로 설정 작업은 끝이 났고, 실제 테스트 코딩을 작성해 보자.

```java
    @DisplayName("Save item")
    @Test
    void testSaveItem() throws Exception {
        //given
        ItemRequestDto requestDto = ItemRequestDto.builder()
                .itemName("testItem")
                .itemType(ItemType.FOOD)
                .price(10L)
                .build();

        //when
        ResultActions result = mockMvc.perform(post(prefixUrl)
                .contextPath(contextPath)
                .contentType(MediaType.APPLICATION_JSON)
                .header(testToken.AUTHORIZATION,
                        String.format("%s%s", testToken.BEARER, getAccessToken()))
                .content(objectMapper.writeValueAsString(requestDto)));

        //then
        result.andExpect(status().isOk())
                .andExpect(jsonPath("$.itemName").value("testItem"))
                .andReturn();

        //docs
        result.andDo(document("save-item",
                preprocessRequest(prettyPrint()),
                preprocessResponse(prettyPrint()),
                requestFields(
                        fieldWithPath("itemName").description("ITEM NAME"),
                        fieldWithPath("itemType").description("ITEM TYPE(FOOD, BOOK, CLOTHES"),
                        fieldWithPath("price").description("ITEM PRICE")
                ),
                responseFields(
                        fieldWithPath("id").description("ID"),
                        fieldWithPath("itemName").description("ITEM NAME"),
                        fieldWithPath("itemType").description("ITEM TYPE(FOOD, BOOK, CLOTHES"),
                        fieldWithPath("price").description("ITEM PRICE")
                )
        ));

    }
```

아이템을 저장하는 테스트 코드이다. given > when > then 에 이어서 docs에서 Spring Rest Docs를 작업을 처리하는 것이다. 이름에서 알수 있듯이 requestFields에 전달 하는 파라미터를 정의하고 responseFields에 리턴되는 값을 정의하면 된다.

다음은 AsciiDoc를 작성해 보자.

`saveItem`를 지정해서 adoc파일(AsciiDoc로 만들어진 파일)을 저장할 경로를 지정한다. 그렇게 하면 빌드할때 해당 파일이 생성된다. 먼저 src/doc/asiidoc 라는 디렉토리를 생성한다. 여기에서 index.adoc를 만들어준다. 이 파일에서 각 adocs파일을 취합하도록 한다.

만약 Intelij를 사용한다면 AsciiDoc이라는 플러그인을 설치하면 adoc파일을 작성 및 미리보기 기능이 제공된다.

아래는 작성한 index.adoc 파일이다

```asciiarmor
= TEST API
notification-api-docs
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 4
:sectlinks:

ifndef::snippets[]
:snippets: ./build/generated-snippets
endif::[]

include::Member-API.adoc[]
include::Item-API.adoc[]
include::Order-API.adoc[]
```

include에 각 adoc파일을 만든다. 여기서는 Member관련 API, Item 관련 API, Order 관련 API에 대한 각각의 adoc파일을 include해준다.

예제로 Item 관련 API가 정의된 Item-API.adoc를 만든다.

```asciiarmor
== Item-API

=== Save-Item

==== Request Sample
include::{snippets}/save-item/http-request.adoc[]

==== Request Field
include::{snippets}/save-item/request-fields.adoc[]

==== Response Sample
include::{snippets}/save-item/http-response.adoc[]

==== Response Fields
include::{snippets}/save-item/response-fields.adoc[]



=== Update-Item

==== Request Sample
include::{snippets}/update-item/http-request.adoc[]

==== Request Field
include::{snippets}/update-item/request-fields.adoc[]

==== Response Sample
include::{snippets}/update-item/http-response.adoc[]

==== Response Fields
include::{snippets}/update-item/response-fields.adoc[]


=== View-Item

==== Request Sample
include::{snippets}/get-item/http-request.adoc[]

==== Request Field
include::{snippets}/get-item/path-parameters.adoc[]

==== Response Sample
include::{snippets}/get-item/http-response.adoc[]

==== Response Fields
include::{snippets}/get-item/response-fields.adoc[]


=== List-Item

==== Request Sample
include::{snippets}/list-item/http-request.adoc[]

==== Request Field
include::{snippets}/list-item/query-parameters.adoc[]

==== Response Sample
include::{snippets}/list-item/http-response.adoc[]

==== Response Fields
include::{snippets}/list-item/response-fields.adoc[]
```

Item에 필요한 Save-Item, Update-Item, View-Item, List-Item에 대한 api에 대한 내용을 기술한다.

문서에 `include::{snippets}/save-item/http-request.adoc[]` 부분에 대한 문서 저장은 `build/generated-snippets` 에 저장되어있는 위치이다. \*

`src/main/resources/static/doc` 디렉토리를 생성하면 adoc파일에 해당 경로의 html로 변환되어 저장된다. 그래서 최종 빌드를 하게 되면 index.html, Item-API.html, Member-API.html, Order-API.html 파일이 만들어지고 index.html에서 확인 할 수가 있다.

마지막으로 jar로 배포하고 실행하면 아래 경로에서 확인이 가능하다.

**\*http://localhost:8080/docs/index.html\***.

![erd](/assets/images/rest_docs.png)
