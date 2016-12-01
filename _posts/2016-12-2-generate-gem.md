---
layout: post
title: 루비 대림절 특집 - Gem 만들기
categories: rails
---

## 개요
루비를 쓰다보면 정말로 많은 gem들을 사용하게 된다. 정말 사용하기 편하고 프로젝트 개발 시간도 훨씬 줄여준다. 그런데 가끔씩 내가 원하는 gem이 없을때가 있다. 이럴때는 어쩔 수 없이 직접 작성하거나 혹은 다른 엯촋분들이 작성해주기를 기다려야한데, 한번 직접 gem을 만들어 보는 것은 어떨까? 물론 누구나 한번씩은 gem을 직접 만들어오는 망상(?)을 해봤을 것이다. 하지만 어디서부터 시작해야할지 난감하고 어려운 거 아니야?? 하는 두려움도 있을 것이다. 그래서 펭귄이 직접 맨땅 헤딩해본 경험을 공유해본다.

## NEW GEM
처음 gem을 만들게 된 계기는 Z사의 코딩 과제였다. 

```
Naver API를 사용해서 좌표는 주소로, 주소는 좌표로 반환 받는 루비 API를 만들어보세요.
```

사실 rest를 쏘는 코드는 많이 작성해봐서 자신 있었다. 그런데 이거를 그냥 만들기는 너무 심심해서 한번 gem으로 만들어보는 것은 어떨까 생각해봤다. 게다가 마침 소스 사이즈도 얼마 안할 거라는 판단이 들어서 시도해봤다. 

우선 예전에 Ruby Mine에서 얼핏 새 gem 만들기를 본 적 있는 거 같아서 New Project..를 실행해봤다. 럭키! 정말 있었다.

![Ruby Mine New Gem]({{ site.baseurl }}/images/2016-12-2-generate-gem/new-gem.png)

