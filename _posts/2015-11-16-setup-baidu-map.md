---
layout: post
title: 바이두 지도 API 사용하기
categories: front
---

## 개요
중국지도를 보려면 구글 맵이 아닌 바이두 맵이 필요하다. 물론 구글 맵으로도 중국지도를 보는 것은 가능하다. 하지만 좌표나 주소 검색이 중국내에서는 불가능하다. 이 때문에 중국지도를 임배디드하려면, 바이두 API가 필요하다. 하지만 온갖 한자로 장식된 바이두를 가입하는 것부터 난관인데, 기것 가입했더니 인증을 위한 핸드폰 번호가 필요하다. 이 때문에 이런 구질구질한 절차 말고 다른 방법이 필요하다.

다행히도 1.5이하 버전 API는 API키를 요구하지 않는다. 이점을 이용해서 구버전 스크립트로 지도를 불러오면 된다. 펭귄이 참고한 [문서](https://github.com/jiazheng/Baidu-Map-API-Learning)는 1.2버전을 사용하였다. 따라서 본 문서도 1.2버전을 기반으로 설명하겠다. 1.3버전도 상관없는 것처럼 보인다. 하지만 1.4버전에는 zooming 버그가 있다. 이것에 대해서는 본문에서 후술하겠다.

## Setup하기

```html
<script src="http://api.map.baidu.com/api?v=1.2"></script>
```

우선 Baidu Map API불러온다. 버전은 앞서 말한대로 1.2버전을 넣는다.

```js
// 바이두 지도 API
// 중국 지도
var chinaMap = new BMap.Map('chinaSQ_Map');
chinaMap.centerAndZoom(new BMap.Point(113.941477, 22.52946), 18);
var marker = new BMap.Marker(new BMap.Point(113.941477, 22.52946));
chinaMap.addOverlay(marker);
```

Google Map API에 비하면 정말 단촐하다. _new BMap.Map()_에 지도가 붙여지길 원하는 div의 ID셀렉터를 넣어주고, [여기서](http://api.map.baidu.com/lbsapi/getpoint/) 얻은 좌표값을 _centerAndZoom_에 넣어주자. 두번째 파라미터는 이름 그대로 zoom 배율이다. 구글 API와 똑같이 마커를 넣어주려면 marker에도 좌표를 넣어서 지도에 _addOverlay()_로 덧붙이면 된다. 귀찮으면 _new BMap.Point_를 변수로 선언해도 좋다. 그러면 끝! 이 아니었다.

## 여긴 어디? 난 누구?
![이상]({{ site.baseurl }}/images/2015-11-16-setup-baidu-map/above.png)
![현실]({{ site.baseurl }}/images/2015-11-16-setup-baidu-map/actual.png)

실행해보면 알겠지만 위 이미지와 같이 마커가 안보인다. 지도도 여긴 어딘지, 엉뚱한 좌표를 보여주고 있다. 엄밀히는 마커는 해당 좌표에 찍혔는데, 지도의 center가 설정된 좌표와 달리 조금 동떨어진 곳에 설정되서 이런 촌극이 발생한 것이다.

지도를 이리저리 드래그해보면 해당 좌표에 마커가 찍힌 것을 확인 할 수 있다. 그리고 이 오차값만큼 center의 좌표값을 직접 수치를 바꿔가면서 맞춰야한다. *미친짓*인거는 알겠는데, 다른 방법을 찾지 못했다. 참고로 저 오차값은 무엇을 기반으로 값이 달라지는지, 펭귄이 작업중인 페이지와 테스트용으로 만든 페이지 모두 오차값이 달랐다.

그나마 1.4버전에서는 이 버그가 해결되는데, 이번에는 zoom을 14이상으로 넣어주면 center와 mark 모두 찍어준 좌표와 달리 엉뚱한 곳을 보여줘서 쓸 수가 없다. 펭귄의 경우 지도가 망망대해를 보여주었다.

---
* Related Links
  * [공식문서](http://developer.baidu.com/map/index.php?title=jspopular)
  * [중국내 좌표를 얻을 수 있는 바이두 지도 페이지](http://api.map.baidu.com/lbsapi/getpoint/)
  * [바이두에서 사용되는 함수](http://bbs.lbsyun.baidu.com/forum.php?mod=viewthread&tid=3498) : 몇버전 기준인지는 알 수가 없다
  * [Baidu Map API Learning](https://github.com/jiazheng/Baidu-Map-API-Learning) : 소스만 덩그러니 있지만 어떤 분위기인지 파악하기 좋다