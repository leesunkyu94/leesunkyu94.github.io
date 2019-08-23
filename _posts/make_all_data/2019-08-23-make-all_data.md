---
layout: single
classes: wide
title: "파이썬을 이용하여 코스피,코스닥 전종목의 일별 종가를 긁어오자"
categories:
    - Data 만들기
---
##현재 시점 코스피,코스닥의 전종목 코드를 가져오자
현재 시점의 코스피와 코스닥의 전종목 코드를 가져오려면 http://marketdata.krx.co.kr/mdi#document=040602 에서 원하는 날짜의 데이터를 CSV로 다운받아 첫 두열을 복사하여 아래 사진과 같이 TXT파일에 저장하면 된다. 저장할때는 종목코드를 6자리로 맞춰 저장해야한다.



<center><img src="/images/make-all-data/1.PNG" ></center>

그 다음 위에서 저장한 종목코드들을 읽어와 아래 링크를 참조하여 ISIN코드로 바꾸어준다.

https://leesunkyu94.github.io/data%20%EB%A7%8C%EB%93%A4%EA%B8%B0/make-isin/

아래의 코드는 위의 링크를 함수화 한것이다.

{% highlight python %}

def code_to_ISIN():
    data=open("code.txt","r")#저장한 텍스트 파일위치는 유동적으로 설정
    code_list=[]
    temp=data.readlines()
    for i in range(0,len(temp)):#종목명과 종목코드를 빼고싶으면 0말고 1부터 돌리면 된다.
        code_list.append(temp[i].split())
    
    for j in range(0,len(code_list)):
        result=0
        for i in range(0,len(code_list[j][0])):
            temp=int(code_list[j][0][i])*(1+(i%2))
            if temp>=10:
                result+=int(temp/10)
                result+=int(temp%10)
            else:
                result+=temp
        result+=20

        isin_code="KR7"+code_list[j][0]+"00"+str((10-result%10)%10)
        code_list[j][0]=isin_code
    return code_list
code_to_ISIN()

{% endhighlight %}

code_to_ISIN을 호출하면 종목코드가 ISIN으로 바뀌어서 return된다.


## ISIN코드를 이용하여 데이터를 받아오자
일별 데이터를 받아오는 방법은 전 포스팅을 첨부하겠다.(https://leesunkyu94.github.io/data%20%EB%A7%8C%EB%93%A4%EA%B8%B0/make_data/) 전포스팅에서 만든 코드를 함수화 해서 보면 아래와 같다.

{% highlight python %}
def get_daily_price(code):

    gen_otp_url = "http://marketdata.krx.co.kr/contents/COM/GenerateOTP.jspx"+"?url=MKD/04/0402/04020100/mkd04020100t3_02"
    gen_otp_data = {
        'name':'fileDown',
        'filetype':'csv',
        'isu_cd': code,
        'adj_stkprc': 'Y',
        'fromdate': '19800101',
        'todate': '20190823',
    }
    
    r = requests.get(gen_otp_url, params=gen_otp_data)
    
    code = r.text # 리턴받은 값을 아래 요청의 입력으로 사용.


    down_url = 'http://file.krx.co.kr/download.jspx'
    down_data = {
        'code': code,
    }

    r = requests.post(down_url, data=down_data, headers={
        'Referer': 'Referer: http://marketdata.krx.co.kr/mdi'
    })

    r.encoding = "utf-8-sig"
    df = pd.read_csv(BytesIO(r.content), header=0, thousands=',')

    return df
    
{% endhighlight %}

위의 두함수를 이용하여 종가만 모으는 코드는 아래와 같다.
{% highlight python %}
code_list=code_to_ISIN()
data = get_daily_price(code_list[0][0])
data.rename(columns={'종가': code_list[0][1]}, inplace=True) #종가를 종목명으로 바꿔준다.

for j in range(1,len(code_list)): #앞에서 만든 data를 다른 종목코드를 이용해 받아온 종가와 합쳐준다.
    df2 = get_daily_price(code_list[j][0])
    df2.rename(columns={'종가': code_list[j][1]}, inplace=True)
    data=data.join(df2[[code_list[j][1]]])
    print(j)
data=data.drop(columns=['대비','거래량(주)',"거래대금(원)","시가","고가","저가","시가총액(백만)","상장주식수(주)"])
data.to_csv('코스피코스닥전종목.csv')

{% endhighlight %}


코스피코스닥전종목.csv에 저장이 완료 되었다.
