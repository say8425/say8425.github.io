---
layout: post
title: gem 버전에서 ~> 은 무슨 의미인가?
categories: rails
---

`Gemfile`을 보면 `~>`을 많이 볼 수 있다. 이 오퍼레이터는 `pessimistic operator`라고 불리며, 루비 코어에서 사용되지 않고 루비 젬파일에서 사용되는 걸로 보인다. 일단 이 오퍼레이터의 의미부터 말하자면 해당 버전부터 소수점 한 자리수 위 미만을 의미한다. 예를 들어 

```ruby
gem 'rest-client', '~> 2.0.1'
```

위와같은 코드가 있다면 아래와 같은 의미이다.

```ruby
2.0.1 <= rest-client gem 버전 < 2.1.0
```

이라는 의미이다. 그래서 `bundle`을 사용해도 버전이 2.1.0 이상 올라가지 않는다. [유의적 버전](http://semver.org/lang/ko/)이라는 체계에 따라 의존성 관리할때 유용하며, thoughtbot에서 또한 [권장](https://github.com/thoughtbot/guides/tree/master/best-practices#bundler)하고 있다.

참고로 이 글을 작성하는 현 시점에서 rest-client 2.0.1 버전은 나오지 않았으며, 예시로 작성된 버전이다.

---
* Related Links
	* [thoughtbot에서 제시한 bundler 가이드](https://robots.thoughtbot.com/a-healthy-bundle)
	* [thoughtbot에서 pessimistic operator을 설명한 글](https://robots.thoughtbot.com/rubys-pessimistic-operator)
	* [유의적 버전 번역글 - 번역해주셔서 고맙습니다](https://github.com/hatemogi/semver)
