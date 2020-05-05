---
layout: post
title: "Spring에서 Pebble Template 사용"
date: 2018-03-10
categories: java
---

Spring에서 아마 지금까지 가장 많이 사용된 템플릿은 jstl 일것이다. jstl이 너무 올드하고 지겨운 감도 있어 여러 템플릿을 찾던 중에 pebble을 알게 되었다. 무엇보다도 성능이 좋고, 사용법이 간단한 것 같다.  

![pebble](/assets/images/pebble.png)

가장 많이 사용되는 freemarker, handle, mustac 보다 성능이 월등하다. Rocker를 살펴봤지만 pebble이 좀더 러닝커브가 적게 판단되어 최종적으로 pebble를 선택하게 되었다. 

[Pebble](http://www.mitchellbosecke.com/pebble/home) 페이지에 가면 다음과 같이 정의되어 있다.

> Pebble is a java template template engine inspired by [Twig](http://twig.sensiolabs.org/)

한 마디로 자바기반의 템플릿 엔진이다. 여기서는 springboot 에서 pebble을 적용하는 방법에 대해 알아본다.

### 1. Spring boot에서 pebble 설치 

먼저 gradle에서 pebble-spring 라이브러리를 추가한다

```groovy
compile 'com.mitchellbosecke:pebble-spring4:2.4.0'
```

그리고 javaconfig에서 빈설정등을 추가해주어야 한다. 

```java 
    @Bean
    public SpringExtension springExtension() {
        return new SpringExtension();
    }

    @Bean
    public PebbleEngine pebbleEngine() {
        return new PebbleEngine.Builder()
                .loader(this.templateLoader())
                .extension(springExtension())
                .cacheActive(false)   //이 옵션을 안주면 html수정시 refresh해도 안됨,
                .build();
    }

    @Bean
    public ViewResolver viewResolver() {
        PebbleViewResolver resolver = new PebbleViewResolver();
        resolver.setCache(true);
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".html");
        resolver.setContentType("text/html; charset=UTF-8");
        resolver.setPebbleEngine(pebbleEngine());
        return resolver;
    }
```

`SpringExtension` 을 생성하고 `PebbleEngine`에서 extension에 추가하고 `viewResolve`에서 `PebbleViewResolver`를 등록해주면 된다. 보통 pebble의 확장자는 `peb` 으로 사용하는데 `html`로 변경해서 사용할 수 있다. 

이것으로 설정은 모두 끝났다.

### 2. Pebble 레이아웃 적용

 jstl 를 사용했다면 레이아웃(Sitemash 등)을 별도로 설치했을것이다. 그러나 pebble은 레이아웃 기능을 자체적으로 제공한다.  먼저 가장 베이스가 되는 페이지를 만든다. 

``` html 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pebble Test Board</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-light bg-light">
    <a class="navbar-brand" href="#">Board</a>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
            <li class="nav-item active">
                <a class="nav-link" href="#">목록 <span class="sr-only">(current)</span></a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="#">글쓰기</a>
            </li>
        </ul>
    </div>
</nav>
{% raw %}
{% block content %}  
{% endblock %}
{% endraw %}
</body>
</html>
```

`block content ` /`endblock` 부분이 바로 Body부분이다.  이제 실제 리스트를 구현하는 페이지를 만들어보자.

Controller 부분에서는 기존에 하던 방식으로 처리하면 된다.  ModelAndView이나 Model 로 처리하면 된다. 

```java 
    @GetMapping({"/", "/board/list"})
    public ModelAndView boardList(@ModelAttribute("search") BoardSearch search) {
        ModelAndView mav = new ModelAndView("board/list");
        int total = boardService.selectBoardCount();
        search.setTotalCount(total);
        search.makePagination();
        List<Board> boardList = boardService.selectBoard(search);
        mav.addObject("boardList", boardList);
        mav.addObject("total", total);
        return mav;
    }
}
```

이제 list.html에서 `boardList`를 받아서 루프로 처리하도록 하자. 

```html 
{% raw %}
{% extends "layout/base" %}
{% block content %}
<script type="text/javascript" src="{{ request.contextPath }}/resources/js/board.js"></script>
<div>
    <form name="frm" method="get">
        <input type="hidden" name="pageNo" value="{{ search.pageNo }}">
    </form>
    Total : {{ search.totalCount }}
    <table class="table">
        <thead class="thead-dark">
        <tr>
            <th scope="col">#</th>
            <th scope="col">제목</th>
            <th scope="col">내용</th>
            <th scope="col">점수</th>
            <th scope="col">등록일</th>
        </tr>
        </thead>
        <tbody>
        {% for board in boardList %}
        <tr>
            <th scope="row">{{ board.id }}</th>
            <td>{{ board.title }}</td>
            <td>{{ board.content }}</td>
            <td>{{ board.score }}</td>
            <td>{{ board.regDate }}</td>
        </tr>
        {% else %}
        <tr>
            <td colspan="4">데이터가 없습니다.</td>
        </tr>
        {% endfor %}
        </tbody>
    </table>
    <!--   페이징 처리 -->
    {% include '/layout/paging' %}
</div>
{% endblock %}
{% endraw %}
```

맨 위 부분에 `extends`를 통해 앞서 만든 베이스 html를 불러오고 block content , endblock 안에 내용를 채우면 된다. `for` 구문이 굉장히 직관적이다. 게다가 `else` 부분도 제공하여 리스트가 없을 경우에도 바로 처리할 수 있다. 데이터 처리방법도 간단하고 페이징 처리같은 것은 별도로 html로 만든다음에 include하여 사용할 수 있다. 

### 3. Pebble Extension 사용하기 

Pebble 홈페이지에 문서를 보면 다양한 메소드를 제공한다. 그런데 그것만으로는 부족하여 개발자가 직접 구현한 메소드를 사용하는 경우가 있을 것이다. 이럴때 바로 pebble에서는 Extension을 통해서 확장가능하도록 해준다. 

위에 소스를 보면 `board.score` 부분은 String으로 저장되어 있다. pebble에서는 String을 Interger로 변환하는 부분이 없다. 그래서 어떤 수식계산을 하기 위해서는 별도의 처리가 필요하다. 그래서 샘플에서는 score점수가 90 이상이면 A, 80이상이면 B, 70이상이면 C를 주는 함수가 필요하다고 가정하고 작업을 해보자.  

적당한 곳에 PebbleCustomExtension 이라는 클래스를 만들었다.

```java 

import com.mitchellbosecke.pebble.extension.AbstractExtension;
import com.mitchellbosecke.pebble.extension.Function;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class PebbleCustomExtension extends AbstractExtension {
    @Override
    public Map<String, Function> getFunctions() {
        Map<String, Function> functions = new HashMap<String, Function>();
        functions.put("scoreEval", new ScoreEval());
        return functions;
    }

    /**
     * 점수계산 pebble 메소드를 만든다
     */
    class ScoreEval implements Function {
        @Override
        public List<String> getArgumentNames() {
            return null;
        }

        @Override
        public Object execute(Map<String, Object> args) {
            return null;
        }
    }
}

```

우리가 만든 PebbleCustomExtension은 반드시 pebble에서 제공하는 `AbstractExtension` 를 상속받아야 한다. 그리고 `getFunctions`를 오버라이딩 해야 한다. 그리고 실제 Function를 만들 클래스를 만드는데 간단히 내부클래스(ScoreEval)로 구현하자.  ScoreEval은 실제 점수를 처리해야하는 클래스이다. 이 클래서는 Pebble에서 제공하는 `Function` 인터페이스를 구현하고 `getArgumentNames`, `execute`를 구현해야 한다.  getArgumentNames는 우리가 만들 함수에 Argument를 정의하고 execute는 실제 처리하는 로직을 구현한다. 

```java 
    /**
     * 점수계산 pebble 클래스(Pebble에서는 함수)를 만든다
     */
    class ScoreEval implements Function {
        @Override
        public List<String> getArgumentNames() {
            List<String> names = new ArrayList<>();
            names.add("score");
            return names;
        }

        @Override
        public Object execute(Map<String, Object> args) {
            if (args.get("score") == null || args.get("score").equals("")) {
                return null;
            }
            int score = Integer.parseInt(args.get("score").toString());
            String result = "";
            if (score >= 90) {
                result = "A";
            } else if (score >= 80) {
                result = "B";
            } else if (score >= 70) {
                result = "C";
            } else {
                result = "D";
            }
            return result;
        }
    }
```

ScoreEval 이라는 클래스를 만들었다. ScoreEval은 PebbleCustomExtension 클래스에서  `functions.put("scoreEval", new ScoreEval());` 으로 등록되어 있다. 실제 html에서 사용하려면 scoreEval('90') 혹은 scoreEval(score='90') 등으로 사용할 수 있다. 우리가 getArgumentNames에 score라고 이름을 주었기 때문이다. 실제 로직처리는 execute에서 처리하도록 한다. 

이제 커스텀 함수도 만들었으니 html에서 사용하면 되는데 우리가 만든 클래스를 스프링에 등록시키고 pebble에도 등록해야 사용가능하다. 

```java 
    @Bean
    public Extension pebbleCustomExtension () {
        return new PebbleCustomExtension();
    }

    @Bean
    public PebbleEngine pebbleEngine() {
        return new PebbleEngine.Builder()
                .loader(this.templateLoader())
                .extension(springExtension())
                .extension(pebbleCustomExtension())
                .cacheActive(false)   //이 옵션을 안주면 html수정시 refresh해도 안됨,
                .build();
    }
```

`pebbleCustomExtension` 이라는 Bean을 생성하고 `PebbleEngine`에 extension에 추가해주면 된다. 그런 다음 html에 실제 사용이 가능해진다. 

```html 
{% raw %}       
{% for board in boardList %}
    <tr>
       <th scope="row">{{ board.id }}</th>
       <td>{{ board.title }}</td>
       <td>{{ board.content }}</td>
       <td>{{ scoreEval(board.score) }}</td>
       <td>{{ board.regDate }}</td>
    </tr>
{% else %}
    <tr>
       <td colspan="4">데이터가 없습니다.</td>
    </tr>
{% endfor %}
{% endraw %}
```

이런식으로 계속 Extension를 추가할 수 있다.  위에서 만든 PebbleCustomExtension 클래스에 계속 함수를 내부 클래스로 추가하면 별도로 스프링설정에 추가할 필요없이 바로 사용이 가능해진다. 

전체 소스는 아래에서 확인할 수 있다.

- [spring-pebble](https://github.com/yookeun/spring-pebble)