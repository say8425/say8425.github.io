---
layout: post
title: Touch Id로 sudo 인증하기
categories: terminal
tags: sudo touchid terminal
---

![Sudo Touch Id]({{ site.baseurl }}/images/2019-9-24-touch-id-sudo/touch-id-sudo.png)

비밀번호로 하던 sudo 인증을 [Touch Id](https://support.apple.com/en-us/HT207054)로 편하게 하자.

## pam_reattach 설치

```shell
brew install fabianishere/personal/pam_reattach
```

[homebrew](https://brew.sh)로 [pam_reattach](https://github.com/fabianishere/pam_reattach) 설치한다.

## sudo 파일 수정

`/etc/pam.d/sudo` 파일을 sudo를 사용하여 선호하는 에디터로 연다.

```shell
auth     optional     pam_reattach.so
auth     sufficient   pam_tid.so
...
```

파일의 맨 처음에 상기 예시와 같은 코드 2줄을 쓴다. 중요한 sudo 파일이므로 다른 줄은 절대로 건들지 말자.
저장할때는 평범하게 `wq`로는 안된다. `wq!`로 강제저장 한다.

## Etc

macOS 업데이트를 하였다면, 이 작업을 다시 해야 할 수도 있다.

---

* Related Links
  * [pam_reattach](https://github.com/fabianishere/pam_reattach)
  * [Birkhoff Lee 님이 작성하신 포스트](https://blog.birkhoff.me/make-sudo-authenticate-with-touch-id-in-a-tmux/)
