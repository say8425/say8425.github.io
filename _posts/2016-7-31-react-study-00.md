---
layout: post
title: reactjs 공부하기 - 00
categories: react
---

본 내용은 [프로 리엑트](http://www.aladin.co.kr/shop/wproduct.aspx?ItemId=84599144)라는 책을 스터디하면서 정리한 내용이다. reactjs에 대한 장단점 등은 다른 글에 모두 상세히 잘 쓰여있으므로 생략한다.

### node.js 설치하기

[node.js 홈페이지](https://nodejs.org/)에서 node.js를 다운받는다. 6 버전은 아직까지 호환성 문제가 많으므로 4.4.7버전을 다운받는다. 참고로 homebrew에는 4 버전이 없으므로 직접 다운 받아 설치하자.

### 디렉토리 및 npm init

npm은 node.js에 사용되는 모듈 패키지 관리 툴이다. node.js를 사용하기 앞서 필수적으로 생성해줘야 한다. 우선 디렉토리를 생성하자. 그리고 진입한 뒤에 npm을 생성해주자.

```
mkdir react-study
cd react-study
npm init
```

참고로 앞으로 터미널을 쓸 일이 매우 많으므로 디렉토리 생성도 터미널로 해주는 것이 편하다. npm init을 해주면 프로젝트 이름, 작성자, git 레포짓 등을 묻는데, 전부 나중에 바꿔줄 수 있으므로 귀찮다.면 enter를 연타 해주자.

그리고 패키지를 설치해주자.

```
npm install react react-dom --save
npm install react webpack webpack-dev-server babel babel-core babel-loader --save-dev
npm install react babel-preset-es2015 babel-preset-react --save-dev
```

첫번째는 보시다시피 react를 설치한다. 두번째와 세번째는 바벨과 웹팩을 설치한다. 이 명렁어가 무엇을 의미하는지는 다음에 상세히 서술하겠다.

### 소스 작성하기

npm 설정 및 설치는 명령어로 할 수 있지만, 나머지 필요한 소스는 직접 작성해야한다. 최종적인 소스트리는 아래와 같을 것이다.

```
.
├── index.html
├── node_modules
│   ├── babel
│   ├── babel-core
│   ├── babel-loader
│   ├── babel-preset-es2015
│   ├── babel-preset-react
│   ├── react
│   ├── react-dom
│   ├── webpack
│   └── webpack-dev-server
├── package.json
├── source
│   └── App.js
└── webpack.config.js
```

#### index.htmll

읽어들일 html 파일을 만들어주자. _index.html_파일을 최상위 path에 생성해 아래와 같이 코딩하자.

```
<!-- index.htmll -->
<!DOCTYPE html>
<html>
<head>
  <title>리엑트 스터디</title>
</head>
<body>
  <div id="root"></div>
  <script type="text/javascript" src="bundle.js"></script>
</body>
</html>
```

보시다시피 평범한 html파일이다. 다만 _root_라는 id셀렉터를 가진 div가 있는데, reactjs가 생성한 컴포넌트는 이 div에 위치될 것이다.
_bundle.js_는 webpack이 js들을 취합하여 생성한 js이다. 엄밀히는 취합이라는 단어가 정확하지 않지만 지금은 이렇게 설명하고 넘어가겠다.

#### webpack.config.js

그 뒤 webpack을 설정하는 파일을 생성한다. 파일명은 _webpack.config.js_로 해주며 path는 마찬가지로 최상위이다.

```
// webpack.config.js
module.exports = {
  entry: [
    './source/App.js'
  ],
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [{
      test: /\.jsx?$/,
      exclude: '/node_modules',
      loader: 'babel',
      query: {
        presets: ['es2015', 'react']
      }
    }]
  }
};
```

#### source/App.js

마지막으로 react 컴포넌트를 생성해주는 js파일을 만들어준다. _App.js_라는 이름으로 만들어주며 path는 _source_디렉토리이다. _App.js_뿐만 아니라, js와 css들은 이 디렉토리에 위치된다. 그래서 webpack설정에도 _entry: ['./source/App.js']_와 같이 잡혀있다.

```
// source/App.js
import React from 'react';

class Hello extends React.Component {
  render() {
    return (
      <h1>Hello World</h1>
    );
  }
}

React.render(<Hello/>, document.getElementById('root'));
```

그리고 _node_modules/.bin/webpack-dev-server_라는 상당히 긴 명령을 터미널에서 실행하면 로컬서버가 돌아간다.
_localhost:8080_을 접속해서 우리가 코딩 한 것을 확인해보자.

![react-error-page]({{ site.baseurl }}/images/2016-7-31-react-study-00/error-blank.png)

하지만 대부분의 사람들은 텅빈 화면을 볼 수 있을 것이다. 브라우저 요소점검에서 콘솔을 확인해보자. 그러면 아래와 같은 에러가 있을 것이다.

```TypeError: _react2.default.render is not a function. (In '_react2.default.render(_react2.default.createElement(Hello, null), document.getElementById('root'))', '_react2.default.render' is undefined)```

상당히 긴 문구의 에러인데 대충 _render_라는 function을 찾을 수 없다는 에러다. 왜 이런 에러가 났을까? 그 이유는 [reactjs 0.14버전부터 render는 react-dom으로 옳겨졌기 때문이다](http://stackoverflow.com/a/33880428/3910390). 그래서 _render_를 찾을 수 없는 것이다. _react-dom_을 import하고 여기에서 _render_를 불러오자.

```
// source/App.js

import React from 'react';
import ReactDOM from 'react-dom';

class Hello extends React.Component {
  render() {
    return (
      <h1>Hello World</h1>
    );
  }
}

ReactDOM.render(<Hello/>, document.getElementById('root'));
```

소스를 위와같이 바꾼뒤 다시 페이지를 새로고침하면 *Hello World* 를 볼 수 있을 것이다.

![react-hello-world]({{ site.baseurl }}/images/2016-7-31-react-study-00/hello-world.png)

### One more thing - npm start

로컬 서버를 시작할때 _node_modules/.bin/webpack-dev-server_를 입력했던 것 기억하는가? 상당히 길어서 타이핑 할때마다 암세포가 생성 될 것이다. 스크립트 명령어를 하나 추가해서 암예방하자. _package.json_을 들어가서 _scripts_ 키에 _"start": "node_modules/.bin/webpack-dev-server --progress"_란 밸류를 추가하자. 눈치 빠른 사람은 알겠지만 _node_modules/.bin/webpack-dev-server_를 start라는 명령어로 떼우겠다는 코드다. 

```
// package.json
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node_modules/.bin/webpack-dev-server --progress"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^15.3.0",
    "react-dom": "^15.3.0"
  },
  "devDependencies": {
    "babel": "^6.5.2",
    "babel-core": "^6.11.4",
    "babel-loader": "^6.2.4",
    "babel-preset-es2015": "^6.9.0",
    "babel-preset-react": "^6.11.1",
    "react": "^15.3.0",
    "webpack": "^1.13.1",
    "webpack-dev-server": "^1.14.1"
  }
}
```

최종적으로 _package.json_ 코드는 위와 같을 것이다. 코드를 작성 하였다면 _npm start_로 간단하게 로컬 서버를 실행하자.

### 다음 포스트

다음 포스트에서는 컴포넌트를 가지고 놀겠다.
참고로 본 연재글은 이론적인 설명보다 소스를 직접 타이핑 해보는 코딩 위주로 진행될 것이다.
좀 더 자세한 이론적인 배경은 구글신에게 신탁을 받자.

---
* Related Links
  * [render를 못찾는 에러에 대한 SO 문서](http://stackoverflow.com/a/33880428/3910390)
  * [velopert의 reactjs 강좌](https://velopert.com/775)
  * [reactjs 둘러보기](https://taegon.kim/archives/5097)