---
layout: post
title: URL에서 host 추출하기
categories: rails
---

```https://www.google.com:80```

와 같은 URI에서 _google.com_ 와 같은 host를 추출해내야 했다. 어떻게 하면 될까? 가장 간단한 방법으로 루비가 지원하는 _URI_라이브러리를 사용하는 방법이 있다. 우선 _URI.parse_를 사용해 _URI_패턴으로 분석한다. 그리고 host를 추출하면 된다.

```
URI.parse('https://google.com:80').host
=> "google.com"
```

정말 심플하다. 프로토콜이나 포트번호같은 것은 제외하고 순수하게 host만 뽑아왔다. 하지만 만약 _https://www.google.com_나 _https://map.googlle.com_와 같이 _www_혹은 _map_과 같은 서브도메인이 포함되었다면 따로 제거해야한다.

```
URI.parse('https://www.google.com:80').host
=> "www.google.com"
URI.parse('https://map.google.com:80').host
=> "map.google.com"
```

보시는 바와 같이 서브도메인이 포함되었다. 이 서브도메인을 제거해야하는데, 가장 무식하고 쉬운 방법으로는 서브도메인 여부를 체크하고 [slice!](http://ruby-doc.org/core-2.2.0/String.html#method-i-slice-21)하는 방법이 있다. 

```
host = URI.parse('https://www.google.com:80').host
host.slice!('www.') if host.start_with?('www.')
host
=> "google.com"
```

하지만 서브도메인이 _www_밖에 있을리가 만무하다. 그런다고 모든 서브도메인을 체크하는 것은 미친 짓이다. 이 때문에 정규식으로 

 다른 방법이 없나 찾아보았는데 [Public Suffix List](https://publicsuffix.org)를 이용하는 방법이 있었다.

과 같은 URI에서 _google.com_만 추출하면 된다. 보통 간단하게 _URI.parse('https://www.google.com:80').host_

__
 * related links
   * http://stackoverflow.com/questions/6674230/how-would-you-parse-a-url-in-ruby-to-get-the-main-domain