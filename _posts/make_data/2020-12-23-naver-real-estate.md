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

저기서 데이터를 요청할때 Request header를 최대한 동일하게 맞춰주는게 중요하다.

Header중 Referer에 a=APT:ABYG:JGC 이부분을 APT하나로 바꿔주는게 좋다 네이버 부동산 페이지를 자세히 보면 아파트, 아파트 분양권, 재건축이 모두 선택되어 있어 아파트 정보만 얻기 위해선 APT만 선택해야하기 때문이다.

temp 에 Json형식으로 데이터를 받았다면 그다음은 필요한 데이터를 선택하는 것이 중요하다.
<center><img src="/images/make_data/crawl5.JPG" ></center>
위의 사진을 보면 complexDetail 안에 단지정보가 포함돼있는것을 볼수 있다.
<center><img src="/images/make_data/crawl6.JPG" ></center>
이 데이터를 LG개포자이라는 index에 면적별로 컬럼으로 저장하려면 아래와 같이 코드를 짤수 있다.
{% highlight python %}
import pandas as pd
apt_data=temp["complexDetail"]
pyoeng_list=apt_data["pyoengNames"].split(", ")
apt_data_pd=pd.DataFrame(data=apt_data,index=range(len(pyoeng_list)))
for i in range(len(pyoeng_list)):
    apt_data_pd.loc[i,"pyoengNames"]=pyoeng_list[i]
apt_data_pd
{% endhighlight %}
위와 같이 코딩을 하면 "pyoengNames"안에 들어있던 평수리스트가 하나씩 분리가 된다.
pyoeng_list로 따로 변수명을 선언한 이유는 저 순서대로 데이터를 긁을 필요가 많기때문에 따로 설정하였다.
결과는 아래와 같이 dataframe으로 깔끔하게 저장이 된다.
<center><img src="/images/make_data/crawl7.JPG" ></center>

다른 정보들도 위의 플로우를 따라서 비슷하게 긁어서 apt_data_pd 변수에 저장된 dataframe에 join함수를 이용하여 합치면 된다.

## 다른 아파트 정보를 자동으로 가져오려면?
위의 크롤링 예시는 LG개포자만을 긁는 코드이다. 하지만 데이터 분석을 위해선 아파트 하나하나를 직접쳐 가져올수가 없다.
네이버 부동산의 아파트,오피스텔 탭을 다시 클릭하여 지도로 돌아가서 f12를 다시 누르자
<center><img src="/images/make_data/crawl8.JPG" ></center>

그리고 서울시를 클릭하면 list?contarNo=000000이라는 파일이 Name탭에 표시될것이다 그걸 클릭하여 preview를보자
<center><img src="/images/make_data/crawl9.JPG" ></center>

여기에 contarNo라는 변수가 서울시를 표시하는 코드인것이다.
또 여기서 서울시를 클릭하면 아래와 같이 시/군/구의 코드를 볼수 있다.
<center><img src="/images/make_data/crawl10.JPG" ></center>
이때 Request URL에 contarNo가 위의위 사진에 있는 서울시의 contarNo인것을 볼수 있다. 

즉 서울시를 선택하면 서울시의 구들이 list형식으로 표시되고 이때 또 구를 선택하면 동이 list형식으로 반환되고 동을 선택하면 아파트들이 list형식으로 반환되는 형식인것이다.

예를들어 LG개포자이를 선택하려면 서울시, 강남구, 개포동, LG개포자이의 고유 넘버를 다 알아야 정보를 얻을수 있는 것이다. 이를 코드로 표현하면 아래와 같다.
{% highlight python %}
def get_sido_info():
    down_url = 'https://new.land.naver.com/api/regions/list?cortarNo=0000000000'
    r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/102378?ms=37.5018495,127.0438028,16&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp=json.loads(r.text)
    temp=list(pd.DataFrame(temp["regionList"])["cortarNo"])
    return temp
def get_gungu_info(sido_code):
    down_url = 'https://new.land.naver.com/api/regions/list?cortarNo='+sido_code
    r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/102378?ms=37.5018495,127.0438028,16&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp=json.loads(r.text)
    temp=list(pd.DataFrame(temp['regionList'])["cortarNo"])
    return temp
def get_dong_info(gungu_code):
    down_url = 'https://new.land.naver.com/api/regions/list?cortarNo='+gungu_code
    r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/102378?ms=37.5018495,127.0438028,16&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp=json.loads(r.text)
    temp=list(pd.DataFrame(temp['regionList'])["cortarNo"])
    return temp
