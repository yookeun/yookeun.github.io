---
layout: post
title:  "Redmine과 Github연동"
date:   2015-01-26
categories: tools
---

Redmine 에서 Github소스와 연동하면 등록된 일감과 연동된 코드를 보면서 업무를 관리할 수 있다.  

먼저 Redmine에서 제공하는 github용 플러그인을 설치해야 한다. 물론 Redmine에 설치된 서버에 git이 설치되어 있어야 한다
Redmine에서는 등록된 git를 통해서 github사이트에서 소스가 변경시 hook방식으로 Redmine에 지정된 경로로 소스를 갱신해준다.
그러므로 Redmine서버에서 먼저 반드시 git설치하고 작업디렉토리를 만들고 git clone를 통해서 프로젝트를 한번 가져오는 것이 좋다.  

이때, github에 <[ssh](https://help.github.com/articles/generating-ssh-keys/)>
등으로 연결한 상태면 별도의 로그인처리 없이 작업을 진행할 수 있다.


아래사이트로 가서 플러그인을 다운받는다.  
https://github.com/koppen/redmine_github_hook

다운 받은 `redmine_github_hook-master.zip`파일을 설치된 레드마인 플러그인 경로 푼다.

```bash
cd /usr/share/redmine/lib/plugins
```

이때 폴더명을 redmine_github_hook으로 변경한다. 그리고 나서 아래 명렁어를 처리하면 플러그인이 설치된다.

```bash
rake redmine:plugins:migrate RAILS_ENV=production
```

이제 톰켓를 재시작한다. 그리고 나서 Redmine관리자로 로그인한후에 관리에 보면 아래와 같이 플러그인이 정상등록되어 있어야 한다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/redmine-github1.jpg" style="width:100%">
</div>

그리고나서 `관리 -> 설정 ->저장소` 탭을 선택하면 지원할 SCM이 나온다.  
Git를 선택하고 아래 정보를 입력한다. 이때 API키를 생성하고 이 키를 github에 등록해주어야 하므로 잘 복사해둔다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/redmine-github2.jpg" style="width:100%">
</div>

키생성을 클릭하여 API키를 생성하고 잘 복사해둔다. (Github에 등록할 Api 키이다)  
Redmine에 해당 프로젝트에 가서 `설정->저장소`로 이동후 저장소 추가를 통해 git경로등을 설정한다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/redmine-github3.jpg" style="width:100%">
</div>

식별자의 git프로젝트명을 등록하고 저장소 경로의 git으로 clone받는 경로를 입력하자.  
예)/home/test/workspace/test-project/.git  

저장을 누르면 Redmine에서 설정은 끝났다. 이제 Github사이트에서 설정해주자. Github에서 해당 프로젝트의 `Settings`로 들어간다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/redmine-github4.jpg" style="width:100%">
</div>

거기서 `Webhooks &Service`에서 `Service`부분의 `Add Service`를 선택하고 Red를 치면 Redmine이 나온다. 그대로 엔터를 쳐서 설치를 한다.
그러면 설치방법에 대한 문서가 나오고 하단에 보면 입력란이 나온다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/redmine-github5.jpg" style="width:100%">
</div>

`Address`부분에 Redmine의 플러그인의 주소를 입력하면 된다.  
`http://redmine이 설치된 URL/redmine/github_hook`

Project에 프로젝트명을 입력한다.  
예) testProject  

`Api Key`에 아까 Redmine에서 생성된 키를 입력하고 `Add Service`를 입력하면 완료된다. 이렇게 완료가 되면 이제부터 Github에 푸시된 소스는 Redmine에서 확인할 수 있다.
단, 이때 해당 일감과 연결된 소스라는 것을 표시하기 위해서 커밋메시지에

`#레드마인 일감번호`를 등록하고 커밋메시지를 입력하면 일감과 연동된 소스를 확인할 수 있다.  
예) git commit -m "#123 게시판의 조회기능 추가"  

최종적으로 Redmine에 등록된 프로젝트에의 저장소메뉴를 클릭하면 Git이력을 확인할 수 있다.  
이제부터 코드리뷰등은 Redmine을 통해 편리하게 진행할 수 있게 된다.