이름, 경로 그리고 루비버전을 설정하고 만들기 했다. 참고로 gem 이름은 소문자로 짓되, 공백은 언더바로 해주면 무난하다. 자세한 짓는 방법은 [이 문서](http://guides.rubygems.org/name-your-gem/)를 참고하자. 

하지만 생성 이후 계속 bundle process만 돌아가고 어떠한 파일도 생성되지 않았다. 뭔가 잘못됐나 해서 몇 번이고 다시 시도한 끝에 알게되었는데, Ruby Mine 터미널에서 무언가 입력을 기다리고 있었던 것이다. 개발자 이름은 무엇이냐? 라이센스는 무엇이냐? README는 생성할 것이냐? CODE_OF_CONDUCT는 생성할 것이냐? 등등을 묻는데, 전부 적당히 입력하고 y로 통쳐도 상관없다. 참고로 한번 세팅하면 gem을 이후 다시 한번 생성해도 물어보지 않는다. 하지만 개발 과정에서 언제든지 바꿀 수 있는 설정이므로 크게 연연하지 않아도 괜찮다. 세팅을 끝내면 다음과 같은 트리구조가 생성될 것이다.

```
├── CODE_OF_CONDUCT.md
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── bin
│   ├── console
│   └── setup
├── lib
│   ├── naver_map
│   │   └── version.rb
│   └── naver_map.rb
├── naver_map.gemspec
└── test
```

## gem 정보 입력하기
우선 우리가 만든 gem의 정보를 입력해주자 `naver_map.gemspec`이라는 파일이 있을 것이다. 열어오면 아래와 같은 코드가 나올 것이다.

```ruby
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'naver_map/version'

Gem::Specification.new do |spec|
  spec.name          = "naver_map"
  spec.version       = MyGem::VERSION
  spec.authors       = ["Penguin"]
  spec.email         = ["say8425@gmail.com"]

  spec.summary       = %q{TODO: Write a short summary, because Rubygems requires one.}
  spec.description   = %q{TODO: Write a longer description or delete this line.}
  spec.homepage      = "TODO: Put your gem's website or public repo URL here."
  spec.license       = "MIT"

  # Prevent pushing this gem to RubyGems.org. To allow pushes either set the 'allowed_push_host'
  # to allow pushing to a single host or delete this section to allow pushing to any host.
  if spec.respond_to?(:metadata)
    spec.metadata['allowed_push_host'] = "TODO: Set to 'http://mygemserver.com'"
  else
    raise "RubyGems 2.0 or newer is required to protect against " \
      "public gem pushes."
  end

  spec.files         = `git ls-files -z`.split("\x0").reject do |f|
    f.match(%r{^(test|spec|features)/})
  end
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.13"
  spec.add_development_dependency "rake", "~> 10.0"
end
```

spec 요소에 gem의 정보를 적어주자. 참고로 여기에 작성된 정보는 아래 이미지와 같이 rubygems.org 에 전부 display가 되므로 정확하게 적어주자.

![Ruby Gem Page]({{ site.baseurl }}/images/2016-12-2-generate-gem/ruby-gem-page.png)

### Version
version은 약간 특이하게 되었는데, `require 'naver_map/version'`를 보고 눈치챘겠지만 따로 Version이라는 클래스에서 받아온다. 따라서 버전을 수정하고 싶다면 `naver_map/version` 파일을 수정하면 된다. 디폴트가 왜 이렇게 번거로운지 모르겠지만, Version 클래스를 지워버리고 gemspec에 직접 버전을 입력해도 별 문제 없다. 심지어 공식 가이드에서도 gemspec에 버전을 직접 입력하는 방법을 보여주고 있으므로 편한대로 작성하자.

### Dependency
다 작성하고 아래를 보면 이런 코드가 있을 것이다.

```ruby
spec.add_development_dependency "bundler", "~> 1.13"
spec.add_development_dependency "rake", "~> 10.0"
```

이름에서도 알 수 있듯이, 이 gem에 대한 디펜던시를 선언하는 코드다. rails에서 하듯이 `Gemfile`에서 직접 선언해도 상관없지만, 공식 가이드에서는 `gemspec`에서 관리하는 방법을 보여주고 있다. 이는 위에서 봤듯이 디펜던시 또한 rubygem.org 에 display가 되는데, `Gemfile`에서 선언하면 보여주지를 못한다.

디펜던시를 선언하는 방법은 `add_runtime_dependency`, `add_development_dependency` 2가지가 있다. 전자는 gem이 실행되는 데 있어서 필요한 디펜던시이고, 후자는 개발과정에서 필요한 디펜던시이다. 즉 `Gemfile`의 `group :development do end`정도에 해당한다. 펭귄은 rest를 쏘는 gem이 필요하므로 `rest-client`를 추가하겠다.

```ruby
spec.add_dependency "rest-client", "~> 2.0", ">= 2.0.0"
```

### Ruby version

```ruby
spec.required_ruby_version = ">= 2.2.0"
```

안해줘도 상관 없는데, 필요한 루비 최소버전을 보여주고 싶다면 추가해주자. 참고로 이 값을 넣지않으면, 필요한 루비 최소버전은 0.0이상이라고 display된다.

다 작성된 코드는 아래와 같다.

```ruby
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'naver_map/version'

Gem::Specification.new do |spec|
  spec.name          = "naver_map"
  spec.version       = NaverMap::VERSION
  spec.author        = ["Penguin"]
  spec.email         = ["say8425@gmail.com"]

  spec.summary       = "Generating Naver map API"
  spec.description   = "Naver support map API. But we don't have any ruby gem for this API.
                        So You can use it with this gem comfortable."
  spec.homepage      = "https://github.com/say8425/naver_map_ruby"
  spec.license       = "MIT"

  # Prevent pushing this gem to RubyGems.org. To allow pushes either set the 'allowed_push_host'
  # to allow pushing to a single host or delete this section to allow pushing to any host.
  if spec.respond_to?(:metadata)
    spec.metadata['allowed_push_host'] = "https://rubygems.org/"
  else
    raise "RubyGems 2.0 or newer is required to protect against " \
      "public gem pushes."
  end

  spec.files         = `git ls-files -z`.split("\x0").reject do |f|
    f.match(%r{^(test|spec|features)/})
  end
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]
  spec.required_ruby_version = ">= 2.2.0"

  spec.add_dependency "rest-client", "~> 2.0", ">= 2.0.0"

  spec.add_development_dependency "bundler", "~> 1.13"
  spec.add_development_dependency "rake", "~> 11.2", ">= 11.2.2"
end
```

## 코딩하기

지겨운 설정은 이쯤에서 마무리 짓고, 이제 직접 코딩해보자. 디렉토리중에서 `lib`이라고 있을 것이다. gem이 작동하는 코드는 이 디렉토리에 있다. `lib/naver_map.rb`를 열어보자.

```ruby
require_relative './naver_map/version'

module MyGem
  # Your code goes here...
end
```






사실 펭귄도 지금까지 페이지 넘기는거, 페이지 목록, 페이지 리스트 등등 두리뭉실하게 불렀는데, 이걸 pagination이라고 부른다는 거 이번에 처음 알게되었다.
pagination을 만들어주는 gem은 몇 개 없다. 대표적인 gem으로 [kaminari](https://github.com/amatsuda/kaminari)와 [will_paginate](https://github.com/mislav/will_paginate)가 있으며, 이 중 펭귄은 will_paginate를 이미 사용하고 있어서, 이 gem으로 만들기로 했다. 물론 [kaminari로도 만들 수 있으니](https://github.com/amatsuda/kaminari/wiki/How-To%3A-Create-Infinite-Scrolling-with-jQuery), 관심있다면 구글신께 신탁을 받아보자.

## 시작은 똑같으리니..

will_paginate gem 설치하는 방법은 생략하겠다. 이미 [gitHub문서](https://github.com/mislav/will_paginate/wiki/Installation)에 상세히 나와있고, 특별히 어려운 거 없이 Gemfile.rb에 붙이고, `bundle install` 돌려주면 끝이다.

```ruby
# CreatorsController
def index
  @creators = Creator.all.paginate(page: params[:page], per_page: 2)
...
end
```

그리고 more button을 적용하고 싶은 곳의 컨트롤러에 will_paginate를 세팅하자. 펭귄은 Creator모델 전체를 대상으로 삼았다. 여기에 paginate를 걸어주자. 그리고 매 로드마다 몇개씩 불러올지 `per_page`로 설정하자. 여기까지는 `will_paginate` 기본 세팅하는 방법과 똑같다. 이후가 다르다.

##
만약 평범한 pagination을 만들고 싶다면, view 파일에 partial를 render해주고, 하단에 `<%= will_paginate @creators %>`를 붙여주면 끝이다. 하지만 우리는 원하는 것은 more 버튼이라서 다른 방법을 찾아야 한다.

우선 partial를 render해주는 점은 똑같다. 앞서 말했듯이 more button은 pagination과 똑같다. 다만 요청된 새로운 페이지를 예전 페이지와 교체하는 것이 아니고 예전 페이지의 뒤에 이어서 붙이는 거다.

정리하면 partial를 렌더한다. 뒤에 새로운 페이지를 요청하는 more button을 배치한다. 요청을 하면 새로운 페이지를 받아와서 기존 render된 partial뒤에 붙여준다. more button은 table에 row가 아직 남아있다면 새로운 페이지를 가리키게 하고, 없으면 지워버린다. 로 정리될 수 있다.




## 결론
사실 펭귄과 같은 이슈는 매우 드문 케이스같아 보였다. 아무리 구글링을 해도 이런 해결법은 어디에도 제시되지 않았기 때문이다. 따라서 보통은 철자가 틀리거나 서버를 껐다키지 않아 이슈가 생긴다. 반대로 말해 철자만 안틀리면 이슈가 생길리가 없다. 물론 `:authentication`이라든가 몇 가지 변수는 있으므로, 재설정하고나면 꼭 서버를 껐다키자.

---
* Related Links
  * [will_pagiante Gem](https://github.com/mislav/will_paginate)
  * [unlimited scroll 하는 법](http://www.sitepoint.com/infinite-scrolling-rails-basics/)
  * [세팅하는 방법에 대한 StackOverflow 문서, 가장 도움되었다](http://stackoverflow.com/a/23591939/3910390)
  * [세팅하는 방법에 대한 StackOverflow 문서 2](http://stackoverflow.com/questions/8378443/will-paginate-with-load-more-button-in-rails-3-1-using-jquery)
  * [세팅하는 방법에 대한 StackOverflow 문서 3](http://stackoverflow.com/questions/5543719/rails-3-kaminari-jquery-load-more-button-problem-to-display-results-it-loads)
  * [ajaxf르 어떻게 짜야할지 힌트를 얻은 StackOverflow 문서](http://stackoverflow.com/questions/7188287/rails-will-paginate-show-next-10-button)
  * [pagination대신 다른걸 만드는 법에 대한 StackOverflow 문서, 시도는 안해봤다](http://stackoverflow.com/questions/2497821/rails-load-more-instead-of-pagination)
  * [kaminari로 more button 만들기](https://github.com/amatsuda/kaminari/wiki/How-To%3A-Create-Infinite-Scrolling-with-jQuery)
