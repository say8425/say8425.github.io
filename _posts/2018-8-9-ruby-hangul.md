---
layout: post
title: Ruby Console 에서 한글 입력하기
categories: ruby
---

irb나 pry에서 한글 입력을 시도하면, 한글이 깨져버려서 사용할 수 없다. 이걸 해결하려면 readline을 사용해서 ruby를 다시 컴파일 해야한다. 본 포스트에서는 macOS 기준으로 설명하겠다. 참고로 이 글은 [seapy님 포스트](https://blog.iamseapy.com/archives/177)를 참고해서 작성되었다.

# readline 설치

```sh
$ brew install readline
```

# readline을 사용해서 ruby 재설치

```sh
$ RUBY_CONFIGURE_OPTS=--with-readline-dir="$(brew --prefix readline)" rbenv install 루비 버전
```

이미 ruby가 설치된 상태에서 `rbenv install 루비 버전`을 시도하면, 이미 설치되었는데, 계속 할거냐고 물을 것이다. 가볍게 `y` 해주자.

## 명령어가 너무 길다!

```sh
$ git clone git://github.com/tpope/rbenv-readline.git ~/.rbenv/plugins/rbenv-readline
$ rbenv install 루비 버전
``` 

만약 매번 기억도 잘안나는 readline path 지정이 귀찮다면, [rbenv-readline 플러그인](https://github.com/tpope/rbenv-readline)을 설치하고 이 귀찮음으로부터 해방될 수 있다.

# 이전에 설치된 gem 을 복구해주자.

```sh
$ gem pristine --all
```

---
* Related Links
  * [seapy님이 올리신 글](https://blog.iamseapy.com/archives/177)
