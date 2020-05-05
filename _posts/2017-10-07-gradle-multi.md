---
layout: post
title: "Gradle 에서 Multi 프로젝트 만들기"
date: 2017-10-07
categories: java
---

gradle 로 멀티프로젝트를 구성해보자. 여기서 멀티프로젝트란 프로젝트를 구성시 web, app 등으로 용도가 다른 프로젝트를 생성할 경우를 말한다. 이때 바라보는 database가 같다면 아마도 상당수 많은 부분이 공통적으로 겹치게 될 것이다.  유지보수 차원에서 공통인 부분이 각각 존재한다면 같은 작업을 여러번 해야 한다는 뜻이다. 이러한 중복을 방지하고 또 빌드를 용이하기 위해서 멀티프로젝트를 구성하고자 한다. 

### 1. 루트 프로젝트 구성

우리의 프로젝트가 크게 web부분과 app부분으로 나누어졌다고 하면 공통되는 부분을 별도로 빼서 관리하고 각각 프로젝트가 참조하면 된다. 그 공통인 부분은 core라고 부르기로 하자. 

따라서 우리는 여기서 샘플로 크게 4부분으로 모듈을 나눌 것이다.

| 프로젝트명             | 용도                            |
| ----------------- | ----------------------------- |
| gradle-multi      | root 용 프로젝트                   |
| gradle-multi-core | 공통클래스관리 (dao, dto)            |
| gradle-multi-web  | Web Application (Sprint boot) |
| gradle-multi-app  | App                           |

프로젝트명을 gradle-multi* 로 시작하는 것은 이클립스에 Package explorer에 순서대로 보이기 위함이다. 실제로는 같은 폴더 안에 서브프로젝트로 구성된다.  이클립스에서 먼저  gradle 프로젝트를 생성하고 이름을 gralde-mulit로 한다. 이것은 루트 프로젝트이고 실제 소스가 있는 것이 아니고 통합적으로 빌드를 구성하기 위해 사용되는 프로젝트이다. 따라서 초기에 만들어진 src/main/java, src/main/resource, src/test/java 같은 패키지는 필요치가 않다. 모두 삭제해주고 리프레쉬 해주면서 다음과 같이 남겨주자.

![multi1](/assets/images/multi1.jpg)

다음은 `settings.gradle`을 만들어 다음과 같이 작성한다.

``` groovy
rootProject.name = 'gradle-multi'
include 'gradle-multi-core'
include 'gradle-multi-app'
include 'gradle-multi-web'
```

보는 바와 같이 루트프로제트명을 지정하고 서브프로젝트를 include에 추가하면 된다. 이렇게 간단하게 멀티프로젝트를 구성할 수 있다. 그런데 이렇게만 하면 서브프로젝트들이 폴더만 생성되므로 gradle 형태의 소스구조를 가려면 `build.gradle`을 다음과 같이 구성해야 한다. 

``` groovy
//=================================================================//
// gradle-multi ROOT
//=================================================================//
buildscript {
	ext {
		springBootVersion = '1.5.6.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
		classpath('io.spring.gradle:dependency-management-plugin:1.0.0.RELEASE')
	}
}

subprojects {
	apply plugin: 'java'
	apply plugin: 'eclipse'
	apply plugin: 'idea'
	apply plugin: 'org.springframework.boot'
	
	sourceCompatibility = 1.8
	repositories {
		mavenLocal()
		mavenCentral()
	}
	
	task initSourceFolders {
	    sourceSets*.java.srcDirs*.each {
	        if( !it.exists() ) {
	            it.mkdirs()
	        }
	    }
	 
	    sourceSets*.resources.srcDirs*.each {
	        if( !it.exists() ) {
	            it.mkdirs()
	        }
	    }
	}

	dependencies {
		compileOnly('org.projectlombok:lombok')
		testCompile("org.springframework.boot:spring-boot-starter-test")
	}
	
}
```

기본적으로 spring boot를 뼈대로 한다. 그리고 `subproject`에 공통적으로 적용할 plugin, repositories을 설정한다. 그리고 나서 task를 통해 서브프로젝트들이 gradle (maven) 구조의 디렉토리를 구성하게 한다. 마지막으로 공통적으로 적용할 dependencies를 정해주면 된다.  아래는 최종적으로 구성된 멀티프로젝트의 구조다. 

![multi2](/assets/images/multi2.jpg)

