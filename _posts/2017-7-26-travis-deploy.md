---
layout: post
title: Travis CI를 사용해 AWS Elastic Beanstalk 에 배포 자동화 해주기
categories: rails
---

Travis CI는 test 환경을 구축해 테스트를 해주는 것은 물론, 테스트가 통과되면 deploy 를 진행하게 할 수도 있다.
Travis 는 [수 많은 deploy 방법](https://docs.travis-ci.com/user/deployment/)을 지원하고 있는데, 펭귄은 그중 rails를 AWS Elastic Beanstalk(이하 EB)에 배포 자동화를 했던 경험을 공유하겠다. 참고로 이 포스팅은 [7월 4일 AWSKrug에서 발표 했던 것](https://www.meetup.com/ko-KR/awskrug/events/241160036/?eventId=241160036)을 토대로 작성한 포스트다.

## travis 환경 구축

만약 travis 를 이미 사용하고 있다면 `.travis.yml`이라는 파일이 이미 있을 것이다. 만약 없다면 travis 를 구축해야 한다. 

구축하는 방법은 2가지다. 하나는 `.travis.yml`을 직접 만들고 [docs 에 나와있는 코드를 그대로 복붙](https://docs.travis-ci.com/user/getting-started/#To-get-started-with-Travis-CI)하는 방법이다. Travis CLI가 그렇게까지 필수는 아니라서 이 방법도 나쁘지 않다. 단 이 경우에는 Travis console 에 들어가서 레포지를 따로 연동해줘야한다. 물론 레포지를 연동하는 방법은 어렵지 않다. travis에서 레포지 목록에 들어가 아래 이미지와 같이 켜주면 된다.

![Enable Travis to Repository]({{ site.baseurl }}/images/2017-7-26-travis-deploy/enable-travis-repository.png)

또 다른 방법은 [Travis CLI](https://github.com/travis-ci/travis.rb)를 사용하는 방법이다. 터미널에서 `travis init` 명령을 해주면 `.travis.yml` 생성은 물론 레포지 연동까지 알아서 해준다. 물론 Travis CLI 가 설치되어 있어야 한다. 다만 이 Travis CLI 가 의외로 homebrew 나 apt-get 과 같은 패키지 관리툴로 설치 할 수 없고 ruby gem 으로 설치 할 수 있다. 이 때문에 ruby는 필수이다. 만약 개발 환경 OS 가 윈도우이고 ruby를 쓸 일이 없다면 Travis CLI 를 설치하는 것 보다 직접 `.travis.yml` 파일을 생성하는 것이 나을 수 있다. 

최종적으로 생성된 `.travis.yml`의 코드는 아래와 같을 것이다. 

```yml
language: ruby
rvm:
  - 2.0.0
  - 1.9.3
  - jruby-19mode
  - rbx-2
```

언어와 언어의 version 은 각자 환경에 맞게 바꿔주면 된다. 펭귄은 ruby 2.2.5 하나만 남기고 모두 지웠다.

## EB 연동하기

이제 EB 를 연동하자. EB 연동은 `travis setup elasticbeanstalk` 명령을 해주거나 [docs를 복붙](https://docs.travis-ci.com/user/deployment/elasticbeanstalk/)하면 되는데, 펭귄은 복붙 해와서 직접 입력하는 것을 추천한다. Travis CLI 명령어를 사용하면 키나 레포등을 물어보는데 성실하게(?) 전부 답변해주면 `.travis.yml`에 그대로 전부 적히는데, 이 `.travis.yml` git에 포함된다! 달리 말하면 **git에 AWS secret key 가 포함 될 수 있다.** 당연히 보안에 전혀 좋지 않으므로, docs에서 복붙해온 양식에 환경변수 같은 것을 사용해서 여러분의 AWS KEY를 지키자. 또한 1.8.8 버전 기준으로 CLI 가 bucket name 을 물어보지 않는다. 이 때문에 CLI 를 사용해 `.travis.yml` 생성된 EB 옵션에는 버킷 이름이 입력 되어있지 않는데, 버킷 이름이 없으면 에러가 발생한다. 따라서 차라리 travis docs에서 제대로 된 양식을 복붙해와서 bucket name 까지 전부 입력해주자.

또한 rails 를 사용한다면 `skip_cleanup: true` 옵션도 넣어줘야한다. 이것이 활성화되어있지 않으면 travis가 Ebuild 된 것을 EB에 전달할때 asset precompile 된 것을 몽땅 날려버리고 전달하므로 deploy가 실패한다. rails 외에 angular 2도 `skip_cleanup: true` 가 활성화 되어있지 않으면 실패한다고 하는데, 다른 환경은 어떠한지 펭귄도 모르겠다. 다만 대다수의 환경이 `skip_cleanup: true` 활성화를 요구하는 걸로 보인다.

최종적으로 생성된 `.travis.yml`의 코드는 아래와 같을 것이다. 

```yml
language: ruby
rvm:
  - 2.2.5
deploy:
  skip_cleanup: true
  provider: elasticbeanstalk
  access_key_id: ENV['AWS_ACCESS_KEY']
  secret_access_key:
    secure: ENV['AWS_SECRET_KEY']
  region: ap-northeast-2
  app: application
  env: application-dev
  bucket_name: application-deploy
```

## DB 사용하기

Travis build시 사용할 DB를 설정해줘야 하는데 까다롭고, 문서에 안 나와있는 설정들이 많다. 사실 그래서 이 포스트는 travis로 배포 자동화하기가 주제지만, 이 DB 설정하기에도 많은 설명을 할애할 것이다.

### PostgreSQL

travis 의 PostgreSQL 기본 버전은 9.1 이다. 대부분의 경우 9.1에서 테스트 해도 별 문제가 없지만, 아무리 그래도 [9.1 버전은 6년전에 나온 구버전](https://www.postgresql.org/about/news/1349/)이다. 또한 psql 특정 버전에서만 지원하는 기능을 사용했을 수도 있다. 결국 9.1 버전을 교체 할 필요가 생긴다.

문서에서는 [psql 버전 바꾸는 방법을 상당히 간단하게 설명](https://docs.travis-ci.com/user/database-setup/#Using-a-different-PostgreSQL-Version)했다. 실제로도 9.4 버전까지는 저렇게 간단히 된다. 하지만 9.5 이상 버전부터는 위 방법대로 해도 에러가 발생한다. 그리고 문서에는 이 이슈에 대해 어디에도 적혀있지 않다. 결국 travis github 이슈를 뒤져보니 [psql 9.5 이상 버전을 사용 하는 방법](https://github.com/travis-ci/travis-ci/issues/4264#issuecomment-263550556)이 있었다. 요약하면 `sudo`와 `dist`를 설정하라는 것이다. 이 설정에 대한 자세한 설명은 [샤님 포스트](http://riseshia.github.io/2017/01/14/use-recent-postgresql-in-travis.html)에서 확인 할 수 있다.

psql 버전이 설정되었으면, 사용하기전에 테스트용 DB를 생성하는 script를 넣어줘서 마무리하면 된다.

최종적인 `.travis.yml` 코드는 아래와 같다.

```yml
language: ruby
rvm:
  - 2.2.5
sudo: required
dist: trusty
addons:
  postgresql: "9.6"
services:
  - postgresql
deploy:
  skip_cleanup: true
  provider: elasticbeanstalk
  access_key_id: ENV['AWS_ACCESS_KEY']
  secret_access_key:
    secure: ENV['AWS_SECRET_KEY']
  region: ap-northeast-2
  app: application
  env: application-dev
  bucket_name: application-deploy
```

### MySQL

travis MySQL 기본 버전은 5.5이다. 역시 대부분 이걸로 무난하게 쓸 수 있지만 특정 버전이 필요할 수 있다. 펭귄의 경우 MySQL 5.7 버전이 필요했지만 travis 가 5.7 버전은 공식 지원하지 않는다. 이 때문에 travis 에 MySQL 5.7을 설치 해야했다. 작성된 코드는 아래와 같다.

```yml
before_script:
  - chmod +x ./travis/*
  - ./travis/mysql-version-upgrade-5.7.sh
  - ./travis/mysql-reset-root-password.sh
  - ./travis/create-test-database.sh
```

보시다시피 `chmod` 로 권한을 먼저 부여한뒤 travis 디렉토리에 있는 스크립트들을 실행시켰다. 이 디렉토리는 펭귄이 편의상 별도로 만든 디렉토리이다. 각각의 스크립트 파일은 아래 gist 를 참고하면 된다.

 * [mysql-version-upgrade-5.7.sh](https://gist.github.com/say8425/25f534b60a7015ae5c03506306e20052#file-mysql-version-upgrade-5-7-sh)
 * [mysql-reset-root-password.sh](https://gist.github.com/say8425/25f534b60a7015ae5c03506306e20052#file-mysql-reset-root-password-sh)
 * [create-test-database.sh](https://gist.github.com/say8425/25f534b60a7015ae5c03506306e20052#file-create-test-database-sh)

MySQL 5.7 을 직접 다운 받아 업그레이드 한뒤, 꼭 root 비밀번호를 reset 해줘야한다. 안그러면 build 실패한다. 이후 psql 과 동일하게 테스트용 DB를 생성하면 된다.

## DEPLOY

여기까지 세팅한뒤 test를 통과했다면 deploy도 자동으로 진행될 것이다. 단 deploy는 기본적으로 maseter branch 에 대해서만 진행한다. 즉 다른 branch 에서 테스트가 진행되어 통과해도 deploy는 진행되지 않으며, master 가 test 되어 통과해야만 deploy 가 된다. 만약 이 설정을 바꾸고 싶다면 [on 설정](https://docs.travis-ci.com/user/deployment/#Conditional-Releases-with-on%3A)을 넣으면 된다.

## 그런데 이게 필요한가?

사실 이 기능은 필수까지는 아니다. 오히려 deploy 정책이 `All at Once` 라든가로 설정되어있어서 deploy 시 무조건 중단이 된다면 사용하기 대단히 곤란할 수 있다. 또한 test 코드가 모든 것을 완벽히 테스트 해주고 있다는 가정이 뒷받침되어야 한다. 이 때문에 오히려 대단히 번거로운 기능이 될 수도 있다. 이 기능의 도입 여부는 개발팀 환경과 성향이 충분히 고려되어야 한다고 생각한다.


---
* Related Links
  * [travis 공식 문서](https://docs.travis-ci.com)
  * [PostgreSQL 9.5 이상 버전 사용하기](http://riseshia.github.io/2017/01/14/use-recent-postgresql-in-travis.html)
  * [MySQL 5.7 버전 설치하기 gist](https://gist.github.com/say8425/25f534b60a7015ae5c03506306e20052)
  * [MySQL 5.7 버전 설치하기 gist - travis 측이 공식 메일로 제시한 방법](https://github.com/cotsog/travis_ci_prod_trusty/blob/789255e261d483a7d70d1e4a689e18f7eb56b90b/.travis.yml)
  