---
layout: post
title: Rails admin 사용하기 - 1
---

# 개요
admin페이지를 만드는데, 생각보다 귀찮은 작업이었다. 그래서 혹시 admin페이지 gem이 있나 찾아보았는데 역시 있었다. 그것도 무려 2가지나!

 * [Rails Admin](https://github.com/sferik/rails_admin)
 * [Active Admin](https://github.com/activeadmin/activeadmin)

찾아본 바로는 이 2가지 gem이 admin계의 양대산맥이었다. 무엇을 쓸까 했는데, rails admin 개발자의 신랄한 active admin[디스](http://www.slideshare.net/benoitbenezech/rails-admin-overbest-practices)에 반해 rails admin으로 정했다.~~물론 영어라서 대부분 한 마디도 못알아들었다~~

# 설치
여느 gem과 마찬가지로 `Gemfile.rb`에 `gem 'rails_admin'`때려박아주고, `rails g rails_admin:install`명령해주면 setup 이 된다. 그리고 주소창에 `주소/admin` 입력해 접속하면 된다. 신기하게도 model이란 model은 싸그 긁어와 보여주는 rails admin을 볼 수 있을 것이다.

#세팅
## [translation.missing](https://github.com/sferik/rails_admin/wiki/Translations)
[I18n](https://github.com/svenfuchs/i18n)gem을 적용했다면, 아마 처음 접속시 translation.missing이라고 뜨면서 일부글들이 제대로 보이지 않을 것이다. 한국어 locale이 없어서 생긴 일인데, [이 translation](https://gist.github.com/YoonjaeYoo/787eb279e5d46c7e96dc)을 다운받아 적당한 이름을 붙여서(펭귄은 rails_admin.ko.yml이라고 붙였다) locale 디렉토리에 넣어주자. 그래도 몇몇 부분이 translation.missing이 뜰 수 있는데, i18n 한국어 locale 파일이 없기 때문이다.. [이 translation](https://github.com/svenfuchs/rails-i18n/blob/master/rails/locale/ko.yml)도 locale 디렉토리에 넣어주자. 끝으로 번역해주신 contributor분들 정말 고맙습니다(__)

## [devise와 연동하기](https://github.com/sferik/rails_admin/wiki/Devise)
rails admin은 관리자 계정과의 연동을 필요로 한다. 물론 없어도 된다. 하지만 상식적으로 권한 제한을 두지않고 admin 페이지를 오픈 하는 건, `들어와들어와~ 우리집, 어서 **해킹**해줘`와 별 다를바가 없어서 사실상 연동은 필수다. 펭귄은 devise로 이미 만들어둔 admin 모델이 있어서 그 모델과 연동하기로 했다.

```ruby
# rails_admin.rb
RailsAdmin.config do |config|
	# 새로 추가될 코드 start
	config.authenticate_with do
	    warden.authenticate! scope: :admin
	  end
	config.current_user_method(&:current_admin)
	# 새로 추가될 코드 end
...
end

# routes.rb
Rails.application.routes.draw do
	devise_for :admin
	mount RailsAdmin::Engine => '/admin', as: 'rails_admin'
	...
end
```

문서에 나오듯이 `rails_admin.rb`에 위 코드를 넣어주자. 만약에 `admin`모델을 따로 만들지 않고, `user`모델을 만들었다면, `:admin`대신 `:user`를 입력해 `user`모델에 연동하자. 물론 `:current_admin`도 `:current_user`로 바꿔준다. `user`모델에 연동하기로 했다면, 해당 계정의 관리자 권한을 체크해 admin 페이지 진입 여부를 결정해주 된다. 자세한 사항은 [devise 문서](https://github.com/plataformatec/devise/wiki/How-To:-Add-an-Admin-Role#option-2---adding-an-admin-attribute)를 참조하자. 여기에서 좀 더 나아가면 [깡통3형제](https://github.com/CanCanCommunity/cancancan) gem과 [연동](https://github.com/sferik/rails_admin/wiki/Cancancan)할 수 있지만 다음 기회에 포스팅하겠다.

`routes.rb`에도 `mount RailsAdmin::Engine => '/admin', as: 'rails_admin'`를 넣어주자. 주의 할 점이 있는데 `devise_for :admin`보다 아래에 넣어줘야만 한다. 안그러면 에러가 난다. 아시다시피 ruby는 `컴파일러`가 아니고 `인터프리터`로 동작하기 때문에 순서가 매우 중요하다.

## [특정 model만 불러오기](https://github.com/sferik/rails_admin/wiki/Navigation)
rails_admin이 모든 model을 불러오는 건 원치 않을 것이다. 원하는 모델만 컨트롤  수 있게 만들자.

```ruby
# rails_admin.rb
RailsAdmin.config do |config|
...
	# Model Config
  	config.included_models = %w(Article Creator Portfolio)
  	config.config.excluded_models = %w(Admin)
end
```

가장 간단한 방법으로할는 `config.included_models`이 있다. 코드 그대로 여기에 포함된 모델들만 불러온다. 위 코드와 같은 경우, `article`, `creator`, `porfolio` 모델만 볼러온다. 참고로 rails admin wiki 문서대로 `%w(Article Creator Portfolio)`대신 `["Article", "Creator", "Portfolio"]`해줘도 상관 없지만, 이 syntax는 ruby style guide에 [어긋난다](https://github.com/bbatsov/ruby-style-guide#syntax). style 대로라면 위 코드가 맞아 이 코드를 추천한다.

역으로 특정 모델만 가리고 싶다면, `config.config.excluded_models`를 사용하면 된다.

## [navigation 세팅하기](https://github.com/sferik/rails_admin/wiki/Navigation)
rails admin 페이지의 left navigation을 건들어서 표기되는 label을 바꾸거나 특정 링크를 넣어줄 수도 있다.

### 라벨 바꾸기
엄밀히는 보여지는 이름을 바꾸는 것이다. 이걸 세팅하기 전까지는 Article Creator등 모델의 이름을 보일 것이다.

```ruby
# rails_admin.rb
RailsAdmin.config do |config|
...
	config.model 'Article' do
	    label '누스기사'
	    label_plural '뉴스기사'
	    navigation_icon 'icon-list-alt'
    end

    config.model 'Creator' do
	    label '크리에이터'
	    label_plural '크리에이터'
	    navigation_icon 'icon-user'
	end

	config.model 'Portfolio' do
	    label '포트폴리오'
	    label_plural '포트폴리오'
	    navigation_icon 'icon-film'
	end
end
```
소스와 같이 model 마다 config를 넣을 수 있는데, `label`로 이름을 정할 수 있다. `label_plural`란 것도 있는데, 이것은 복수형 라벨을 의미한다.

![label_plural.png]({{ site.baseurl }}/images/2015-11-9-setup-rails-admin/label_plural.png)

위 이미지는 `Portfolid`모델의 `label_plural`를 '포트폴리오들'로 설정한 이미지이다. 보시다시피 UI상 복수형 명사로 표기해야 할 경우 `포르폴리오들`이라고 표기된다. 하지만 한국어에서 복수형은 어색하므로, 전부 label과 동일하게 바꿔주자. 참고로 `label_plural`를 설정해주지 않으면, 모든 label마다 끝에 s가 붙는 소소한 참사가 일어나므로(예를 들면 포트폴리오s), 꼭 설정해주자.

`navigation_icon`은 이름마다 앞에 Glyphicons을 붙여주는 옵션이다. bootstrap에서 [원하는 icon을 찾아서](http://getbootstrap.com/components/) `glyphicon glyphicon-plus`라는 접두어 대신 `icon`이라는 접두어를 붙여서 넣어주면 된다. 필요하다면 디자이너와 파이팅을 벌이고 넣어주자.

### 특정 링크 넣기
navigation에 특정 링크가 표시되게 만들 수 있다. 운영자가 작업중에 자주 사용하는 링크가 있다면, 소소한 도움을 줄 수 있다.

```ruby
# rails_admin.rb
RailsAdmin.config do |config|
...
	config.navigation_static_links = {
	  'Google' => 'http://www.google.com'
	}
end
```

굳이 꼭 넣어줄 기능은 아니다. 일종의 선택사항이므로 필요하다면 넣어주자.

---

__계속__



 * Related Links
  - [Rails admin wiki](https://github.com/sferik/rails_admin/wiki)
  - [부가기능 설치하기](https://gist.github.com/re4lfl0w/fadc6bee495c63b4f893)
  - [CloudFlare로 도메인 사용하기](https://addnull.net/build-a-blog-with-jekyll-github-pages-and-cloudflare/)