---
layout: post
title: bundle 설치된 gem 삭제하기
categories: rails
---

## 개요
대개의 gem파일든은 _Gemfile.rb_에서 딜릿하는 것만으로 깔끔하게 끝낼 수 있다. 하지만 _devise_나 _bootsy_와 같이 _model_이나 _config_의 설치를 요구하는 gem들은 삭제하기 정말 골룸하다. 수동으로 직접 딜릿하는 방법도 있지만, 위험하고 무엇보다 **귀찮다!** 이 때문에 조금이나마 자동화 된 방법을 소개한다.

## 방법
### table 날리기
우선 migration을 통해 설치된 table을 제거한다. _bootsy_를 예로 들자면, 총 2개의 migrtion이 설치된다. migrate시 _bootsy_images_와 _bootsy_image_galleries_테이블이 설치된다.

```ruby
# This migration comes from bootsy (originally 20120624171333)
class CreateBootsyImages < ActiveRecord::Migration
  def change
    create_table :bootsy_images do |t|
      t.string :image_file
      t.references :image_gallery
      t.timestamps
    end
  end
end

# This migration comes from bootsy (originally 20120628124845)
class CreateBootsyImageGalleries < ActiveRecord::Migration
  def change
    create_table :bootsy_image_galleries do |t|
      t.references :bootsy_resource, polymorphic: true
      t.timestamps
    end
  end
end
```

우선 이 테이블들을 _drop_해주는 migration을 만들어준다.

```
rails g migration DropBootsy
```

그리고 table을 drop해주는 소스를 코딩하자.

```ruby
# VERSION_remove_bootsy.rb
class RemoveBootsy < ActiveRecord::Migration
  def change
    drop_table :bootsy_image_galleries
    drop_table :bootsy_images
  end
end
```

이후 _rake db:migrate_해준다. 그러면 소스대로 _bootsy_image_galleries_과 _bootsy_images_ 테이블이 drop된다.

### config와 model 박멸하기
사실 여기까지는 별로 어려운 것도 아니고 보편적인 방법이다. config와 model을 삭제하는 일이 문제인데, 이곳저곳 꽁꽁 숨어있어서, 일일이 딜릿해주다가 놓치는 경우가 많다. 이 작업을 자동으로 해주는 명령어는 아쉽게도 없다. 하지만 설치한 것을 역으로 파괴하는 명령어는 있다.

가령 bootsy를 예로 들자면 설치시 _rails generate bootsy:install_ 명령어를 입력한다. 이와 반대로 uninstall 명령을 해서 해결했으면 좋겠는데, 안타깝게도 rails에 이런 명령어는 없다. 대신 _rails destroy bootsy:install_라는 *파괴적인* 명령어가 있다.

```ruby
pengShip-II:Leferi Penguin$ rails destroy bootsy:install
       route  mount Bootsy::Engine => '/bootsy', as: 'bootsy'
      remove  config/locales/bootsy.en.yml
     skipped  insert into app/assets/javascripts/application.js
   not found  app/assets/stylesheets/application.css not found. You must
            manually require Bootsy in your assets pipeline.
      remove  config/initializers/bootsy.rb
```

실행하면 _rails generate bootsy:install_로 생성된 파일이나 _routes.rb_나 _application.css_등에 삽입한 코드들을 삭제한다. 가끔 log와 같이 _not found_가 뜨는 경우가 있는데, 유저가 진즉에 지웠거나 파일 이름이 바껴서 찾지 못하는 거일 수 있다. 로그를 보고 추정되는 파일을 직접 확인하면 된다.

### 번외:migration 생성 파일은 destroy 하면 안된다
가끔 설치 명령어에 migration 파일 생성도 같이 있는 경우가 있다. 이 때문에 destroy시 migration도 같이 파괴당해 table drop을 migration 할 수 없는 경우가 생긴다. 이 때문에 log를 보고 migraion도 같이 파괴됐다면 살려주자. migration은 함부로 삭제하면 안된다. 삭제하면 유지보수가 힘든 것은 물론, 퍼블리싱시 에러가 발생 할 수 있으므로 가급적 삭제하지 말자.

## 결론
앞서 말했듯이 복잡한 명령어 없이 수동으로 직접 딜릿딜릿 해줄 수 있지만, 하지만 그러기에는 생성되는 파일들이 많아 한두개 빼먹을 수 있다. 이 때문에 훗날 에러가 발생해 삽질 할 수 있다. 따라서 수동으로 직접 해주는 것보다 명령어를 통해서 자동으로 해줄 것을 추천한다.

---
* Related Links
  * [참고한 StackOverflow 문서](http://stackoverflow.com/a/26219130/3910390)