---
layout: single
classes: wide
title: "파이썬을 이용하여 네이버 부동산 아파트 데이터 가져오기"
categories:
    - Data 만들기
---

## 부동산 정보의 끝판왕 네이버 부동산
네이버 부동산엔 잘 찾을수 없는 정보들이 많다. 예를 들어 매달 관리비가 얼마나 나오는지, 그 부동산에서 어디 초등학교에 배정받을 수 있는지 등 부동산 데이터의 총집합이라고 볼 수 있다.
## 네이버 부동산의 무슨 데이터를 저장할 것인지?
제일먼저 https://land.naver.com/ 를 들어가 홈화면을 보자
<center><img src="/images/make_data/home.JPG" ></center>

이창에서 서울시-> 강남구->개포동을 선택하고 "확인매물보기"를 누르면 아래와 같이 사진이 뜰것이다.
<center><img src="/images/make_data/naver_land_gaepo.JPG" ></center>

우리가 원하는것은 위와같이 GUI로 표시된 정보를 컴퓨터가 읽기 편하게 Table형식으로 만드는 것 목표이다.

그렇다면 무슨 정보를 저장할 것인지는 아래 사진과 같이 단지 버튼을 누르로 LG개포자이를 선택해보고 결정하자.
<center><img src="/images/make_data/naver_land_gaepo2.JPG" ></center>
LG개포자이를 선택하면 단지정보, 시세/실거래가, 동호수/공시가격, 학군정보등의 정보들이 있다.
<center><img src="/images/make_data/naver_land_gaepo3.JPG" ></center>
아래로 내려보면 단지 내 면적별 정보가 있는데 이또한 모두 저장한다.
<center><img src="/images/make_data/naver_land_gaepo4.JPG" ></center>
이와 같이 다른 정보들도 모두 저장하여 나중에 취사 선택하는것이 좋을 것 같으니 코딩으로 들어가보자

## 네이버 부동산 긁기

아까 개포동을 선택하여 LG개포자이를 선택하기전 F12버튼을 눌러주자(크롬기준) 그러면 알수 없는 정보들이 떠있는 요상한 화면이 뜰것이다.
<center><img src="/images/make_data/crawl1.JPG" ></center>
이때 맨위에 network 탭을 누르고 아까와 같이 LG개포자이를 마우스로 클릭해보도록 하자
<center><img src="/images/make_data/crawl2.JPG" ></center>
LG개포자이를 클릭했다면 위의 사진과 같이 이상한 정보들이 주르륵 뜰것이다 여기서 preview탭을 누르고 위에서 부터 하나씩 누르면 정보가 떠있는것 같은 창이 보일것이다. 그걸 클릭하자.
<center><img src="/images/make_data/crawl3.JPG" ></center>
그다음 header탭을 클릭하여 어떤식으로 데이터를 요청하는지 보자 
<center><img src="/images/make_data/crawl4.JPG" ></center>
GET방식을 이용하여 데이터를 받는 것을 볼수 있다. 똑같이 LG개포자이 데이터를 받기위한 코드를 파이썬으로 작성해보면 아래와 같다.

{% highlight python %}
import requests
import json
down_url = 'https://new.land.naver.com/api/complexes/8928'
r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
    "Accept-Encoding": "gzip",
    "Host": "new.land.naver.com",
    "Referer": "https://new.land.naver.com/complexes/8928?ms=37.482968,127.0634,16&a=APT&b=A1&e=RETAIL",
    "Sec-Fetch-Dest": "empty",
    "Sec-Fetch-Mode": "cors",
    "Sec-Fetch-Site": "same-origin",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
})
r.encoding = "utf-8-sig"
temp=json.loads(r.text)
{% endhighlight %}

temp가 preview탭에 들어있는 정보가 동일하게 있는 것을 볼수 있다.


