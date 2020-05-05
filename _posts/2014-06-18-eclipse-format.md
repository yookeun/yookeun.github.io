---
layout: post
title:  "eclipse 사용자 정의 format 설정"
date:   2014-06-18
categories: tools
---

이클립스에서 개발하다보면 개발자들끼리 줄바꿈, 탭, 그리고 {} 위치등이 틀려서 보기 불편할때가 있다.
이럴때는 하나의 format으로 통일하는 것이 좋다.  

기존에 `crtl+shift+f` 키를 누르면 기본 적용이 되지만 줄라인이 80자이면 줄바꿈이 되서 요새같은 와이드모니터에서는 불편하다.
그래서 그 부분만 160으로 변경하고 나머지는 그대로 사용하는 format를 만든다.

>Eclipse -> Window -> Preference 에서 다시 Java -> Code Style -> Formatter로 간다.

기본것은 수정은 안되기 때문에 `New`를 이용해서 새로 만든다.

![eclipse-format](/assets/images/eclipse-format.jpg)

`New`를 클릭하면 아래의 `New Profile`창에서 적당한 이름을 적는다.

![eclipse-format2](/assets/images/eclipse-format2.jpg)

그리고 나면 `Fomatter` 창에서 `Active-profile`에서 방금 저장한 것을 선택한 다음 `Edit`를 눌러준다.

![eclipse-format3](/assets/images/eclipse-format3.jpg)

그리고 `Line Wrapping`에 `General Settings`에서 `Maximum line width`를 `80`에서 `160`으로 변경하고 저장하면 된다.  

저장된 포멧을 프로젝트에 일괄저장하려면 프로젝트명에서 오른쪽 마우스 클릭`Source -> Format`를 선택하면 일괄 저장된다.
또한 저장된 Format스타일을 다른 개발자에게 주려면  

>Eclipse -> Window -> Preference -> Java -> Code Style -> Formatter

에서 `Export all`를 선택하고 xml로 저장하고 그 파일을 전달하고 전달받은 쪽에서 `import` 하면 된다.
