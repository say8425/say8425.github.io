---
layout: post
title: rails가 precompile assets를 읽지 못할때
---

1. `rake assets:precompile` run 한다. 보통 이 선에서 작동 한다.
2. 안되면 `public/assets`를 날려버린다.
3. 그래도 안되면 `tmp/cache/assets`도 날려버린다.

---
* Related Links
 * [3번 방법을 알려준 Stack Overflow](http://stackoverflow.com/a/11107762/3910390)