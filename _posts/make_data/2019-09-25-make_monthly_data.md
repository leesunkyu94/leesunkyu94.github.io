---
layout: single
classes: wide
title: "파이썬으로 KOSPI200 일별종가를 월별 수익률로 전환하자"
categories:
    - Data 만들기
---

## 변동성측정을 위한 첫번째과정
Minimum Variance Portfolio나 risk parity Portfolio 등을 만들기 위한 첫번째과정은 변동성과 수익률 추정이다. 변동성이나 평균 수익률은 보통 월별 수익률을 이용하여 측정한다. 이를 위해 저번 포스팅에서 긁어왔던 데이터를 조금 가공하면 현재의 KOSPI200 종가를 쭉 가져올수 있다(과거 K200 구성종목은 다르다) 일별 데이터는 아래 링크에 올려두겠다. 수정주가를 적용한것으로 액면분할 배당 무상증자등이 모두 반영된 수치이다.
(https://github.com/leesunkyu94/leesunkyu94.github.io/blob/master/_posts/make_data/K200_Adj.csv)



## 데이터를 읽어 Dataframe형태로 바꾼다


{% highlight python %}
import pandas as pd
import numpy as np

data=pd.read_csv("K200_Adj.csv",index_col="date", encoding='CP949',dtype=np.float64)
daily_data=data.iloc[::-1]
{% endhighlight %}

이때 한글이 포함되어있으므르 encoding옵션에 CP949을 넣어준다.

월별 수익률은 (t시점 월말종가/t-1시점 월말종가)-1 이므로 아래와 같이 월말 인덱스를 따로 저장한다.

{% highlight python %}
datelist=daily_data.index
change_point=[]
for i in range(len(datelist)-1):
    if str(datelist[i])[5:6]!=str(datelist[i+1])[5:6]:
        change_point.append(i)
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
