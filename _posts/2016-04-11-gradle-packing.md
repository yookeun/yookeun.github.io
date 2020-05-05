---
layout: post
title:  "Gradle에서 서버별 패키징 하기"
date:   2016-04-11
categories: java
---

Gradle에서 패키징하는 방법을 알아본다.  

본 문서는 <[Maven에서 서버별 패키징하기](http://yookeun.github.io/java/2015/07/20/maven-pacaking/)>의
Gradle 버전용 문서이다.
따라서 디렉토리 설정은 같고 다만, Maven은 pom.xml에서 설정하지만 Gradle은 build.gradle에서 설정을 해주는 부분만 다르기 때문에
`build.gradle`파일에 아래내용만 추가해주면 된다.


```bash
//디폴트 패키징
final String DEFAULT_PROFILE = 'local'

allprojects {
	if (!project.hasProperty('profile') || !profile) {
		ext.profile = DEFAULT_PROFILE
	}

	sourceSets {
		main {
			resources {
				srcDir "src/main/resources-${profile}"
			}
		}
	}
}
```

그리고 빌드할 때 옵션(Pprofile)만 달라주면 된다.

> gradle -Pprofile=local (or real, demo)
