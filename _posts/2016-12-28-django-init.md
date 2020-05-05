---
layout: post
title:  "Django migrations 초기화"
date:   2016-12-28
categories: python
---

장고에서 마이그레이션을 초기화하는 방법 

1. 해당 app의 migrations 폴더에서 `__init__.py` 만 남겨두고 나머지 파일들 삭제 
2. DB에서 해당 테이블을 Drop 한다.
3. DELETE FROM django_migrations WHERE app = 'your_app_name'
4. python manage.py makemigrations
5. python manage.py migrate


