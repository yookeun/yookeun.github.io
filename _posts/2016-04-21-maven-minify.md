---
layout: post
title:  "Maven에서 js 파일 minify 하기"
date:   2016-04-21
categories: java
---
Maven에서 js파일을 압축하여 패키징하도록 해보자.
일단, 개발자들의 로컬에서는 압축되지 않은 상태로 사용하다고 패키징할때 압축하여 배포하는 방식으로 사용하면 편하다.
보통은 aa.js을 aa.min.js으로 압축하지만 만약 각각의 jsp나 html파일에 js경로가 적혀있다면 각각 수정해야 하는 번거로움이 생긴다.
그러므로 기존의 `src/js/aa.js` 파일을 압축하여 특정경로에 `src/jsmin/aa.js` 파일등으로 압축하여 옮기고 배포시에는 `src/js` 폴더는 빼도록 하자.

### 1. maven minify plugin 설치

js(혹은 css)파일을 압축하려면 maven 플러그인이 필요하다.
[Minify Maven Plugin](http://samaxes.github.io/minify-maven-plugin/)을 사용하자.  
pom.xml에서 아래 소스를 `<plugins></plugins>`안에 넣자.

```xml
<plugin>
  <groupId>com.samaxes.maven</groupId>
    <artifactId>minify-maven-plugin</artifactId>                   
      <version>1.7.4</version>
      <executions>
        <execution>
          <id>minify</id>
          <phase>process-resources</phase>
          <goals>
            <goal>minify</goal>
          </goals>
          <configuration>
            <charset>utf-8</charset>                            
            <skipMerge>true</skipMerge>
            <nosuffix>true</nosuffix>
            <jsEngine>CLOSURE</jsEngine>                   
            <jsSourceDir>resources/js/cores</jsSourceDir>
            <jsTargetDir>resources/js/coresmin</jsTargetDir>
            <jsSourceIncludes>
            <jsSourceInclude>*.js</jsSourceInclude>
            </jsSourceIncludes>                                                                              		
          </configuration>
        </execution>
    </executions>
</plugin>
```

`<jsSourceDir>resources/js/cores</jsSourceDir>`에 원본소스 js파일들의 경로를 적는다.
`<jsTargetDir>resources/js/coresmin</jsTargetDir>`에 압축하여 복사될 경로를 적는다.

패키징할때 `resources/js/cores`은 필요가 없다. 압축된 파일을 사용할 것이므로..
그래서 war로 묶어주는 플러그인에 `packagingExcludes` 옵션을 넣어 해결한다.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <version>2.3</version>
  <configuration>
    <warName>test-web</warName>
     <packagingExcludes>resources/js/cores/**</packagingExcludes>
  </configuration>
</plugin>
```

pom.xml 설정은 이것으로 끝났다. 하지만 문제가 있다. 위에 처럼 패키징하면 다음과 같이
각각의 jsp, html파일에 script 경로를 모두 `resources/js/coresmin/`을 수정해야한다.

```html
<script src="resources/js/cores/test.js"></script>
```

즉, 로컬에서 패키징하기전에 수정해야 한다는 것이다.이 부분을 해결할 플러그인을 찾지 못해 다음과 같은 방법으로 해결했다.


### 2. properties를 이용하여 경로 변경하기

본 블로그의 <[Maven에서 서버별 패키징하기](http://yookeun.github.io/java/2015/07/20/maven-pacaking/)>로 패키징이 되어있어야 한다.
그 다음에 <[Spring에서 Properties 사용하기](http://yookeun.github.io/java/2015/12/22/spring-properties/)>로 각 패키징폴더의 properties를 이용해서 패키징하는 것이다.

일단, 개발자 로컬에서는 `src/main/resources-local/properties/test.properties` 라는 파일을 만들고 그안에
`jsdir=cores`라는 설정을 넣는다.
그 다음 패키징할 경로 `src/main/resources-demo/properties/test.properties`에 역시 같은 파일을 만들고
`jsdir=coresmin`라는 설정을 넣는다.

이번에는 controller를 수정하자.

```java
@Value("#{enpick['jsdir']}")
private String jsdir;

/**
 * 루트경로
 */
@RequestMapping(value = "/", method = RequestMethod.GET)
protected String home(Locale locale, Model model) {
  model.addAttribute("jsdir", jsdir);
  return "index";
}
```

이런식으로 controller에서 Properties를 읽어들인후 Model를 이용하여 넣어준다.
이제 jsp파일을 보자.

```html
<script src="resources/js/${jsdir}/test.js"></script>
<script src="resources/js/${jsdir}/pay.js"></script>
```

바로 저 `${jsdir}`에 각 패키징에 따른 js디렉토리가 보여진다.
이렇게 되면 로컬에서 압축안된 파일로 작업해도 경로를 다시 변경할 필요가 없고 패키징하면 압축된 경로로 보이게 된다.

### 3. 패키징하기
자, 이제 로컬에서 마음껏 작업하고 아래처럼 패키징하면 원본 js파일을 해당경로로 압축하고 원본은 삭제하면서 war로 만들어준다.
Jenkins와 연동되어 있다면 그냥 Push 하면 상황종료다!

```bash
mvn package ( or mvn package -P demo)
```

### 4. 주의사항
`mvn package` 하면 만약 js파일에 문제가 있다면 컴파일 에러가 표시된다.

```bash
Apr 21, 2016 3:43:03 PM com.google.javascript.jscomp.LoggerErrorManager println
SEVERE: twcompetitor.js:15: ERROR - Parse error. IE8 (and below) will parse trailing commas in array and object literals incorrectly. If you are targeting newer versions of JS, set the appropriate language_in option.
		channel_id : 0,
		              ^

Apr 21, 2016 3:43:03 PM com.google.javascript.jscomp.LoggerErrorManager printSummary
WARNING: 1 error(s), 0 warning(s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.264 s
[INFO] Finished at: 2016-04-21T15:43:03+09:00
[INFO] Final Memory: 9M/219M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal com.samaxes.maven:minify-maven-plugin:1.7.4:minify (minify) on project enpick: org.mozilla.javascript.EvaluatorException: Parse error. IE8 (and below) will parse trailing commas in array and object literals incorrectly. If you are targeting newer versions of JS, set the appropriate language_in option. -> [Help 1]
```

문제는 해당 js파일에 마지막 객체 속성(`age: "",`)에 콤머 들어간 것이다.

```javascript
var TEST = {
		name : "",
		age: "",
};
```

<[Closure Compiler](https://closure-compiler.appspot.com/home)>에서는 이럴 경우 알아서 마지막 콤머를 제거하고 압축하는데 `maven minify plugin`는 구글 압축기를 사용하는 것 같은데
그렇게 해주지 않고 바로 에러처리하므로 주의해야 한다.

정상적으로 컴파일되면 BUILD SUCCESS가 보일 것이다.
