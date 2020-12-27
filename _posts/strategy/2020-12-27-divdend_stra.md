---
layout: single
classes: wide
title: "파이썬으로 배당락 투자전략을 시뮬레이션 해보자! 데이터 크롤링부터 시뮬레이션까지"
categories:
    - 투자 전략
---

## 배당락 전날 투자 전략
배당락 전날 종가에 투자하면 그 다음날인 배당락일 주주명부가 확정되면서 3월 정기 주주총회에서 확정된 배당금을 배당락 입금일에 받을 수 있다.


하지만 배당락일엔 단기 투자자의 매물로 인한 배당락이 발생하며, 배당금 입금일까지 약 3개월의 공백이 있기때문에 배당락 전날에 투자를 한다는것은 정확한 계산을 해보지 않으면 수익을 정확히 알수가 없다. 


예를들어 배당락 전날 종가가 10000원이고 예상 배당이 500원이면 배당락인 다음날 시가는 9500원일 것이다. 하지만 통계적으로 배당락의 시가보다 종가가 더 높으므로 종가가 9700원이 되었다고 가정하고, 배당금이 예상과 다르게 450원으로 결정 되었다고 하자. 이때 투자자는 배당락 종가에 팔았다고 하면 투자자의 순익은 9700원*(1-0.0025)- 10000원 +450(1-0.154)=56.45원이 순익이 되는것이다.(Time Value of Money는 제외하고 계산) 이때 0.0025는 거래세 이고 0.154는 배당소득세이다. 즉 투자자의 수익률은 0.56%이고 이는 해볼만한 투자인것이다.


<center><img src="/images/dividend_stra/div_stra_ex.PNG" ></center>


이러한 투자전략은 세가지로 나눌수 있는다 아래 사진과 같다.
<center><img src="/images/dividend_stra/strategy.PNG" ></center>

위의 세가지 전략을 한국시장 20년 데이터로 시뮬레이션 해보고 통계를 내보자.

## KRX에서 일별종가 데이터와 배당데이터 긁어오기

