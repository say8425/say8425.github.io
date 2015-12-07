---
layout: post
title: rails가 precompile assets를 읽지 못할때
---

1. 이름을 바꾼다. 가장 쉽고 가장 무책임한 방법이다.
1. `rake assets:precompile`을 run 한다. 보통 이 선에서 해결된다.
1. 안되면 `public/assets`를 날려버린다.
1. 그래도 안되면 `tmp/cache/assets`도 날려버린다.

---
* Related Links
 * [4번 방법을 알려준 Stack Overflow](http://stackoverflow.com/a/11107762/3910390)