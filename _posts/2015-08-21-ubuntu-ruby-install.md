---
layout: post
title:  "Ubuntu에서 Ruby 설치하기"
date:   2015-08-21
categories: ruby
---

우분투(14.04)에서 루비(2.2.3)를 설치해보자.  

루비를 설치하는 방법은 두 rvm과 rbenv로 설치하는 두 가지 방법이 있다. 나중에 레일즈도 설치해야 하므로 rbenv로 설치를 진행한다.
rbenv가 rvm보다 사용법이 간단하다고 한다.

먼저, 사전에 git등은 설치되어 있어야 한다.  그리고 아래 라이브러리를 설치한다.

```bash
sudo apt-get install curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev
```

 그리고 github에서 rbenv를 클론한다. 자신의 홈디렉토리에서 작업을 수행한다.

```bash
git clone git://github.com/sstephenson/rbenv.git .rbenv
```

이렇게 하면 .rbenv라는 디레토리가 생성되고 그 안에 소스가 설치된 것을 볼 수 있다.
다음은 환경변수를 등록해준다.

`.bash_profile` 에 다음을 추가한다.

```bash
[ -f "$HOME/.bashrc" ] && source "$HOME/.bashrc"
```

그리고 나서 `.bashrc` 을 열고 아래 부분을 입력한다.

```bash
export RBENV_ROOT="${HOME}/.rbenv"
if [ -d "${RBENV_ROOT}" ]; then
  export PATH="${RBENV_ROOT}/bin:${PATH}"
  eval "$(rbenv init -)"
fi
```

rbenv를 이용해서 ruby를 설치하려면 ruby-build가 있어야 한다. `.rbenv/plugins` 폴더를 만들고 그안에 플러그인을 github로 받자.

```bash
mkdir -p ~/.rbenv/plugins
cd ~/.rbenv/plugins
git clone git://github.com/sstephenson/ruby-build.git
```

자. 이제 루비를 설치하자. 2.2.3 버전을 설치한다.

```bash
rbenv install 2.2.3
```

생각보다 시간이 걸릴 수 있다. 차분이 기다리자.
설치가 완료되면 rehash를 통해서 rbenv를 재설정하고 전역옵션으로 시스템이 설정된 루비버전을 사용하게 한다.
그리고 나서 ruby -v로 설치버전을 확인하여 정상으로 나오면 완료된 것이다.

```bash
rbenv rehash
rbenv global 2.2.3
ruby -v
```

그리고 루비젬을 관리하기 위해 bunlder를 설치한다.

```bash
gem install bundler
rbenv rehash
```

그런데 만약 `sublime text3`을 통해서 ruby를 빌드하면 ruby를 찾을 수 없다는 에러가 표시되면서 빌드가 되지 못한다.
서브라임에서는 기본적으로 /usr/bin에 ruby가 있다고 판단하고 빌드를 하기 때문이다.
따라서 우리는 ln -s를 이용해서 usr/bin에 링크파일을 만들어줘야 한다.

먼저 현 시스템에서 ruby 설치경로를 `which ruby` 를 통해서 찾고

다음과 같이 링크를 만들어준다.

```bash
sudo ln -s /home/사용자홈디렉토리/.rbenv/shims/ruby /usr/bin/ruby
```
이렇게 하면 서브라임에서도 루비로 빌드가 가능해진다.
