---
layout: post
title: Ruby Console 에서 한글 입력하기
categories: ruby
---

macOS 기준으로 설명하겠다.

1. readline 설치

```sh
$ brew install readline
```

2. readline을 사용해서 ruby 재설치
```sh
$ RUBY_CONFIGURE_OPTS=--with-readline-dir="$(brew --prefix readline)" rbenv install 루비 버전
```

이미 ruby가 설치된 상태에서 `rbenv install 루비 버전`을 하면, 재설치 할거냐고 물어볼 것이다. 가볍게 `yes` 해주자.

```sh
$ git clone git://github.com/tpope/rbenv-readline.git ~/.rbenv/plugins/rbenv-readline
$ rbenv install 루비 버전
``` 

만약 매번 기억도 잘안나는 readline path 지정이 귀찮다면, [rbenv-readline 플러그인](https://github.com/tpope/rbenv-readline)을 설치하고 이 귀찮음으로부터 해방될 수 있다.

3. 이전에 설치된 gem 을 복구해주자.

```sh
$ gem pristine --all
```
