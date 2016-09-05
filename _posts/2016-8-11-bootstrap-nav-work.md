---
layout: post
title: bootstrap tab과 pill을 data 속성으로 동작시키기
categories: bootstrap
---

[bootstrap navs](http://getbootstrap.com/components/#nav)는 크게 두 부분으로 나뉘어진다. 메뉴를 보여주는 __.nav__와 각 메뉴의 내용인 __.tab-pane__을 품고있는 __.tab-content__로 나뉘어진다. 각 메뉴를 클릭하면 해당 메뉴는 target하고 있는 __.tab-pane__을 보여주고 다른 __.tab-pane__은 감춘다. 간단한 소스로 보여주면 아래와 같다.

```
<ul class="nav nav-pills">
  <li role="presentation" class="active"><a data-toggle="pill" data-target="#first">1번 필</a></li>
  <li role="presentation"><a data-toggle="pill" data-target="#second">2번 필</a></li>
  <li role="presentation"><a data-toggle="pill" data-target="#third">3번 필</a></li>
</ul>

<div class="tab-content">
  <div class="tab-pane active" id="first">
    1번 필
  </div>
  <div class="tab-pane" id="second">
    2번 필
  </div>
  <div class="tab-pane" id="third">
    3번 필
  </div>
</div>
```

__data-target__은 id 셀렉터든 클래스 셀렉터든 상관 없으며, 여러개를 target할 수 있다. 또한 __data-target__속성 대신 __href__속성으로도 target할 수 있다. 하지만 bootstrap 가이드에서는 __href__와 __data-target__속성을 같이 지정했는데, 아마도 __data__속성을 지원하지 않는 몇몇 브라우저를 대비한 것이 아닌가 추측된다. 

___
* related links
  * [bootstrap nav component 가이드 - nav 메뉴의 외양 설정](http://getbootstrap.com/components/#nav)
  * [bootstrap nav js 가이드 - js 동작 및 이벤트](http://getbootstrap.com/components/#nav)
  * [data 속성으로 작동시키기에 대한 SO 문서](http://stackoverflow.com/questions/19225968/bootstrap-tab-is-not-working-when-tab-with-data-target-instead-of-href)