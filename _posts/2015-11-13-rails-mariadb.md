---
layout: post
title:  "Rails에서 Mariadb 연동"
date:   2015-11-13
categories: ruby
---

Rails 에서 mariadb를 연동한다.
환경은 아래와 같다.

설치|버전
---|---
Ubuntu|14.04
ruby | 2.2.3
rails | 4.2.4

rails에서 mariadb를 사용시에 gem mysql2 가 진행되지 않는다.
그래서 관련된 라이브러리를 먼저 설치해주어야 한다.

```bash
sudo apt-get install libmariadbclient-dev
```

gem mysql2를 설치하는데, mysql2의 최신버전이 rails 4.2.4에 호환성문제가 있다고 한다.
그러니 아래와 같은 버전으로 설치하면 문제가 해결된다.

```ruby
gem 'mysql2', '~> 0.3.18'
```

관련링크 :

- <http://stackoverflow.com/questions/16304311/use-mariadb-instead-of-mysql-in-my-rails-project>  
- <http://stackoverflow.com/questions/22932282/gemloaderror-for-mysql2-gem-but-its-already-in-gemfile>
