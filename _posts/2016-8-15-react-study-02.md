---
layout: post
title: reactjs 공부하기 - 02
categories: react
---

이제 컴포넌트를 다뤄보자. 컴포넌트는 리엑트를 구성하는 덩어리들이다. reactjs는 이 공통되는 최소사항들을 컴포넌트 단위로 쪼개고 조립 할 것을 권장한다. 말이 어려우니 직접 코딩해보자.

### 컴포넌트

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

class GroceryList extends Component {
  render() {
    return (
      <ul>
        <ListItem quantity="1" name="Bread" />
        <ListItem quantity="6" name="Eggs" />
        <ListItem quantity="2" name="Milk" />
      </ul>
    );
  }
}

class ListItem extends Component {
  render() {
    return (
      <li>
        {this.props.quantity} x {this.props.name}
      </li>
    );
  }
}

ReactDOM.render(<GroceryList />, document.getElementById('root'));
```

전에 작성한 코드와 달리 `class` 가 2개가 선언 된 것이 보일 것이다. `extends Component` 가 보인가? 해당 클래스를 컴포넌트로 사용하겠다고 선언 한 것이다. 그러니까 우리는 이미 컴포넌트를 사용한 것이다. 첫번째 컴포넌트 `GroceryList` 를 보자. `<ListItem quantity="1" name="Bread" />` 라는 구문이 있다. 이것은 `ListItem`이라는 컴포넌트를 사용하고, 해당 컴포넌트의 quantity와 name이라는 property에 각각 "1"과 "Bread"라는 값을 넣겠다는 것이다. 그러면 이 컴포넌트는 어디 있는가? 바로 아래에 있다. 

아래에 `class ListItem extends Component`라는 또다른 클래스가 선언 된 것이 보일 것이다. 이 컴포넌트를 보자. 이 컴포넌트는 <li>태그를 반환한다. 어떻게 반환하는가? `{this.props.quantity} x {this.props.name}`라는 값을 품고. 이 값은 무엇일까? `{}` 혹시 기억하는가? 이것은 변환 될 수 있는 값임을 의미한다. `{}`안에는 `this.props.quantity`라는 구문이 있다. 보이는대로 이 컴포넌트의 __quantity라는 속성__을 의미한다. 다시 `GroceryList`컴포넌트에서 호출한 `<ListItem quantity="1" name="Bread" />`구문을 보자. `quantity="1"`가 보인가? `ListItem`라는 컴포넌트를 호출하면서 quantity의 속성은 "1"로 넣어줬다. 그리고 name 속성은 "Bread". 그래서 결좌적으로 `ListItem`컴포넌트는 아래와 같이 반환한다.

```
<li>1 x Bread</li>
```

`<ListItem quantity="6" name="Eggs" />`도 마찬가지로 `<li>6 x Eggs</li>`라는 구문을 반환한다. 다만 실제 반환되는 값은 이렇게 간단하지 않지만 지금은 이렇게 알고 가자.

### IDE

이쯤 됐으면 좀 더 편하게 코딩하고 싶을 것이다. 설마 notepad에서 작업하고 있었다면 빨리 다른 IDE를 알아보자. 대부분 [아톰](https://atom.io)이나 [서브라임 텍스트](http://www.sublimetext.com)를 사용할텐데 펭귄은 아톰을 사용하고 있다. 서브라임 자체도 매우 훌룡한 에디터지만 플러그인 메인테인이 워낙 지지부진 해서다. 그리고 프로젝트 단위로 작업하기도 아톰이 편하다. 만약 아톰을 사용할 것이라면 [babel 패키지](https://github.com/gandm/language-babel)를 설치하고 사용 할 것을 추천한다. 당연히 자동완성도 지원하며 발암율도 급격히 내려갈 것이다. 

{% include image.html
           img="images/2016-8-15-react-study-02/auto-complete.png"
           title="You neee autocomplete"
           caption="auto complete 없이 코딩한 자의 최후" %}

babel 패키지 말고 [react 패키지](https://github.com/orktes/atom-react)도 따로 있다. 둘 다 메인테인이 매우 활발하니 골라 쓰자. 다만 펭귄은 babel 패키지에도 이미 react 하이라이트와 자동완성이 포함되있고 어차피 사용하는 거 좀 더 포괄적인 것을 사용하고 싶어서 babel 패키지를 사용하고 있다.

---
* Related Links
  * [Atom](https://atom.io)
  * [Sublime Text](http://www.sublimetext.com)
  * [Atom Babel Package](https://github.com/gandm/language-babel)
  * [Atom React Package](https://github.com/orktes/atom-react)
