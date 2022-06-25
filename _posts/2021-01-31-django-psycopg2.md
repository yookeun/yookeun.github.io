---
layout: post
title:  "Django에 postgresql 설정"
date:   2021-01-31
categories: python
---

### 1. settings.py 데이터베이스 설정 

``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '디비명',
        'USER': '계정명',
        'PASSWORD': '패스워드',
        'HOST': '아이피',
        'PORT': '5432',
    }
}
```

### 2. psycopg2 대신 psycopg2-binary 설치

django랑 연동하려면 해당 모듈이 필요하다. 보통 psycopg2가 관련 모듈인데, pip으로 인스톨시 아래와 같은 에러가 나온 경우가 있다(맥 기준)

``` bash
(venv) (base) ➜  inoutbook pip install psycopg2
Collecting psycopg2
  Downloading psycopg2-2.8.6.tar.gz (383 kB)
     |████████████████████████████████| 383 kB 1.2 MB/s 
    ERROR: Command errored out with exit status 1:
     command: /Users/ykkim/workspace/inoutbook/venv/bin/python -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/var/folders/v5/2kbt25w951xc3x2hjtk51jq00000gn/T/pip-install-qpcd3pkb/psycopg2_3b8a5c8e8b4d453ebea1f36cad0b541b/setup.py'"'"'; __file__='"'"'/private/var/folders/v5/2kbt25w951xc3x2hjtk51jq00000gn/T/pip-install-qpcd3pkb/psycopg2_3b8a5c8e8b4d453ebea1f36cad0b541b/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /private/var/folders/v5/2kbt25w951xc3x2hjtk51jq00000gn/T/pip-pip-egg-info-7aon6rnz
         cwd: /private/var/folders/v5/2kbt25w951xc3x2hjtk51jq00000gn/T/pip-install-qpcd3pkb/psycopg2_3b8a5c8e8b4d453ebea1f36cad0b541b/
    Complete output (23 lines):
    running egg_info

```

위 내용으로 검색하면 path설정을 export하라고 했지만, 실제로 해보니 잘되지 않았다. psycopg2-binary 를 설치하니 잘 연동이 되었다.

``` python
pip install psycopg2-binary
```



