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


<center><img src="/images/dividend_stra/div_stra_ex.png" ></center>


이러한 투자전략은 세가지로 나눌수 있는다 아래 사진과 같다.
<center><img src="/images/dividend_stra/strategy.png" ></center>

위의 세가지 전략을 한국시장 20년 데이터로 시뮬레이션 해보고 통계를 내보자.

## KRX에서 일별종가 데이터와 배당데이터 긁어오기

먼저 어떤 데이터를 이용하게 될지 모르니 KRX market data에서 20년치 일별 종가를 긁어오자.
[KRX 시가총액 상위 링크](https://marketdata.krx.co.kr/mdi#document=040402)
<center><img src="/images/dividend_stra/market_data1.JPG" ></center>
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

<center><img src="/images/dividend_stra/market_data2.JPG" ></center>


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
이때 사용한 방법은 일별 종가를 저장할때 장이 열린날만 저장하도록 했는데 그 데이터가 저장된 폴더의 리스트를 가져와 장이 열린날을 저장하였다.
배당락은 그 해의 영업일 종료 2영업일 전이기 때문데 위의 코드와 같이 저장하면된다. 
또한 배당금 입금일은 4월 10일 전후로 입금이 되는데 각 주식의 정확한 입금일은 다 다르므로, 배당락 이후 70거래일로 일괄 설정하였다.

## 사용할 데이터 만들기

우리가 사용할 데이터는 배당락 전날, 배당락일, 배당금입금일, 배당금 이 네가지 데이터를 사용해야한다. 이는 아래 코드와 같이 가져올수 있다.

{% highlight python %}
#######배당락 전일 주가
price_data=pd.read_csv("D:\코스피종가\\"+ex_dividend_date_eve[i],encoding="CP949") ####배당락 전일
price_data=price_data[["현재가","종목코드","종목명"]]
price_data.columns=["배당락전일종가","종목코드","종목명"]
price_data=price_data.set_index("종목코드")
########## 배당금 입금일 주가
price_data2=pd.read_csv("D:\코스피종가\\"+dividend_deposit_day[i],encoding="CP949") ####배당락 전일
price_data2=price_data2[["현재가","종목코드","종목명"]]
price_data2.columns=["배당락입금일종가","종목코드","종목명"]
price_data2=price_data2.set_index("종목코드")
########## 배당락일 시가 종가
price_data3=pd.read_csv("D:\코스피종가\\"+ex_dividend_date[i],encoding="CP949") ####배당락 전일
price_data3=price_data3[["현재가","시가","종목코드","종목명"]]
price_data3.columns=["배당락일_종가","배당락일_시가","종목코드","종목명"]
price_data3=price_data3.set_index("종목코드")
####### 배당금 데이터
div_data=pd.read_csv("C:\\Users\LEE\Desktop\KRX\\"+ex_dividend_date_eve[i][:4]+"배당.csv",encoding="CP949")
div_data=div_data[["종목코드","종목명","주당배당금"]]
div_data=div_data.set_index("종목코드")
price_data=price_data.join(div_data[["주당배당금"]],how="outer")
price_data=price_data.join(price_data2[["배당락입금일종가"]],how="outer")
price_data=price_data.join(price_data3[["배당락일_종가","배당락일_시가"]],how="outer")
price_data=price_data.loc[price_data["배당락전일종가"].dropna().index].fillna(0) ## 중간에 상장폐지 됐다면 0
{% endhighlight %}
이때 인덱스 i는 배당락 일을 계산할때 연도별 배당락일의 인덱스이다. 즉 i=0 일때 아래 사진과 같이 데이터가 저장된다.
<center><img src="/images/dividend_stra/div_data1.JPG" ></center>
이때 주의해야할점은 배당금 입금일 종가 데이터가 없을때는 몇가지 경우가 있다.
1.상장폐지를 당했다.
2.자진상장폐지를 하였다.
3.인수합병되었다.(우선주가 본주에 합병되는것도 포함)
4.청산되었다.
등의 경우로 나눌 수 있는데 보수적으로 시뮬레이션 하기위해 이는 전부 100%손해로 시뮬레이션 하였다.

또한 주식분할, 주식병합, 무상증자, 주식배당, 액면분할등으로 인한 주가 변화는 그 기간동안 매우 적은 샘플이기 때문에 정상주가라고 가정하고 시뮬레이션 하였다.(종목 2000개 이상)

## 데이터를 이용하여 수익률 계산

위의 데이터를 반복문을 이용하여 연도별로 수익률을 구해보는 코드는 아래와 같다. 이때 벤치마크는 동기간 코스피 수익률이다. 거래세는 0.3%로 가정하였고 배당소득세는 15.4%로 가정 하였다.

코스피, 코스닥 전종목을 Equal weighted로 보유 했다고 가정한 수익률이다. 예를들어 삼성전자, LG전자, 하이닉스가 코스피 코스닥 전종목이라면 각각 100만원씩 총 300만원의 포트폴리오를 가지고있다는 전략이다.
{% highlight python %}
div_return_data_raw=[0]*20
div_return_result=pd.DataFrame(index=range(2000,2019,1),columns=["전략1","전략2","전략3","전략1_벤치마크","전략2_벤치마크","전략3_벤치마크"])
market_data=pd.read_csv("market.csv",encoding="CP949").set_index("날짜")
transaction_tax=0.003
dividend_income_tax=0.154
for i in range(5,25,1):
    #######배당락 전일 주가
    price_data=pd.read_csv("D:\코스피종가\\"+ex_dividend_date_eve[i],encoding="CP949") ####배당락 전일
    price_data=price_data[["현재가","종목코드","종목명"]]
    price_data.columns=["배당락전일종가","종목코드","종목명"]
    price_data=price_data.set_index("종목코드")
    ########## 배당금 입금일 주가
    price_data2=pd.read_csv("D:\코스피종가\\"+dividend_deposit_day[i],encoding="CP949") ####배당락 전일
    price_data2=price_data2[["현재가","종목코드","종목명"]]
    price_data2.columns=["배당락입금일종가","종목코드","종목명"]
    price_data2=price_data2.set_index("종목코드")
    ########## 배당락일 시가 종가
    price_data3=pd.read_csv("D:\코스피종가\\"+ex_dividend_date[i],encoding="CP949") ####배당락 전일
    price_data3=price_data3[["현재가","시가","종목코드","종목명"]]
    price_data3.columns=["배당락일_종가","배당락일_시가","종목코드","종목명"]
    price_data3=price_data3.set_index("종목코드")
    ####### 배당금 데이터
    div_data=pd.read_csv("C:\\Users\LEE\Desktop\KRX\\"+ex_dividend_date_eve[i][:4]+"배당.csv",encoding="CP949")
    div_data=div_data[["종목코드","종목명","주당배당금"]]
    div_data=div_data.set_index("종목코드")
    price_data=price_data.join(div_data[["주당배당금"]],how="outer")
    price_data=price_data.join(price_data2[["배당락입금일종가"]],how="outer")
    price_data=price_data.join(price_data3[["배당락일_종가","배당락일_시가"]],how="outer")
    ##### 배당락전일 종가 매수 배당락일 시가매도
    price_data["배당락일_시가매도_수익금"]=price_data["배당락일_시가"]*(1-transaction_tax)-price_data["배당락전일종가"]+price_data["주당배당금"]*(1-dividend_income_tax)
    price_data["배당락일_시가매도_수익률"]=price_data["배당락일_시가매도_수익금"].dropna()/price_data["배당락전일종가"]
    ### 아웃라이어 제거
    price_data["배당락일_시가매도_수익률"]=price_data[abs(price_data["배당락일_시가매도_수익률"])<1.01]["배당락일_시가매도_수익률"]
    price_data=price_data.loc[price_data["배당락전일종가"].dropna().index].fillna(0) ## 중간에 상장폐지 됐다면 0
    #####################################################
    ##### 배당락전일 종가 매수 배당락일 종가매도
    price_data["배당락일_종가매도_수익금"]=price_data["배당락일_종가"]*(1-transaction_tax)-price_data["배당락전일종가"]+price_data["주당배당금"]*(1-dividend_income_tax)
    price_data["배당락일_종가매도_수익률"]=price_data["배당락일_종가매도_수익금"].dropna()/price_data["배당락전일종가"]
    ### 아웃라이어 제거
    price_data["배당락일_종가매도_수익률"]=price_data[abs(price_data["배당락일_종가매도_수익률"])<1.01]["배당락일_종가매도_수익률"]
    #####################################################
    ##### 배당락전일 종가 매수 입금일 종가매도
    price_data["배당입금일_종가매도_수익금"]=price_data["배당락입금일종가"]*(1-transaction_tax)-price_data["배당락전일종가"]+price_data["주당배당금"]*(1-dividend_income_tax)
    price_data["배당입금일_종가매도_수익률"]=price_data["배당입금일_종가매도_수익금"].dropna()/price_data["배당락전일종가"]
    ### 아웃라이어 제거
    price_data["배당입금일_종가매도_수익률"]=price_data[abs(price_data["배당입금일_종가매도_수익률"])<3]["배당입금일_종가매도_수익률"]
    #####################################################
    div_return_data_raw[i-5]=price_data
    div_return_result.loc[2000+i-5,"전략1"]=price_data["배당락일_시가매도_수익률"].mean()
    div_return_result.loc[2000+i-5,"전략1_벤치마크"]=market_data.loc[int(ex_dividend_date[i][:8])]["시가"]/market_data.loc[int(ex_dividend_date_eve[i][:8])]["종가"]-1
    div_return_result.loc[2000+i-5,"전략2"]=price_data["배당락일_종가매도_수익률"].mean()
    div_return_result.loc[2000+i-5,"전략2_벤치마크"]=market_data.loc[int(ex_dividend_date[i][:8])]["종가"]/market_data.loc[int(ex_dividend_date_eve[i][:8])]["종가"]-1    
    div_return_result.loc[2000+i-5,"전략3"]=price_data["배당입금일_종가매도_수익률"].mean()
    div_return_result.loc[2000+i-5,"전략3_벤치마크"]=market_data.loc[int(dividend_deposit_day[i][:8])]["종가"]/market_data.loc[int(ex_dividend_date_eve[i][:8])]["종가"]-1
div_return_result["전략1_알파"]=div_return_result["전략1"]-div_return_result["전략1_벤치마크"]
div_return_result["전략2_알파"]=div_return_result["전략2"]-div_return_result["전략2_벤치마크"]
div_return_result["전략3_알파"]=div_return_result["전략3"]-div_return_result["전략3_벤치마크"]

{% endhighlight %}

위 코드의 실행 결과인 div_return_result의 결과는 아래 사진과 같다.
<center><img src="/images/dividend_stra/div_data2.JPG" ></center>
위의 코드를 좀더 보기 쉽게 아래코드를 이용하여 평균과 표준편차를 구하면
{% highlight python %}
cols=div_return_result.columns
for i in range(div_return_result.shape[1]):
    div_return_result.loc["평균수익률",cols[i]]=div_return_result[cols[i]].mean()
    div_return_result.loc["표준편차",cols[i]]=div_return_result[cols[i]].std()
{% endhighlight %}
아래 사진과 같다.
<center><img src="/images/dividend_stra/div_data3.JPG" ></center>

표를 분석해보면 전략3은 전종목을 배당락 전날 종가에 매수하여 배당금 입금일 일괄 매도 한다는 전략인데 약 5%의 초과 수익을 내었다. 하지만 이는 홀드하고 있으면 돈을 번다는 의미보단 시가총액이 작은 종목들이 더 올랐다는 의미로 받아 들어야한다. 즉 파마프렌치의 팩터중 SMB(Small Minus Big)의 영향인듯 보인다. 작은 기업의 수익률이 큰 기업의 수익률보다 큰 걸 뜻한다.

하지만 전략2의 수익률은 약 1% 정도로 약 3개월간의 입금 기간을 고려하면 연환산 수익률로 4%이다. 이는 꽤 좋은 수익률로 괜찮은 투자법으로 보인다.


아래 사진은 매년 전략별 수익률을 히스토그램으로 표현한 사진이다.
<center><img src="/images/dividend_stra/image.jpg" ></center>

위의 사진을 만드는 코드는 아래와 같다.
{% highlight python %}
plt.figure(figsize=(20,150))
for i in range(20):
    plt.subplot(20,3,3*i+1)
    plt.hist(div_return_data_raw[i]["배당락일_시가매도_수익률"],bins=50)
    plt.xlabel("Return",fontsize=15)
    plt.ylabel("N",fontsize=15)
    plt.title(ex_dividend_date_eve[i+5][:4]+"_Strategy1",fontsize=20)

    plt.subplot(20,3,3*i+2)
    plt.hist(div_return_data_raw[i]["배당락일_종가매도_수익률"],bins=50)
    plt.xlabel("Return",fontsize=15)
    plt.ylabel("N",fontsize=15)
    plt.title(ex_dividend_date_eve[i+5][:4]+"_Strategy2",fontsize=20)
    
    plt.subplot(20,3,3*i+3)
    plt.hist(div_return_data_raw[i]["배당입금일_종가매도_수익률"],bins=50)
    plt.xlabel("Return",fontsize=15)
    plt.ylabel("N",fontsize=15)
    plt.title(ex_dividend_date_eve[i+5][:4]+"_Strategy3",fontsize=20)

plt.tight_layout()
plt.savefig('image.jpg')
plt.show()
{% endhighlight %}

위의 사진을 보면 2019년에 비해 2000년의 표준편차가 더 큰걸 볼수 있다. 즉 시장이 점점 효율적으로 변하면서 예상 배당금 만큼 투자자들이 행동한다는 것을 알 수 있다.


## 종목별 전략 수익률


아래의 코드는 SK텔레콤의 수익률을 2000부터 2019년까지 나타 낼 수 있는 코드이다.
{% highlight python %}
stock_name="SK텔레콤"
stock_return_result=pd.DataFrame(index=range(2000,2019,1),columns=["전략1","전략2","전략3"])
stock_code=div_return_data_raw[19][div_return_data_raw[19]["종목명"]==stock_name].index[0]
print(stock_code)
for i in range(19,-1,-1):
    try:
        stock_return_result.loc[int(ex_dividend_date_eve[i+5][:4]),"전략1"]=div_return_data_raw[i].loc[stock_code]["배당락일_시가매도_수익률"]
        stock_return_result.loc[int(ex_dividend_date_eve[i+5][:4]),"전략2"]=div_return_data_raw[i].loc[stock_code]["배당락일_종가매도_수익률"]
        stock_return_result.loc[int(ex_dividend_date_eve[i+5][:4]),"전략3"]=div_return_data_raw[i].loc[stock_code]["배당입금일_종가매도_수익률"]
    except KeyError:
        pass
stock_return_result.loc["수익률평균","전략1"]=stock_return_result["전략1"].mean()
stock_return_result.loc["수익률평균","전략2"]=stock_return_result["전략2"].mean()
stock_return_result.loc["수익률평균","전략3"]=stock_return_result["전략3"].mean()
stock_return_result
{% endhighlight %}
위 코드의 결과는 아래와 같다. 
<center><img src="/images/dividend_stra/div_data4.JPG" ></center>
SK텔레콤은 우리나라 시장의 대표적인 배당주로 배당투자의 정석이라고 볼수 있다. SK텔레콤은 전략1이 전략2보다 더 나은 성과를 보이는데, 이는 단기 투자자들의 매물이 다음날에 몰리는 경향때문일것이라고 예상한다.

위의 코드를 이용하여 2020년 12월 24일 기준 시가총액 상위 50개 기업의 과거 20년 전략별 수익률을 코드로 나타내면 아래와 같다.

{% highlight python %}
recent_marketcap=pd.read_csv("D:\코스피종가\\"+arr[-1],encoding="CP949")
N=50
stock_list=recent_marketcap["종목코드"].iloc[:N]
stock_name_list=recent_marketcap["종목명"].iloc[:N]
stock_result=pd.DataFrame(columns=["전략1","전략2","전략3"])
for code in stock_list:
    stock_return_result=pd.DataFrame(index=range(2000,2019,1),columns=["전략1","전략2","전략3"])
    for i in range(19,-1,-1):
        try:
            stock_return_result.loc[int(ex_dividend_date_eve[i+5][:4]),"전략1"]=div_return_data_raw[i].loc[code]["배당락일_시가매도_수익률"]
            stock_return_result.loc[int(ex_dividend_date_eve[i+5][:4]),"전략2"]=div_return_data_raw[i].loc[code]["배당락일_종가매도_수익률"]
            stock_return_result.loc[int(ex_dividend_date_eve[i+5][:4]),"전략3"]=div_return_data_raw[i].loc[code]["배당입금일_종가매도_수익률"]
        except KeyError:
            pass
    stock_return_result.loc["수익률평균","전략1"]=stock_return_result["전략1"].mean()
    stock_return_result.loc["수익률평균","전략2"]=stock_return_result["전략2"].mean()
    stock_return_result.loc["수익률평균","전략3"]=stock_return_result["전략3"].mean()
    stock_result.loc[code]=stock_return_result.loc["수익률평균"]
{% endhighlight %}

위 코드의 결과는 아래 사진과 같다. 
<center><img src="/images/dividend_stra/div_result1.png" ></center>

전략1의 평균은 0.2%, 전략2의 평균은 0.6%, 전략3의 평균은 5%가 나왔다. 하지만 전략3은 survivor bias(주가가 올랐기때문에 시가총액이 상위 50위임)로 인하여 정확한 결과 해석이 어렵고, 전략2의 평균인 0.6%는 연환산으로 2.4%이익이므로 과거 20년 평균 이자율 수준이다. 하지만 이러한 단순한 계산은 의미가 없다. 
<center><img src="/images/dividend_stra/interest_rate.jpg" ></center>

<center><img src="/images/dividend_stra/div_data5.JPG" ></center>
하지만 위의 사진은 배당금 입금액을 제외한 전략1과 전략2를 "배당금제외 전략1"과 "배당금제외 전략2"로 계산한 값이다. "배당금제외 전략1"은 배당락으로 인한 약 5%의 손실이 났지만 "배당금제외 전략2"는 거래세 정도의 손해만 났다.

즉 하루의 투자(예수금 기간 제외)로 배당금이 입금될때까지의 3개월동안 유동성은 그대로 유지하고 배당금의 이익만 가져가는것이다.

결론은 배당락일 종가에 사서 다음날 종가에 팔면 유동성은 거의 유지하면서 연환산 수익률 약 3%의 이익을 볼수 있다.
## 한계점

매우 적은 비율이긴 하지만 액면분할 등 가격등락 이외의 가격변화 요인들을 정확히 반영하지 않음
시가총액식인 코스피 지수와 균등분할 포트폴리오인 실험 데이터의 단순한 알파 비교는 정확하지 않음

## 추후 생각해볼것

1. 고배당이 예상되는 종목들 또는 고배당을 줘왔던 회사들의 배당락날 주가의 흐름과 그 외의 종목의 주가 흐름분석
2. 투자주체중 금융투자들이 배당락 10일 전부터 대규모 순매수 이유
3. 기관투자가들의 세금관련 이슈중 배당락이 미치는 영향 (배당락전날 매수 후 다음날 매도 했을때 법인세 감면이슈?)
