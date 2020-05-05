---
layout: post
title:  "Gradle Javascript, CSS Minify 하는 방법"
date:   2016-07-31
categories: java
---

Gradle에서 css, javascript를 Minify를 하기 위헤서는 해당 플러그인을 설치해야 한다.  
`eriwen` 이라는 개발자가 만든 플러그인이다. 

- [gradle-js-plugin](http://eriwen.github.com/gradle-js-plugin)
- [gradle-css-plugin](https://github.com/eriwen/gradle-css-plugin)

그런데 사이트에서는 모든 css/js 파일을 하나로 묶고, 하나의 all.min.css, all.min.js 등으로 만드는 가이드만 있다.
여기서는 각각의 css, js파일을 원본을 그대로 두고 같은 경로에 min파일을 생성하는 방법을 설명한다.  
즉, a1.css, b1.js 파일이 a1.min.css, b1.min.js 파일로 만드는 방법을 설명한다. 

먼저 build.gradle 파일에 플러그인을 세팅하자. 가장 상단에 작성한다. Gradle 버전에 맞게 가급적 최신버전을 사용하는 것이 좋다.

```bash
plugins {
    id "com.eriwen.gradle.js" version "2.14.1"
    id "com.eriwen.gradle.css" version "2.14.0"
}
apply plugin: 'css'
apply plugin: 'js'
```

다음은 Minify 세팅을 하자.  
css파일의 경로가 `src/main/webapp/WEB-INF/public/css` 에 있고 
js파일의 경로가 `src/main/webapp/WEB-INF/public/script`에 있다고 하면 우리는 모든 js, css파일을 처리해야 한다.

```bash
css.source {
    custom {
        css {
            srcDir "src/main/webapp/WEB-INF/public/css"
            include "*.css"
            exclude "*.min.css"
        }
    }
}

javascript.source {
    custom {
        js {
            srcDir "src/main/webapp/WEB-INF/public/script"
            include "*.js"
            exclude "*.min.js"
        }
    }
}
```

`include`에서는 모든 파일이므로 *.css, *.js 로 표시하고 이미 Minify된 파일을 제외해야하므로 `exclude`에서 제외처리를 해준다.
`css.source.custom.css`, `javascript.source.custom.js` 등에서 `custom`은 용도에 맞게 사용하면된다. 만약 배포할때 개발서버는 그대로 두고 서버에만 Minify한다면 custom을 dev, prod 등으로 별도로 만들 수 있다. 여기서는 그냥 로컬이나 개발에서도 같이 처리하기 위헤서 custom이라는 것을 만들어 사용했다. 이름은 알아서 지으면 된다. 

이제 위에서 설정한 값으로 처리하는 부분을 만들어보자 

```bash
css.source.custom.css.files.eachWithIndex { cssFile, idx ->
    tasks.create(name: "minifyCss${idx}", type: com.eriwen.gradle.css.tasks.MinifyCssTask) {
        source = cssFile
        dest = cssFile.getAbsolutePath().replace('.css','.min.css')
        yuicompressor {
            lineBreakPos = -1
        }
    }
}

javascript.source.custom.js.files.eachWithIndex { jsFile, idx ->
    tasks.create(name: "minifyJs${idx}", type: com.eriwen.gradle.js.tasks.MinifyJsTask) {
        source = jsFile
        dest = jsFile.getAbsolutePath().replace('.js','.min.js')
        yuicompressor {
            lineBreakPos = -1
        }
    }
}
```

위 처리는 각각 정의된 디렉토리에서 해당 파일들을 읽어서 css파일들일 경우는 플러그인에서 제공하는 `MinifyCssTask`를 이용하고, javascript파일을 경우에는 `MinifyJsTask`가 실행되어 각각 파일을 Minify한다는 뜻이다.`{idx}`가 해당파일의 인덱스라고 보면 된다. 
그리고 `dest`에 Minify한 파일을 넣어주는데, 이때 주의할 점은 
dest에 cssFile, jsFile등으로 source와 같이 처리하면 원본파일이 Minify 되어 버린다. 그러므로 `.css`파일은 `min.css`로 하고 `.js`파일은 `min.js`로 생성하도록 하기 위해서 `getAbsolutePath().replace()`로 처리해준다.  
압축툴은 `yuicompressor`를 사용한다. 이것은 옵션이므로 바꾸어도 된다. 자세한 것은 플러그인 사이트를 보도록 하자. 

이제 해당 작업을 실행할 `Task`를 만들자.

```bash
task allMinifyCss(dependsOn: tasks.matching { Task task ->
    task.name.startsWith("minifyCss")
})

task allMinifyJs(dependsOn: tasks.matching { Task task ->
    task.name.startsWith("minifyJs")
})
```

`gradle allMinifyCss`, `gradle allMinifyJs` 로 처리하면 각각의 테스크가 진행되어진다. 
이클립스나 인텔리에서 해당 태스크를 실행하면 된다. 
그런데 매번 각각 해당 Task를 실행면 번거로우므로 하나로 묶자.

```bash
task minify(dependsOn: [allMinifyCss, allMinifyJs])
```

`gradle minify`를 처리하면 일괄적으로 모두 처리된다. 
빌드할때 `minify` 옵션만 넣어주면 완료된다. 


아래는 최종 전체 build.gradle 전체 소스이다. 

```bash
plugins {
    id "com.eriwen.gradle.js" version "2.14.1"
    id "com.eriwen.gradle.css" version "2.14.0"
}
apply plugin: 'css'
apply plugin: 'js'

css.source {
    custom {
        css {
            srcDir "src/main/webapp/WEB-INF/public/css"
            include "*.css"
            exclude "*.min.css"
        }
    }
}

javascript.source {
    custom {
        js {
            srcDir "src/main/webapp/WEB-INF/public/script"
            include "*.js"
            exclude "*.min.js"
        }
    }
}

css.source.custom.css.files.eachWithIndex { cssFile, idx ->
    tasks.create(name: "minifyCss${idx}", type: com.eriwen.gradle.css.tasks.MinifyCssTask) {
        source = cssFile
        dest = cssFile.getAbsolutePath().replace('.css','.min.css')
        yuicompressor {
            lineBreakPos = -1
        }
    }
}

javascript.source.custom.js.files.eachWithIndex { jsFile, idx ->
    tasks.create(name: "minifyJs${idx}", type: com.eriwen.gradle.js.tasks.MinifyJsTask) {
        source = jsFile
        dest = jsFile.getAbsolutePath().replace('.js','.min.js')
        yuicompressor {
            lineBreakPos = -1
        }
    }
}

task allMinifyCss(dependsOn: tasks.matching { Task task ->
    task.name.startsWith("minifyCss")
})

task allMinifyJs(dependsOn: tasks.matching { Task task ->
    task.name.startsWith("minifyJs")
})

task minify(dependsOn: [allMinifyCss, allMinifyJs])
```




