---
layout: post
title: CSV를 utf-8 with BOM으로 내보내기
categories: rails
---

## 개요
레일즈에서 테이블을 csv로 내보내기 함수를 만들어 잘 사용하고 있었는데, 최근에 이 csv를 엑셀에서 읽으면 한글이 깨지는 이슈를 발견했다.

```ruby
def to_csv
  CSV.generate do |csv|
    csv << column_names
    all.each do |result|
      csv << result.attributes.values_at(*column_names)
    end
  end
end

send_data scope.to_csv,
          filename: name.concat('.csv'),
          type: 'text/csv',
          disposition: 'attachment'
```

이 CSV는 기본적으로 UTF-8 형식으로 내보내지고 있는데, 엑셀에서는 엉뚱하게도 일본어로 출력되었다.

## BOM이 뭐지요,,?
구글링 해보니 이런 [문서](https://www.skoumal.net/en/making-utf-8-csv-excel/)가 있다. 문서에는 이런 말이 있었다.

> Excel default encoding depends on the system. The workaround is to put three magical bytes to the file beginning. They are called BOM (Byte order mark) and say to the editor that file is encoded as UTF-8.

대충 이런 내용이다. 엑셀은 기본적으로 시스템 인코딩을 따른다. 그리고 BOM이라는 ~~신비한~~ 바이트 3개를 파일이 시작될 때 넣어줘야 UTF-8이라고 인지한다.

## BOM을 심어주자!
그런면 BOM을 심어주자. 찾아보니 이미 [열도국에서 누군가 시도](http://qiita.com/necojackarc/items/5e865a4aa039fdd3f2c0#実装)했다.

```
require "csv"

bom = %w(EF BB BF).map { |e| e.hex.chr }.join
CSV.generate(bom) do |csv|
  ...
end
```

`bom = %w(EF BB BF).map { |e| e.hex.chr }.join`은 앞서 말한 `BOM`이다. `EF BB BF`은 아스키코드화 되어서 `"\xEF\xBB\xBF"`로 조합한다. 그리고 코드에서 이 `bom`은 `generate` 함수에 들어갔다. 저것이 무슨 의미인가 [문서](https://docs.ruby-lang.org/en/2.1.0/CSV.html#method-c-generate)에서 찾아보았다. 대충 해석하면 이러하다. 생성된 block앞에 붙인다는 얘기이다. 거꾸로 말해서, block이 이 파라미터로 시작한다는 얘기이다.

## 실제로 출력해보자

```=> "\xEF\xBB\xBFid,host,port,...```

실제로 출력해보니 위와 같이 BOM으로 시작하고 있다는 것을 확인 할 수 있었다. 결과물의 인코딩 또한 확인해봤다. 펭귄은 [서브라임에서 확인](http://stackoverflow.com/questions/16195871/how-do-i-see-the-current-encoding-of-a-file-in-sublime-text-2/16199148#16199148)했는데, 서브라임의 콘솔창을 열고 `view.encoding()`라는 명령어를 치면 된다. 보기 좋게 `'UTF-8 with BOM'`이라는 결과물을 볼 수 있을 것이다. 그리고 엑셀로 실행해보면 언제 그랬냐는 듯이 한글을 정상적으로 보여줄 것이다.

---
* Related Links
	* [엑셀은 BOM이 필요하다는 것을 설명한 문서](https://www.skoumal.net/en/making-utf-8-csv-excel/)
	* [열도국에서 누군가 BOM을 시도한 문서](http://qiita.com/necojackarc/items/5e865a4aa039fdd3f2c0#実装)
	* [SO에서 설명한 문서](http://stackoverflow.com/a/19678391/3910390)
	* [서브라임에서 현재 인코딩 확인하는 방법에 대한 SO문서](http://stackoverflow.com/a/16199148/3910390)