def get_apt_list(dong_code):
    down_url = 'https://new.land.naver.com/api/regions/complexes?cortarNo='+dong_code+'&realEstateType=APT&order='
    r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/102378?ms=37.5018495,127.0438028,16&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp=json.loads(r.text)
    try:
        temp=list(pd.DataFrame(temp['complexList'])["complexNo"])
    except:
        temp=[]
    return temp

{% endhighlight %}

get_sido_info()의 함수는 서울시, 경기도, 부산시...등의 특별시와 도의 고유코드를 list 형식으로 return 해주는 함수이다.
get_gungu_info(sido_code)함수는 get_sido_info()에서 return받은 리스트의 값중 하나를 함수 인자로 넣으면 시/군/구의 고유코드를 list 형식으로 return 해주는 함수이다.
get_dong_info(gungu_code)함수는 get_gungu_info(sido_code)에서 return받은 리스트의 값중 하나를 함수 인자로 넣으면 읍/면/동의 고유코드를 list 형식으로 return 해주는 함수이다.
get_apt_list(dong_code)함수는 get_dong_info(gungu_code)에서 return받은 리스트의 값중 하나를 함수 인자로 넣으면 아파트의 고유코드를 list 형식으로 return 해주는 함수이다.

즉 LG개포자이의 고유 코드는 아래와 같이 구할수 있는 것이다.
{% highlight python %}
sido_list=get_sido_info() 
gungu_list=get_gungu_info(sido_list[0])
dong_list=get_dong_info(gungu_list[0])
get_apt_list(dong_list[0])[0]
{% endhighlight %}
<center><img src="/images/make_data/crawl11.JPG" ></center>