먼저 어떤 데이터를 이용하게 될지 모르니 KRX market data에서 20년치 일별 종가를 긁어오자.
[KRX 시가총액 상위 링크](https://marketdata.krx.co.kr/mdi#document=040402)
<center><img src="/images/dividend_stra/market_data1.jpg" ></center>
위의 링크를 타고 가거나 사진처럼 들어가면 일별 시가총액 상위종목을 1위부터 꼴찌 까지 볼수 있다. 즉 이 데이터를 2000년 1월 1일까지 하나하나 손으로 받아 csv파일로 저장하면 좋겠지만 그건 불가능하다. 그래서 크롤링을 이용하여 모든 날의 데이터를 받아보자

아래 코드는 오늘을 기준으로 하여 과거 전체 데이터를 받는 코드이다.

{% highlight python %}
import pandas as pd
import numpy as np
import requests
from io import BytesIO
from datetime import datetime, timedelta
import csv
def get_daily_price(date):


    gen_otp_url = "http://marketdata.krx.co.kr/contents/COM/GenerateOTP.jspx"
    gen_otp_data = {
        'name':'fileDown',
        'filetype':'csv',
        'market_gubun': 'ALL',
        'url': 'MKD/04/0404/04040200/mkd04040200_01',
        'indx_ind_cd': '',
        'sect_tp_cd': 'ALL',
        'schdate': date,
        'pagePath': '/contents/MKD/04/0404/04040200/MKD04040200.jsp'
    }
    header={
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36',
        'x-requested-with': 'XMLHttpRequest'
    }
    r = requests.get(gen_otp_url,headers=header, params=gen_otp_data)
    
    code = r.text 
    

    down_url = 'http://file.krx.co.kr/download.jspx'
    down_data = {
        'code': code,
    }

    r = requests.post(down_url, data=down_data, headers={
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-encoding': 'gzip, deflate, br',
        'accept-language': 'ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7',
        'cache-control': 'max-age=0',
        'content-length': '417',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://marketdata.krx.co.kr',
        'referer': 'https://marketdata.krx.co.kr/',
        'sec-fetch-dest': 'iframe',
        'sec-fetch-mode': 'navigate',
        'sec-fetch-site': 'same-site',
        'sec-fetch-user': '?1',
        'upgrade-insecure-requests': '1',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36'
        
    })

    r.encoding = "utf-8-sig"
    df = pd.read_csv(BytesIO(r.content), header=0, thousands=',')
    #print(df)
    return df

for i in range(0,100000,1):
    date = (datetime.today() - timedelta(days=i)).strftime('%Y%m%d') 
    data = get_daily_price(date)
    print(i,date)
    if data.shape[0]!=0:
        data.to_csv("D:/코스피종가/"+date+".csv",encoding="CP949",index=False)
{% endhighlight %}
위의 코드는 일별 종가에 대한 코드이고 배당금 데이터는 어디서 구하나?

배당금 데이터는 아래 사진에 나온것처럼 통계->주식->투자참고->PER/PBR/배당수익률 탭에서 볼수 있다.

<center><img src="/images/dividend_stra/market_data2.jpg" ></center>


이때 2019년 기말 배당금은 2020년 5월이 지나고 알 수 있으니 2019 배당 데이터는 2020년 5월 이후의 데이터에서 가져와야한다. 
즉 아래 코드와 같이 매년 배당금 데이터를 가져올 수 있다.
{% highlight python %}
import pandas as pd
import numpy as np
import requests
from io import BytesIO
from datetime import datetime, timedelta
import csv
def get_div_price(year):
    year=str(int(year)+1)
    # STEP 01: Generate OTP
    gen_otp_url = "http://marketdata.krx.co.kr/contents/COM/GenerateOTP.jspx"
    gen_otp_data = {
        'name':'fileDown',
        'filetype':'csv',
        'ind_tp': 'ALL',
        'url': 'MKD/13/1302/13020401/mkd13020401',
        'market_gubun': 'ALL',
        'gubun': '1',
        'isu_cdnm': 'A005930/삼성전자',
        'isu_cd': 'KR7005930003',
        'isu_nm': '삼성전자',
        'isu_srt_cd': 'A005930',
        'schdate': year+'1221',
        'fromdate': year+'1216',
        'todate': year+'1220',
        'pagePath': '/contents/MKD/13/1302/13020401/MKD13020401.jsp'
    }
    header={
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36',
        'x-requested-with': 'XMLHttpRequest'
    }
    r = requests.get(gen_otp_url,headers=header, params=gen_otp_data)
    
    code = r.text # 리턴받은 값을 아래 요청의 입력으로 사용.

    # STEP 02: download
    down_url = 'http://file.krx.co.kr/download.jspx'
    down_data = {
        'code': code,
    }

    r = requests.post(down_url, data=down_data, headers={
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-encoding': 'gzip, deflate, br',
        'accept-language': 'ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7',
        'cache-control': 'max-age=0',
        'content-length': '419',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://marketdata.krx.co.kr',
        'referer': 'https://marketdata.krx.co.kr/',
        'sec-fetch-dest': 'iframe',
        'sec-fetch-mode': 'navigate',
        'sec-fetch-site': 'same-site',
        'sec-fetch-user': '?1',
        'upgrade-insecure-requests': '1',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36'
        
    })

    r.encoding = "utf-8-sig"
    df = pd.read_csv(BytesIO(r.content), header=0, thousands=',')
    #print(df)
    return df
for i in range(1995,2020,1):

    data=get_div_price(str(i))
    data.to_csv(str(i)+"배당.csv",encoding="CP949")
    print(i)
{% endhighlight %}

크롤링에 대한 설명은 다른 포스트에 자세히 적혀있으니 생략한다. (데이터를 너무 많이 받아서 KRX에서 내 IP주소를 차단한듯 하다. VPN을 이용하자!)

데이터를 모두 받았으면 다음 단계로 넘어간다!

## 배당락일 계산

현재 우리가 시뮬레이션 하기위한 데이터 포인트는

1. 배당락전날 종가
2. 배달락일 시가, 종가
3. 배당금 입금일 종가
4. 배당금

네가지 데이터를 알아야한다. 그러므로 아래와 같이 코드를 짜서 일자를 리스트로 저장하자.

{% highlight python %}

import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

arr = os.listdir("D:\코스피종가")
ex_dividend_date_eve=[]   ####배당락 전날
ex_dividend_date=[]       ####배당락일
dividend_deposit_day=[]   ####배당금 입금일
for i in range(len(arr)-100):
    if arr[i][:4]!=arr[i+3][:4] and arr[i][:4]==arr[i+1][:4] and arr[i][:4]==arr[i+2][:4]:
        ex_dividend_date_eve.append(arr[i])
        dividend_deposit_day.append(arr[i+70])
    if arr[i][:4]!=arr[i+2][:4] and arr[i][:4]==arr[i+1][:4]:
        ex_dividend_date.append(arr[i])
        
{% endhighlight %}
이때 사용한 방법은 일별 종가를 저장할때 장이 열린날만 저장하도록 했는데 배당락은 그 해의 영업일 종료 2영업일 전이다.
그러므로 위의 코드와 같이 저장하면된다.



월별 수익률은 (t시점 월말종가/t-1시점 월말종가)-1 이므로 아래와 같이 월말 인덱스를 따로 저장한다.

{% highlight python %}

{% endhighlight %}

그리고 위에서 저장한 월말 인덱스를 이용하여 각 월별 수익률을 DataFrame으로 저장하여 csv로 저장한다.

{% highlight python %}
i=0
monthly_return_data=daily_data.iloc[change_point[i+1]]/daily_data.iloc[change_point[i]]-1
monthly_return_data=(monthly_return_data).to_frame()
monthly_return_data.columns = [str(int(daily_data.index[change_point[i+1]]))[:-2]]

for i in range(1,len(change_point)-1):
    monthly_return_data2=daily_data.iloc[change_point[i+1]]/daily_data.iloc[change_point[i]]-1

    monthly_return_data2=(monthly_return_data2).to_frame()
    monthly_return_data2.columns = [str(int(daily_data.index[change_point[i+1]]))[:-2]]
    monthly_return_data=monthly_return_data.join(monthly_return_data2)
monthly_return_data=(monthly_return_data.T)
monthly_return_data.to_csv("K200_monthly_return.csv", encoding='CP949')
{% endhighlight %}

그럼 아래 사진과 같이 저장이 된다.

<center><img src="/images/make_data/monthly_return.PNG" ></center>
