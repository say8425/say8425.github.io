---
layout: post
title: reactjs 공부하기 - 01
categories: react
---

전 포스트에서 간단하게 `Hello World`를 출력했다. 본래는 이번 포스트에서 컴포넌트를 다루기로 했지만, 그전에 전 포스트에서 작성한 코드를 기반으로 몇 가지를 추가로 포스트한다.

### 구조분해 할당

들어가기 앞서 전 포스트에서 작성한 코드의 앞줄을 잠깐만 보자.

```
import React from 'react';
import ReactDOM from 'react-dom';

class Hello extends React.Component {
  
// 중략
```

보이는 대로 소스에서는 `React.Component`를 사용하였다. 그리고 이를 위해서 `import React from 'react'`를 사용해서 react라는 모듈을 통째로 import 해왔다. 하지만 지금은 이 모듈이 전부 필요하지 않고, `Component`만 필요하다. 그렇다면 `Component`만 접근할 수 없을까? 가능하다.

```
import React, { Component} from 'react';
import ReactDOM from 'react-dom';

class Hello extends Component {

// 중략
```

`import React, { Component} from 'react'`와 같이 코드를 작성해서 react 모듈 내부의 `Component`를 직접 접근할 수 있다. 이렇게 하면 사용할때도 `React.Component`라고 쓸 필요 없이, 당장 `Componet`로 접근 할 수 있다. 책에서는 이것을 **구조분해 할당 destrucutring assignemt** 이라고 하지만, MDN에서는 [비구조화 할당](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)이라고 번역했다. 참고로 ES6에서 포함된 개념이다. 사실 이로 인한 생산성 혹은 퍼포먼스 효과는 얼마나 큰지 당장은 모르겠지만, 책에서는 생산성에 도움이 될 것이다는 릐앙스로 적혀있었다. 정말 그런지는 사용해보면서 판단해야 할 거 같다.

### 동적값

JSX에서 `{}`로 감싸진 구문은 동적으로 표현된다. 루비에도 비슷한 표현이 있는 것으로 보아 대부분 언어에서 사용되는 표현이 아닌가 싶다.

```
import React, { Component} from 'react';
import ReactDOM from 'react-dom';

class Hello extends Component {
  render() {
    var place = "World"
    return (
      <h1>Hello {place}</h1>
    );
  }
}

ReactDOM.render(<Hello/>, document.getElementById('root'));
```

코드에서는 `palce` 변수의 값을 표출하였으며, 이 변수를 조작해서 출력되는 `h1`도 조작 할 수 있다.

### 다음 포스트

컴포넌트를 포스트 하기전에 몇 가지 기능을 포스트했다. 동적 값은 매우 필수적인 기능이지만, 구조분해 할당은 정말 유용한지 좀 더 연구해봐야 할 듯 싶다. 아직까지는 이게 필요한가? 감이 안잡히기 때문이다. 쓰면서 느낀 바나 알게된 사실이 있으면 추가하겠다.

---
* Related Links
  * [비구화 할당 MDN 문서](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