위의 사진처럼 LG개포자이의 고유 코드는 8928이고 이는 LG개포자이의 정보를 GET함수를 이용하여 구할때 코드안에 포함된것을 볼수 있다.(아래 코드)

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
이를 8298을 변수로 바꾸고 함수화 시키면 아래와 같이 표현할 수 있다.
{% highlight python %}
def get_apt_info(apt_code):
    down_url = 'https://new.land.naver.com/api/complexes/'+apt_code+'?sameAddressGroup=false'
    r = requests.get(down_url,data={"sameAddressGroup":"false"},headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/"+apt_code+"?ms=37.482968,127.0634,16&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp=json.loads(r.text)
    return temp
{% endhighlight %}

위의 코드들을 이용하여 네이버 부동산의 코드를 긁을수 있다. 아래의 코드는 학군정보와 아파트 가격정보를 가져오는 함수이다.

{% highlight python %}
def get_school_info(apt_code):
    down_url = 'https://new.land.naver.com/api/complexes/'+apt_code+'/schools'
    r = requests.get(down_url,headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/"+apt_code+"?ms=37.482968,127.0634,16&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp_school=json.loads(r.text)
    return temp_school
##################가격정보
def apt_price(apt_code,index):
    p_num=temp["complexPyeongDetailList"][index]["pyeongNo"]
    down_url = 'https://new.land.naver.com/api/complexes/'+apt_code+'/prices?complexNo='+apt_code+'&tradeType=A1&year=5&priceChartChange=true&areaNo='+p_num+'&areaChange=true&type=table'

    r = requests.get(down_url,headers={
        "Accept-Encoding": "gzip",
        "Host": "new.land.naver.com",
        "Referer": "https://new.land.naver.com/complexes/"+apt_code+"?ms=37.4830877,127.0579863,15&a=APT&b=A1&e=RETAIL",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "same-origin",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"
    })
    r.encoding = "utf-8-sig"
    temp_price=json.loads(r.text)
    return temp_price
{% endhighlight %}
get_school_info(apt_code) 함수는 apt_code에 아파트 고유 코드를 str형태로 넣으면 학군정보를 return해준다.
apt_price(apt_code,index) 함수는 apt_code에 아파트 고유 코드를 str형태로 넣고, index에 위에 선언한 pyoeng_list의 인덱스를 넣으면 가격정보를 return해준다.

아래는 내가 전국 아파트의 정보를 긁은 코드이다. 참고하실분은 참고하면된다.
{% highlight python %}
sido_list=get_sido_info()
for m in range(len(sido_list)):
    gungu_list=get_gungu_info(sido_list[m])
    gungu_apt_list=[0]*len(gungu_list)
    for j in range(len(gungu_list)):#구 마다 하나씩 저장
        dong_list=get_dong_info(gungu_list[j])
        dong_apt_list=[0]*len(dong_list)
        for k in range(len(dong_list)):#동마다 하나씩 저장
            apt_list=get_apt_list(dong_list[k])
            apt_list_data=[0]*len(apt_list)
            for n in range(len(apt_list)):#아파트 마다 하나씩 저장
                temp=get_apt_info(apt_list[n])
                try:
                    area_list=temp["complexDetail"]["pyoengNames"].split(", ")
                    ex_flag=1
                except KeyError:   
                    ex_flag=0
                    temp_data=pd.DataFrame(columns=temp_data.columns)
                if ex_flag==1:
                    temp_school=get_school_info(apt_list[n])
                    temp_data=pd.DataFrame(index=range(len(area_list)))
                    for i in range(len(area_list)):
                        print(temp["complexDetail"]["address"],temp["complexDetail"]["complexName"])
                        temp_data.loc[i,"아파트명"]=temp["complexDetail"]["complexName"]
                        temp_data.loc[i,"면적"]=area_list[i]
                        temp_data.loc[i,"법정동주소"]=temp["complexDetail"]["address"]+" "+temp["complexDetail"]["detailAddress"]
                        try:
                            temp_data.loc[i,"도로명주소"]=temp["complexDetail"]["roadAddressPrefix"]+" "+temp["complexDetail"]["roadAddress"]
                        except KeyError:
                            temp_data.loc[i,"도로명주소"]=temp["complexDetail"]["roadAddressPrefix"]
                        temp_data.loc[i,"latitude"]=temp["complexDetail"]["latitude"]
                        temp_data.loc[i,"longitude"]=temp["complexDetail"]["longitude"]
                        temp_data.loc[i,"세대수"]=temp["complexDetail"]["totalHouseholdCount"]
                        temp_data.loc[i,"임대세대수"]=temp["complexDetail"]["totalLeaseHouseholdCount"]
                        temp_data.loc[i,"최고층"]=temp["complexDetail"]["highFloor"]
                        temp_data.loc[i,"최저층"]=temp["complexDetail"]["lowFloor"]
                        try:
                            temp_data.loc[i,"용적률"]=temp["complexDetail"]["batlRatio"]
                        except KeyError:
                            temp_data.loc[i,"용적률"]=""
                        try:
                            temp_data.loc[i,"건폐율"]=temp["complexDetail"]["btlRatio"]
                        except KeyError:
                            temp_data.loc[i,"건폐율"]=""
                        try:
                            temp_data.loc[i,"주차대수"]=temp["complexDetail"]["parkingPossibleCount"]
                        except KeyError:
                            temp_data.loc[i,"주차대수"]=""
                        try:
                            temp_data.loc[i,"건설사"]=temp["complexDetail"]["constructionCompanyName"]
                        except KeyError:   
                            temp_data.loc[i,"건설사"]=""
                        try:
                            temp_data.loc[i,"난방"]=temp["complexDetail"]["heatMethodTypeCode"]
                        except KeyError:   
                            temp_data.loc[i,"난방"]=""
                        try:
                            temp_data.loc[i,"공급면적"]=temp["complexPyeongDetailList"][i]["supplyArea"]
                        except KeyError:   
                            temp_data.loc[i,"공급면적"]=""
                        try:
                            temp_data.loc[i,"전용면적"]=temp["complexPyeongDetailList"][i]["exclusiveArea"]
                        except KeyError:   
                            temp_data.loc[i,"전용면적"]=""
                        try:
                            temp_data.loc[i,"전용율"]=temp["complexPyeongDetailList"][i]["exclusiveRate"]
                        except KeyError:   
                            temp_data.loc[i,"전용율"]=""
                        try:
                            temp_data.loc[i,"방수"]=temp["complexPyeongDetailList"][i]["roomCnt"]
                        except KeyError:   
                            temp_data.loc[i,"방수"]=""
                        try:
                            temp_data.loc[i,"욕실수"]=temp["complexPyeongDetailList"][i]["bathroomCnt"]
                        except KeyError:   
                            temp_data.loc[i,"욕실수"]=""
                        try:
                            temp_data.loc[i,"해당면적_세대수"]=temp["complexPyeongDetailList"][i]["householdCountByPyeong"]
                        except KeyError:   
                            temp_data.loc[i,"해당면적_세대수"]=""
                        try:
                            temp_data.loc[i,"현관구조"]=temp["complexPyeongDetailList"][i]["entranceType"]
                        except KeyError:   
                            temp_data.loc[i,"현관구조"]=""
                        try:
                            temp_data.loc[i,"재산세"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["propertyTax"]
                        except KeyError:   
                            temp_data.loc[i,"재산세"]=""
                        try:
                            temp_data.loc[i,"재산세합계"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["propertyTotalTax"]
                        except KeyError:   
                            temp_data.loc[i,"재산세합계"]=""
                        try:
                            temp_data.loc[i,"지방교육세"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["localEduTax"]
                        except KeyError:   
                            temp_data.loc[i,"지방교육세"]=""
                        try:
                            temp_data.loc[i,"재산세_도시지역분"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["cityAreaTax"]
                        except KeyError:   
                            temp_data.loc[i,"재산세_도시지역분"]=""
                        try:
                            temp_data.loc[i,"종합부동산세"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["realEstateTotalTax"]
                        except KeyError:   
                            temp_data.loc[i,"종합부동산세"]=""
                        try:
                            temp_data.loc[i,"결정세액"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["decisionTax"]
                        except KeyError:   
                            temp_data.loc[i,"결정세액"]=""
                        try:
                            temp_data.loc[i,"농어촌특별세"]=temp["complexPyeongDetailList"][i]["landPriceMaxByPtp"]["landPriceTax"]["ruralSpecialTax"]
                        except KeyError:   
                            temp_data.loc[i,"농어촌특별세"]=""    

                        temp_price=apt_price(apt_list[0],i)
                        try:
                            temp_data.loc[i,"가격"]=temp_price["marketPrices"][0]["dealAveragePrice"]
                        except KeyError:   
                            temp_data.loc[i,"가격"]=""
                        try:
                            temp_data.loc[i,"겨울관리비"]=temp["complexPyeongDetailList"][i]["averageMaintenanceCost"]["winterTotalPrice"]
                        except KeyError:   
                            temp_data.loc[i,"겨울관리비"]=""
                        try:
                            temp_data.loc[i,"여름관리비"]=temp["complexPyeongDetailList"][i]["averageMaintenanceCost"]["summerTotalPrice"]
                        except KeyError:   
                            temp_data.loc[i,"여름관리비"]=""
                        try:
                            temp_data.loc[i,"매매호가"]=temp["complexPyeongDetailList"][i]["articleStatistics"]["dealPriceString"]
                        except KeyError:   
                            temp_data.loc[i,"매매호가"]=""
                        try:
                            temp_data.loc[i,"전세호가"]=temp["complexPyeongDetailList"][i]["articleStatistics"]["leasePriceString"]
                        except KeyError:   
                            temp_data.loc[i,"전세호가"]=""
                        try:
                            temp_data.loc[i,"월세호가"]=temp["complexPyeongDetailList"][i]["articleStatistics"]["rentPriceString"]
                        except KeyError:   
                            temp_data.loc[i,"월세호가"]=""
                        try:
                            temp_data.loc[i,"실거래가"]=temp["complexPyeongDetailList"][i]["articleStatistics"]["rentPriceString"]
                        except KeyError:   
                            temp_data.loc[i,"실거래가"]=""
                        try:
                            temp_data.loc[i,"초등학교_학군정보"]=temp_school['schools'][0]["schoolName"]
                        except KeyError:   
                            temp_data.loc[i,"초등학교_학군정보"]=""
                        except IndexError :   
                            temp_data.loc[i,"초등학교_학군정보"]=""
                        try:
                            temp_data.loc[i,"초등학교_설립정보"]=temp_school['schools'][0]["organizationType"]
                        except KeyError:   
                            temp_data.loc[i,"초등학교_설립정보"]=""
                        except IndexError :   
                            temp_data.loc[i,"초등학교_설립정보"]=""
                        try:
                            temp_data.loc[i,"초등학교_남학생수"]=temp_school['schools'][0]["maleStudentCount"]
                        except KeyError:   
                            temp_data.loc[i,"초등학교_남학생수"]=""
                        except IndexError :   
                            temp_data.loc[i,"초등학교_남학생수"]=""
                        try:
                            temp_data.loc[i,"초등학교_여학생수"]=temp_school['schools'][0]["femaleStudentCount"]
                        except KeyError:   
                            temp_data.loc[i,"초등학교_여학생수"]=""
                        except IndexError :   
                            temp_data.loc[i,"초등학교_여학생수"]=""

                    #time.sleep(1)
                apt_list_data[n]=temp_data
            if apt_list_data==[]:
                dong_apt_list[k]=pd.DataFrame(columns=temp_data.columns)
            else:
                dong_apt_list[k]=pd.concat(apt_list_data)
        gungu_apt_list[j]=pd.concat(dong_apt_list)
        gungu_apt_list[j].to_csv(temp["complexDetail"]["roadAddressPrefix"]+".csv",encoding="CP949")
    final_data=pd.concat(gungu_apt_list)
    final_data.to_csv(temp["complexDetail"]["roadAddressPrefix"].split()[0]+".csv",encoding="CP949")
{% endhighlight %}

시도, 시군구, 동면읍 아파트 별로 빈 리스트를 만들어 그 리스트 안에 DataFrame으로 저장하고 나중에 한꺼번에 concat하는 원리이다.
네이버 부동산에서 빈데이터가 return되면 멈추는 현상이 있어 데이터 저장 시점마다 try문을 넣어 빈칸을 저장하도록 하였다.
