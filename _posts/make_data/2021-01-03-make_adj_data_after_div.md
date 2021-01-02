---
layout: single
classes: wide
title: "파이썬을 이용하여 배당금이 고려된 수정주가를 산출하자(결과 파일 포함)"
categories:
    - Data 만들기
---

## 수정주가란?
수정 주가란 유 · 무상증자，액면분할 등에 따른 주식 수 변화를 반영해 비교할 수 있는 주가다. 예를들어 삼성전자는 액면분할을하여 주식수가 50배 늘어난 대신 주가는 1/50으로 조정해주어 현재 주가기준으로 과거에 얼마였는지 알수 있는 지표이다.

장이 개장하기전 한국거래소는 주식 수 변화가 있는 종목의 기준가를 산정하여 당일 거래가 시작된다. 즉 이벤트가 없는 종목은 전일종가/당일기초가 비율은 1일것이고 삼성전자의 경우 액면분할 이벤트가 있는 날엔 전일종가/당일기초가는 50이 될것이다. 가격이 1/50이 되었기 때문이다. 

하지만 2001년 이전인 2000년까지는 강제 배당락으로 인하여 수정주가에 배당금이 반영 되었지만, 이후에는 자율 배당락으로 바뀌어 수정주가에 배당금이 포함되지 않는다.

수정주가는 과거에 투자한 자금 대비 현재 주가가 어느정도 수준에 와 있는지 알아보는 것이기 때문에 배당을 고려하는것이 매우 중요하다. 

[이전포스팅 수정주가를 산출하는법](https://leesunkyu94.github.io/data%20%EB%A7%8C%EB%93%A4%EA%B8%B0/make_adj_data/)



#수정주가에서 배당을 고려하는법 

간단하게 생각해서 현재의 주가가 10000원이고 과거에 배당을 500원을 주었다. 그리고 그 과거 시점에도 주가가 10000원이라면 수정주가는 어떻게 계산해야할까?

10000원에서 500원을 빼주면 되는것이다. 

또 중간에 액면분할등 주식수가 변하는 이벤트가 있었던 주식을 예로 들어보자.

현재의 주가가 10000원이고 액면분할 이전에 10000원을 배당으로 주었다. 그때의 주가는 백만원이였고, 분할비율은 100:1이였다. 수정주가는 어떻게 계산해야할까?

100만원/100-10000원/100을 해주면 9900원이 수정주가가 되는것이다.

또한 배당락일 기준전 모든 날짜의 주가에 수정배당금을 빼주어야한다. 그래야 현재 시점에서 과거 시점의 주가를 정확히 볼 수 있기 때문이다.

이를 위해 전 포스팅에서 일자별 ratio를 저장해 놓은것이다.



## 배당포함 수정주가 계산


위의 로직을 코드로 표현하면 아래와 같다.
{% highlight python %}
import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

arr = os.listdir("D:\코스피종가")
ex_dividend_date=[]       ####배당락일
for i in range(len(arr)-100):
    if arr[i][:4]!=arr[i+2][:4] and arr[i][:4]==arr[i+1][:4]:
        ex_dividend_date.append(arr[i])
adj_price=pd.read_csv("KRX_adj_except_div.csv",encoding="CP949",index_col="Unnamed: 0")
ratio=pd.read_csv("ratio.csv",encoding="CP949",index_col="Unnamed: 0")

for i in range(20):
    #연도별 배당금 데이터
    div_data=pd.read_csv("C:\\Users\LEE\Desktop\블로그\배당금데이터\\"+ex_dividend_date[-(i+1)][:4]+"배당.csv",encoding="CP949",index_col="종목코드")
    #배당금을 ratio와 곱해주어 현재 주가 기준으로 배당을 얼마 받았는지 본다
    temp_df=div_data["주당배당금"].to_frame().join(ratio.loc[int(ex_dividend_date[-(i+1)][:8])],how="outer")
    temp_df["수정배당금"]=(temp_df["주당배당금"]*temp_df[int(ex_dividend_date[-(i+1)][:8])]).fillna(0)
    #배당락 이전 주가에서 모두 배당금을 빼준다
    adj_price.loc[int(ex_dividend_date[-(i+1)][:8]):]=adj_price.loc[int(ex_dividend_date[-(i+1)][:8]):]-temp_df["수정배당금"]

{% endhighlight %}

코드는 매우 간단하다. ratio를 이전 포스팅에서 계산해 놓은 덕분이다.

## 배당금을 포함한 삼성전자와 SK텔레콤의 주가

우리나라 대표주식 삼성전자와 대표 배당주식 SK텔레콤의 배당금을 포함한 수정주가는 어떨까?
먼저 배당금을 반영 안한 주가의 추이를 보자
<center><img src="/images/make_data/samsung1.JPG" ></center>

<center><img src="/images/make_data/skt1.JPG" ></center>
삼성전자는 1995년 5월8일 기준 1563원을 투자하면 2020년12월24일 주가인 73900원이 된다고 볼수 있다
SK텔레콤은 1995년 5월8일 기준 39758원을 투자하면 2020년12월24일 주가인 245000원이 된다고 볼수 있다.

하지만 배당금을 포함하면 어떨까?
<center><img src="/images/make_data/samsung.JPG" ></center>

<center><img src="/images/make_data/skt.JPG" ></center>

삼성전자는 1995년 5월8일 기준 -4601원을 투자하면 2020년12월24일 주가인 73900원이 된다고 볼수 있다
SK텔레콤은 1995년 5월8일 기준 -121271원을 투자하면 2020년12월24일 주가인 245000원이 된다고 볼수 있다.

즉 오히려 회사에서 돈과 주식을 함께 주는 것으로 생각 할수 있다.


이처럼 배당을 포함하고 안하고는 장기 투자 성과에 큰 영향을 미칠수 있다.


한계점

분기배당을 한꺼번에 연말배당으로 계산하여 약간의 오류가 있음!


위의 데이터를 받고 싶은 사람은 아래 링크 클릭

[1995년5월8일~2020년12월24일 코스피 코스닥 전종목 수정주가 배당미포함](https://drive.google.com/file/d/1aIdjnUl1aRW7xbAhYtf_756Zy5LUO4Yq/view?usp=sharing)

[1995년5월8일~2020년12월24일 코스피 코스닥 전종목 수정주가 배당포함](https://drive.google.com/file/d/1EGsdhcyYXXlU7w74TyXICzRn02YkbKC-/view?usp=sharing)



시작점
