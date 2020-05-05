---
layout: post
title:  "SSH 인증 여러개 사용하기"
date:   2016-06-26
categories: tools
---

Github나 Gitlab(or Bitbucket) 등 여러개의 GIT 서비스를 사용한다면 ssh를 각각 해두고 싶을 때가 있다.  
개인프로젝트는 Github, 회사는 Gitlab등을 사용하는 경우가 그렇다.

이럴때는 ssh의 인증을 여러개 만들 수 있다. 여기서는 두개만 생성한다고 가정한다.

### 1. Github 개인계정을 만들 경우

```bash
ssh-keygen -t rsa -b 4096 -C "example@home.com"
```

위에 처럼 생성되면 아래의 같은 키이름을 지정하는 문구가 나온다. 여기서 개인용을 지정하자.

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/kim/.ssh/id_rsa):
```

개인용은 그냥 디폴트은 `id_rsa`를 사용해도 되므로 엔터, 엔터를 계속치면 된다.

### 2. Gitlab(or Bitbucket) 업무용 계정을 만들 경우.

```bash
ssh-keygen -t rsa -b 4096 -C "example@company.com"
```

회사메일로 적고 위에 처럼 key저장을 물을 때는 그냥 엔터를 치지 말고 고유의 이름을 정하면 된다.
이때, 전체 경로를 적어줘야 그 위치에 저장된다.

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/kim/.ssh/id_rsa):
```

> /Users/kim/.ssh/company_id_rsa

그리고 나서 엔터를 치면 각각의 `id_rsa, id_rsa.pub, company_id_rsa, company_id_rsa.pub` 파일이 보여진다.

### 3. config 파일을 만든다.

각각의 ssh키로 접근하기 위해서 config 파일을 직접 만들어야 한다. ~/.ssh 안에 만들어야 한다.

```bash
Host github.com
	User kim
	IdentityFile ~/.ssh/id_rsa
Host gitlab.com
	User kim2
	IdentityFile ~/.ssh/company_id_rsa
```

`User`에는 git 서비스 계정을 각각 입력하면 된다.
이렇게 하면 알아서 호스트명에 따라 알아서 ssh키로 접근하게 된다.