### 2. 공통(core) 프로젝트 구성

이제 공통(core)부분을 처리하는 프로젝트를 만들어보자. core에는 직접 db와 핸들링하는 dao부분이랑 객체 엔티티가 존재하게  한다. 즉 database 관리하는 부분을 총괄하게 한다. 이렇게 하면 공통적으로 사용할 수 있다.  먼저 build.gradle 부터 설정하자. 

``` groovy
//=================================================================//
// gradle-multi-core
//=================================================================//

bootRepackage {
	enabled = false
}

dependencies {	
	compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.1')
	runtime('mysql:mysql-connector-java')	
	compile 'com.zaxxer:HikariCP:2.6.2'
}
```

core에 dependencies에 사용할 라이브러리를 등록하자. 주로 db와 관련된  mybatis, mysql-conneter, hikari 을 등록한다. 그리고 core는 스프링 부트의 메인클래스가 없기 때문에 반드시 `bootRepackage`는 `false`로 설정해주어야 한다. 


그리고 나서 mybatis config 설정 클래스를 만들고 mapper 폴더를 src/main/resoucres 에 만들어 query xml은 거기서 관리하도록 한다. 이제 boardDto 라는 게시판 테이블에 연관된 엔티티를 만들고 그것을 처리할 dao를 만들자.

``` java 
@Repository
public class BoardDao {
	@Autowired
	private SqlSession sqlSession;
	
	public List<BoardDto> selectList() {
		return sqlSession.selectList("boardMapper.selectList");
	}
}
```

이렇게 해서 core 프로젝트는 간단히 완료가 되었다. 

### 3. 서브 프로젝트 구성 

core 프로젝트를 사용하는 모든 프로젝트의 build.gradle에는 다음과 같이 `compile project` 가 설정되어야 한다. 그래야만 core 프로젝트에 접근해서 사용할 수 있다(물론 이 부분은 루트프로젝트에서 설정할 수도 있다. 설정방법이 다르니 유의할것)

``` java 
//=================================================================//
// gradle-multi-web
//=================================================================//
apply plugin: 'war'

dependencies {
	compile project(':gradle-multi-core')

	providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
	compile 'javax.servlet:jstl:1.2'
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.springframework.boot:spring-boot-starter-security')		
	testCompile('org.springframework.security:spring-security-test')
}
```

이 프로젝트에는 web에 관련된 dependencies만 구성하면 된다. 이렇게 서브프로젝트에서 필요로하는 부분만 등록해주면 된다. 스프링부트의 메인클래스를 보도록 하자. 

``` java 

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan("com.example")
public class GradleMultiWebMain {
	public static void main(String[] args) {
		SpringApplication.run(GradleMultiWebMain.class, args);
	}
}
```

`@SpringBootApplication` 으로 설정한다. 그런데 중요한 것은 `@ComponentScan` 부분이다. 사실 `@SpringBootApplication` 으로 설정하면 알아서 해당 패키지 아래에 있는 클래스들(@Service, @Component 등)은 component scan 이 이루어진다. 그런데 멀티프로젝트에서 보통 패키지명을 다르게 지정되므로 (com.example.core, com.example.web, com.example.app) 최상위 패키지명을 지정해주어야 autowired 할 수 있다. 

이제 core dao를 접근하는 service를 만들자.

``` java 
package com.example.web.board.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.core.board.dao.BoardDao;
import com.example.core.board.dto.BoardDto;

@Service
public class BoardService {

	@Autowired
	private BoardDao boardDao;
	
	public List<BoardDto> selectList() {
		return boardDao.selectList();
	}
}

```

이와 같이 각 서브프로젝트에서는 코어프로젝트인 com.example.core.board,dao, dto 등을 접근해서 클래스를 사용할 수 있다.  별도의 app프로젝트(gradle-mulit-app)도 같은식으로 개발하면 된다. 그런데 실제 war나 jar로 빌드되면 core부분은 어떻게 같이 패키징될까? 실제로 빌드된 파일을 풀어보면 우리가 만든 core 부분은 WEB-INF/lib 안에 gradle-multi-core.jar(코어 프로젝트명) 로 묶여져 있음을 확인할 수 있다. 

이렇게 함으로써 각각 완벽하게 빌드되고 배포할 수 있게 된다. 

전체소스는 아래에서 확인할 수 있다. 

- [gradle-multi](https://github.com/yookeun/gradle-multi)

