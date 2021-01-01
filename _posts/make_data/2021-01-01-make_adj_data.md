---
layout: single
classes: wide
title: "파이썬과 KRX marketdata를 이용한 수정주가 산출(코스피 코스닥 전종목 상장폐지종목 포함)"
categories:
    - Data 만들기
---

## 수정주가란?
수정 주가란 유 · 무상증자，액면분할 등에 따른 주식 수 변화를 반영해 비교할 수 있는 주가다. 예를들어 삼성전자는 액면분할을하여 주식수가 50배 늘어난 대신 주가는 1/50으로 조정해주어 현재 주가기준으로 과거에 얼마였는지 알수 있는 지표이다.

장이 개장하기전 한국거래소는 주식 수 변화가 있는 종목의 기준가를 산정하여 당일 거래가 시작된다. 즉 이벤트가 없는 종목은 전일종가/당일기초가 비율은 1일것이고 삼성전자의 경우 액면분할 이벤트가 있는 날엔 전일종가/당일기초가는 50이 될것이다. 가격이 1/50이 되었기 때문이다. KRX marketdata는 1995년5월8일까지 과거 전종목 주가를 크롤링 할수있게 해놓았다. 

[KRX maketdata 크롤링 링크](https://leesunkyu94.github.io/%ED%88%AC%EC%9E%90%20%EC%A0%84%EB%9E%B5/divdend_stra/#)

위의 링크를 참고하여 크롤링 하면 되지만 친절하게 아래 두가지 코드를 올려두겠다. 아래 두 코드는 코스피 코스닥 전종목 종가와 기준가를 받는 코드이다.(VPN을 써서 받자 전 너무 많이 요청을해서 차단됐다;)

{% highlight python %}
import pandas as pd
import numpy as np
import requests
from io import BytesIO
from datetime import datetime, timedelta
import csv
def get_daily_price(date):

    # STEP 01: Generate OTP
    gen_otp_url = "http://marketdata.krx.co.kr/contents/COM/GenerateOTP.jspx"
    gen_otp_data = {
        'name':'fileDown',
        'filetype':'csv',
        'ind_tp': 'ALL',
        'url': 'MKD/13/1302/13020102/mkd13020102',
        'period_strt_dd': date,
        'period_end_dd': date,
        'pagePath': '/contents/MKD/13/1302/13020102/MKD13020102.jsp'
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

for i in range(100000):
    date = (datetime.today() - timedelta(days=i)).strftime('%Y%m%d') 
    data = get_daily_price(date)
    print(i,date)
    if data.shape[0]!=0:
        data.to_csv("D:/코스피기준가/"+date+".csv",encoding="CP949",index=False)

{% endhighlight %}
위의 코드는 주식기준가를 일별로 받는 코드이다.

{% highlight python %}
import pandas as pd
import numpy as np
import requests
from io import BytesIO
from datetime import datetime, timedelta
import csv
def get_daily_price(date):

    # STEP 01: Generate OTP
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
위의 코드는 주식종가데이터를 받는 코드이다.

위 두가지 코드를 VPN을 꼭 사용하여 10시간정도 돌려주면 1995년5월8일까지 약 6500거래일의 데이터를 받을 수 있다.

## 수정주가 계산
위의 수정주가 로직을 로직으로 표현하면 아래 사진과 같다.
<center><img src="/images/make_data/sudo_code.png" ></center>

Cummulative ratio인 이유는 이벤트가 하나만 있는것이 아니고 누적으로 여러가지 있을 수 있기 때문이다. 즉 하루하루 Ratio를 계산하여 매일 곱해주어야 한다는 뜻이다.

아래는 위의 로직을 위에서 받은 데이터로 구현한 것이다.
{% highlight python %}

#####최초 ratio 만들기
ex_last_price=pd.read_csv("D:\코스피종가\\"+arr[-2],encoding="CP949",index_col="종목코드")
ex_last_price=index_to_6digit(ex_last_price)
base_price=pd.read_csv("D:\코스피기준가\\"+arr[-1],encoding="CP949",index_col="종목코드")
base_price=index_to_6digit(base_price)
final_result_data=[0]*len(arr)
final_ratio=[0]*len(arr)

ratio=ex_last_price["현재가"].to_frame().join(base_price["시작일기준가(원)"],how="outer")
ratio=ratio.fillna(1)### 신규 상장 1로 설정
ratio=ratio["시작일기준가(원)"]/ratio["현재가"]
ratio.name="ratio"

last_price=pd.read_csv("D:\코스피종가\\"+arr[-2],encoding="CP949",index_col="종목코드")
last_price=index_to_6digit(last_price)
last_price=last_price.join(ratio,how="outer")
last_price["현재가"]=last_price["현재가"]*last_price["ratio"]

result_data=last_price["현재가"].to_frame().T
result_data.index=[int(arr[-1][:8])]
final_result_data[0]=result_data

temp=last_price["ratio"].to_frame().T
temp.index=[int(arr[-1][:8])]
final_ratio[0]=temp

ratio_data=ratio.to_frame().T
ratio_data.index=[int(arr[-1][:8])]


for i in range(len(arr)-2):
    ############# raio 계산
    ex_last_price=pd.read_csv("D:\코스피종가\\"+arr[-(i+2)],encoding="CP949",index_col="종목코드")
    ex_last_price=index_to_6digit(ex_last_price)
    base_price=pd.read_csv("D:\코스피기준가\\"+arr[-(i+1)],encoding="CP949",index_col="종목코드")
    base_price=index_to_6digit(base_price)
    #전날종가와 당일 기준가 join 전일종가는 nan 값 가능
    ratio2=ex_last_price["현재가"].to_frame().join(base_price["시작일기준가(원)"],how="outer") 
    
    ### 전날 계산된 ratio 병합 ratio가 nan이면 신규상장 -> 1로 변경
    ratio2=ratio2.join(ratio,how="outer")
    ratio2["ratio"]=ratio2["ratio"].fillna(1)
    ratio=(ratio2["시작일기준가(원)"]/ratio2["현재가"]).fillna(1)*ratio2["ratio"]
    ratio.name="ratio"
    
    ############# 비율 계산
    last_price2=pd.read_csv("D:\코스피종가\\"+arr[-(i+2)],encoding="CP949",index_col="종목코드")
    last_price2=index_to_6digit(last_price2)
    last_price2=last_price2.join(ratio,how="outer")
    
    last_price2["현재가"]=last_price2["현재가"]*last_price2["ratio"]#-last_price2["누적배당금"]

    temp=last_price2["ratio"].to_frame().T
    temp.index=[int(arr[-(i+2)][:8])]
    final_ratio[i+1]=temp
    
    result_data2=last_price2["현재가"].to_frame().T
    result_data2.index=[int(arr[-(i+2)][:8])]
    final_result_data[i+1]=result_data2

    final_ratio[i+1]
    print(i,arr[-(i+1)][:8])
    
    
ratio_result=pd.concat(final_ratio[:6500])
ratio_result.to_csv("ratio.csv",encoding="CP949")

result=pd.concat(final_result_data[:6500])
result.to_csv("KRX_adj_except_div.csv",encoding="CP949")    

{% endhighlight %}
위의 코드를 3시간 정도 돌리면 결과를 볼수 있다. 이때 ratio를 따로 저장하는 이유는 다음 포스트는 이 ratio를 이용하여 배당을 포함한 수정주가를 만들어 볼것이기 때문이다.

만들어진 데이터의 컬럼은 종목코드이고 인덱스는 날짜이다. 컬럼아래 내용이 중간부터 있거나 중간에 사라지거나 시작과 끝이 없고 중간만 있는 데이터가 있다. 이는 상장폐지 종목도 포함 되었기 때문이다. 약 4000개의 상장폐지된 종목을 포함한 회사 데이터가 6500 거래일의 수정주가가 완성되었다.
<center><img src="/images/make_data/adj_price.JPG" ></center>

2001년부터 배당락일 강제 배당락은 폐지되어 자율 배당락이 되었다. 이는 배당락일에 전일 종가와 기준가가 같다는 의미로 배당이 수정주가에 포함되지 않았다는 말이다.
이는 2001년부터는 정확한 수익률 계산을 원래의 수정주가로 할수 없다는 이야기로 다음 포스트는 배당금이 포함되었을때 수정주가 수익률을 구해볼 것이다.

