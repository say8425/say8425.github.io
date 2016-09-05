---
layout: post
title: ActionMailer의 smtp가 gmail과 연동되지 않을때
categories: rails
---

## 개요
ActionMailer 초기 구츅하는 방법은 과감하게 생략한다. [구축하는 방법](https://matharvard.ca/posts/2014/jan/11/contact-form-in-rails-4/)이 길어서 그렇지 어렵지는 않다. 조금이나마 단축하고 싶다면 [mail_form](https://github.com/plataformatec/mail_form) gem을 사용하자. 이외에 단순하게 contact us만 구성할 수 있는 [contacu_us](https://github.com/JDutil/contact_us) gem도 있다. 하지만 controller와 routes를 기본적으로 건들 수 없다. 물론 마음만 먹으면 건들 수 있지만 그 정도로 건들 것을 감안하고 만들어진 gem은 아니므로, 차라리 직접 만들거나 `mail_form` gem을 사용할 것을 추천한다.

## smtp 설정하기

```ruby
config.action_mailer.default_url_options = { host: '도메인' }
config.action_mailer.delivery_method = :smtp
config.action_mailer.perform_deliveries = true
config.action_mailer.raise_delivery_errors = true
config.action_mailer.smtp_settings = {
        :address              => 'smtp.gmail.com',
        :port                 => 587,
        :domain               => '도메인',
        :user_name            => '계정@도메인',
        :password             => '비밀번호',
        :authentication       => 'plain',
        :enable_starttls_auto => true
}
```

### config.action_mailer.default_url_options

> config.action_mailer.default_url_options = { host: '도메인' }

mailer view에서 사용될 `url`를 설정한다. mailer_view에서는 `welcome_path`와 같은 _path helper를 사용할 수 없고, `welcome_url`과 같은 _url helper만 사용할 수 있다. 그리고 이때 사용된 _url helper의 host에 들어가는 것이 `config.action_mailer.default_url_options`이다. 이렇게 골룸하게 설정해야 하는 이유는 mailer는 controller와 달리 incoming request가 없기때문이다. 자세한 설명은 [문서](http://guides.rubyonrails.org/action_mailer_basics.html#generating-urls-in-action-mailer-views)를 참조.

### config.action_mailer.delivery_method

> config.action_mailer.delivery_method = :smtp

메일을 보낼때 사용할 method를 설정한다. 당연하지만 smtp를 사용한다.

### config.action_mailer.perform_deliveries

> config.action_mailer.perform_deliveries = true

development에서 작동여부를 [설정한다](http://stackoverflow.com/a/20770131/3910390). true시 작동하고, false시 작동안한다. `config/environments`에서 환경별로 true, false를 설정해주거나, `ENV['ACTION_MAILER_PERFORM_DELIVERIES']`와 같은 환경변수를 만들어서 관리해주자.

### config.action_mailer.raise_delivery_errors

> config.action_mailer.raise_delivery_errors = true

에러 발생시 에러를 던진다. console로 보여주지 않고 말그대로 throw 하기때문에 begin-resque를 [설정](https://github.com/rails/rails/issues/7473)해줘야 한다.

### config.action_mailer.smtp_settings

> config.action_mailer.smtp_settings = { <br>
&nbsp;&nbsp;&nbsp;        :address              => 'smtp.gmail.com',<br>
&nbsp;&nbsp;&nbsp;        :port                 => 587,<br>
&nbsp;&nbsp;&nbsp;        :domain               => '도메인',<br>
&nbsp;&nbsp;&nbsp;        :user_name            => '계정@도메인',<br>
&nbsp;&nbsp;&nbsp;        :password             => '비밀번호',<br>
&nbsp;&nbsp;&nbsp;        :authentication       => 'plain',<br>
&nbsp;&nbsp;&nbsp;        :enable_starttls_auto => true<br>
}

smtp에 대한 설정이다. 사실 smtp만 동작하게 만들거면 `config.action_mailer.delivery_method = :smtp`만 빼고 다른 설정은 전부 필요없다.

#### :address

> :address              => 'smtp.gmail.com'

smtp가 접속할 주소다. 더이상 설명이 必要韓紙?

#### :port

> :port                 => 587,

접속할 포트번호를 설정한다. 마찬가지로 자세한 설명은 생략한다.

#### :domain

> :domain               => '도메인'

툭 까놓고 말해 gmail을 사용할거라면, 무엇을 넣어도 상관없다. ~~이름값을 못한다~~ 이름과 달리 `:domain`은 [HELO 도메인](https://en.wikipedia.org/wiki/Anti-spam_techniques#HELO.2FEHLO_checking)을 설정한다. 하지만 gmail smtp는 HELO 도메인을 [사용하지 않으므로](http://stackoverflow.com/a/8439535/3910390) 무엇을 입력해도 상관없다. 하지만 혹시 모르니 [유지보수 어렵게 코딩하기](http://www.hanbit.co.kr/ebook/look.html?isbn=9788979149418) 마냥 삐딱선 타지 말고 정상적으로 입력해주자. HELO 도메인은 펭귄도 이해하기 어려워서 본 문서에서는 생략하겠다.

#### :user_name

> :user_name            => '계정@도메인'

유저네임. 즉 ID다. 주의할 점은 `say8425@gmail.com`와 같이 전체주소를 입력해줘야한다. 혹시 gmail에서 도메인을 생성해 사용하고 있다면, 역시 `say8425@penguin.com`와 같이 전체주소를 입력해줘야한다.

#### :password

> :password             => '비밀번호'

비밀번호. 보안상 그대로 노출하기 난감하므로 환경변수를 따로 만들어 주는 것이 좋다.

#### :authentication

> :authentication       => 'plain'

암호화 인증방법을 설정한다. gmail의 경우 plain, 즉 없다. 이것이 보안적으로 무엇을 의미하는지 모르겠다. 하지만 base64암호화를 [요구](http://stackoverflow.com/questions/25872389/rails-4-how-to-correctly-configure-smtp-settings-gmail#comment48605346_25872863)하는 경우도 있어보인다. 이 경우 'login'을 사용해야한데, 일반적으로 'plain'이면 충분하다.

#### :enable_starttls_auto

> :enable_starttls_auto => true

smtp 서버가 tls를 사용하는지 감지해서, 사용하면 켜고, 미사용시 끈다. 과거에는 [smtp-tls](https://github.com/ambethia/smtp-tls) gem이 따로 필요했다.


## smtp를 사용합니다. 안되잖아?

사실 이렇게 설정해도 smtp는 안되는 경우가 많다. 보통 철자가 틀리거나, 서버를 껐다키지 않아 setting이 반영되지 않아 그런 걸 수 있으므로 서버를 꼭 껐다 켜보자. 하지만 펭귄의 경우 무슨 수를 써도 절대로 접속이 실패했는데, 알고보니 gmail에서 접근을 거부한 것이었다.

![로그인 시도가 차단됨]({{ site.baseurl }}/images/2015-12-2-Do-Not-Work-SMTP/mail.png)`

만약 첫 접속시 이런 메일을 받았다면, 접근 거부 당한 것을 의심해봐야한다. 이 경우 [보안 수준이 낮은 앱의 액세스](https://www.google.com/settings/security/lesssecureapps)를 허용해주자. 일반 gmail의 경우 이런 일이 없었는데, 사설 도메인을 사용하자 이런 일이 발생했다. 사설 도메인이 기업용 서비스라서 생긴 이슈가 아닐까 추측만해본다.

## 결론
사실 펭귄과 같은 이슈는 매우 드문 케이스같아 보였다. 아무리 구글링을 해도 이런 해결법은 어디에도 제시되지 않았기 때문이다. 따라서 보통은 철자가 틀리거나 서버를 껐다키지 않아 이슈가 생긴다. 반대로 말해 철자만 안틀리면 이슈가 생길리가 없다. 물론 `:authentication`이라든가 몇 가지 변수는 있으므로, 재설정하고나면 꼭 서버를 껐다키자.

---
* Related Links
  * [Action Mailer 공식 가이드](http://edgeguides.rubyonrails.org/action_mailer_basics.html)
  * [Gmail 보안수준이 낮은 앱 액세스 허용하기](https://www.google.com/settings/security/lesssecureapps)
  * [contacu_us gem으로 contacu us 페이지 만들기](http://l4u.github.io/articles/create-a-rails-4-site-with-contact-us-form/)
  * [Action Mailer로 초기 세팅부터 페이지까지 만들기](https://matharvard.ca/posts/2014/jan/11/contact-form-in-rails-4/)
  * [smtp 설정하기](http://usingname.space/2015/07/25/gmail-smtp-ruby-on-rails-actionmailer-and-you/)