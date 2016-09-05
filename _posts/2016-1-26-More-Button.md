---
layout: post
title: will_paginate gem으로 more button 만들기
categories: rails
---

## 개요
more button은 몇 개의 아이템을 샘플로 보여주고, 이후 유저의 의향에 따라 더 많은 아이템을 보여주는 흐름을 가지고 있다. 이 때문에 주로 사진 갤럭리에서 많이 쓰이며, 대표적으로 Instagram web 버전이 있다. 이 more button을 만들어주는 Gem이 있을 줄 알았는데, 안타깝게도 아직까지는 없어보인다. 대신 pagination으로 만드는 방법이 많이 소개되었다. 사실 둘은 내부적으로 비슷하다. 샘플로 보여준 아이템들을 첫번째 페이지라고 생각하고, 이후 더 불러올 아이템들을 n번째 페이지라고 생각하면 똑같다. 다만 페이지가 교체되는 것이 아니고, 누적된다는 점에서 다르다. 이점에서 착안해 pagination으로 more button 만들기가 시도되고 있는 것으로 보인다.

펭귄은 [will_pagination Gem](https://github.com/mislav/will_paginate)으로 삽질했다. 다만 꼭 이 gem으로 할 이유는 없다. 이미 다른 gem으로도 많이 시도되고 있으니, 관심 있다면 구글신께 신탁을 받자.

## 그런데 pagination이 뭡니까?

![pagination]({{ site.baseurl }}/images/2016-1-26-More-Button/pagination.png)

~더 이상 자세한 설명을 생략한다.~

사실 펭귄도 지금까지 페이지 넘기는거, 페이지 목록, 페이지 리스트 등등 두리뭉실하게 불렀는데, 이걸 pagination이라고 부른다는 거 이번에 처음 알게되었다.
pagination을 만들어주는 gem은 몇 개 없다. 대표적인 gem으로 [kaminari](https://github.com/amatsuda/kaminari)와 [will_paginate](https://github.com/mislav/will_paginate)가 있으며, 이 중 펭귄은 will_paginate를 이미 사용하고 있어서, 이 gem으로 만들기로 했다. 물론 [kaminari로도 만들 수 있으니](https://github.com/amatsuda/kaminari/wiki/How-To%3A-Create-Infinite-Scrolling-with-jQuery), 관심있다면 구글신께 신탁을 받아보자.

## 시작은 똑같으리니..
will_paginate gem 설치하는 방법은 생략하겠다. 이미 [gitHub문서](https://github.com/mislav/will_paginate/wiki/Installation)에 상세히 나와있고, 특별히 어려운 거 없이 Gemfile.rb에 붙이고, __bundle install__ 돌려주면 끝이다.

```ruby
# CreatorsController
def index
	@creators = Creator.all.paginate(page: params[:page], per_page: 2)
...
end
```

그리고 more button을 적용하고 싶은 곳의 컨트롤러에 will_paginate를 세팅하자. 펭귄은 Creator모델 전체를 대상으로 삼았다. 여기에 paginate를 걸어주자. 그리고 매 로드마다 몇개씩 불러올지 __per_page__로 설정하자. 여기까지는 __will_paginate__ 기본 세팅하는 방법과 똑같다. 이후가 다르다.

##
만약 평범한 pagination을 만들고 싶다면, view 파일에 partial를 render해주고, 하단에 __<%= will_paginate @creators %>__를 붙여주면 끝이다. 하지만 우리는 원하는 것은 more 버튼이라서 다른 방법을 찾아야 한다.

우선 partial를 render해주는 점은 똑같다. 앞서 말했듯이 more button은 pagination과 똑같다. 다만 요청된 새로운 페이지를 예전 페이지와 교체하는 것이 아니고 예전 페이지의 뒤에 이어서 붙이는 거다.

정리하면 partial를 렌더한다. 뒤에 새로운 페이지를 요청하는 more button을 배치한다. 요청을 하면 새로운 페이지를 받아와서 기존 render된 partial뒤에 붙여준다. more button은 table에 row가 아직 남아있다면 새로운 페이지를 가리키게 하고, 없으면 지워버린다. 로 정리될 수 있다.




## 결론
사실 펭귄과 같은 이슈는 매우 드문 케이스같아 보였다. 아무리 구글링을 해도 이런 해결법은 어디에도 제시되지 않았기 때문이다. 따라서 보통은 철자가 틀리거나 서버를 껐다키지 않아 이슈가 생긴다. 반대로 말해 철자만 안틀리면 이슈가 생길리가 없다. 물론 __:authentication__이라든가 몇 가지 변수는 있으므로, 재설정하고나면 꼭 서버를 껐다키자.

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
