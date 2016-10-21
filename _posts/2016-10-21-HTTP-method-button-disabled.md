---
layout: post
title: HTTP method 전달과 disabled 동시에 하기
categories: rails
---

HTTP method로 전달 하되, 버튼을 클릭시 `disabled`가 되게 만들 일이 생겼다. 그래서 우선 `button tag`로 할 수 있는 방법을 찾아봤다.

```html
<button name="button" onclick="window.location='http://www.example.com';" class="btn btn-danger" data-disable-with="삭제중">삭제</button>
```

`onclick`을 이용해서 원하는 링크 주소를 입력해주면 된다. 하지만 method를 정해주는 방법을 찾을 수 없었다. 그래서 `a tag`를 시도해봤다.

```html
<a class="btn btn-danger" data-method="delete" data-disable-with="삭제중" href="http://localhost:3000/systems/delete?id=4&amp;kind=accessors">삭제</a>
```

`a tag` 답게 코드도 훨씬 깔끔해졌다. 하지만 치명적인 문제가 있는데 `a tag`는 disabled가 안된다. 이 때문에 disabled 하는 방법을 찾아보았는데, 클릭시 [return false를 내놓는 방법](http://stackoverflow.com/a/13955726/3910390)과 [href 요소를 제거하는 방법](http://stackoverflow.com/a/13955716/3910390)이 있지만 펭귄에게는 두 방법이 모두가 안되었다. 그래서 클릭시 [disabled style을 입히는 방법](http://stackoverflow.com/a/27057625/3910390)을 시도했다.

```js
$('.confirm').click(function() {
  $(this).addClass('disabled');
});
```

```css
.disabled {
  opacity: 0.65;
  cursor: not-allowed;
  pointer-events: none;
  box-shadow: 0;
}
```

부트스트랩 기본 disabled button 과 통일성을 맞추려고 `.btn.disabled`을 참고해서 만들었다. 주의할 점은 [pointer-events는 IE 11이상에서만 지원](http://caniuse.com/#feat=pointer-events)한다. 그래서 만약 낮은 버전의 IE도 지원해야한다면 다른 방법도 참고해야한다.

---
* Related Links
  * [a tag를 disabled 하기 SO문서](http://stackoverflow.com/questions/13955667/disabled-href-tag)