---
layout: post
title:  "1.이동평균을 이용한 항공권 수요 모멘텀 분석과 시각화"
date:   2020-01-21
categories: Analytics
---
&nbsp;&nbsp; 온라인 여행 검색/예약 서비스의 마케팅팀에서 가장 많이 의뢰하는 문제 중 하나는 시의적절한 마케팅 캠페인 기간에 관한 것이다. 이를테면 8월 휴가철에 여행을 떠나려는 수요가 언제부터 본격적으로 올라오는지를 파악하여 해당 시기에 캠페인을 집중하고자 하는 것이다. 가장 단순한 방법으로는 전년도 수요(웹사이트 방문, 검색 등)의 WoW와 MoM 트렌드를 보고 가장 급격하게 올라오는 시점을 포착하는 것을 고려해 볼 수 있다. 그러나 이 방법의 경우 어느 정도의 상승폭을 모멘텀의 시작으로 인식할 것인지 그 기준점을 정하기 쉽지 않은 문제가 있다. 실제로 마케터들에게 WoW 트렌드를 시각화한 자료를 보여주었을 때 마케터들은 저마다 다른 기준의 상승폭을 급격하다고 표현하는 것을 발견할 수 있었다.
 
&nbsp;&nbsp;이러한 한계점을 극복하기 위해 주식 투자에서 기초적인 기술적 분석 방법으로 쓰이는 [이동 평균][Wikipedia - Moving Average]을 적용해보게 되었다. 마치 트레이더가 이동 평균을 통해 주가의 모멘텀을 이해하는 것과 같이 마케터가 항공권 수요의 모멘텀을 이해할 수 있게 돕는 것이다. 그럼 먼저 태블로(Tableau)를 통해 이동 평균 중 가장 기본이 되는 SMA(Simple Moving Average, 단순 이동 평균)를 직접 시각화해보도록 하자. 여기서 수요를 인식하기 위한 지표로는 편의상 구글 애널리틱스 기준의 'Sessions'을 사용했다.  

```sql
#SMA_7day 
  WINDOW_AVG(SUM([Sessions]), -6, 0)

#SMA_28day
  WINDOW_AVG(SUM([Sessions]), -27, 0)
  
#SMA_Crossover
  [SMA_7day] >= [SMA_28day]
```

&nbsp;&nbsp;태블로의 **WINDOW_AVG()** 함수를 이용하면 쉽게 SMA를 구할 수 있다. 여기서 필자는 SMA의 이동 구간을 각각 7일과 28일로 정의했는데 그것은 온라인 여행 서비스의 트래픽은 일주일 단위로 패턴화 할 수 있기 때문이다(여타 이커머스 서비스와 같이 월/화에 최고점을 찍고 토요일에 최저점을 찍는 패턴을 반복). 주식 시장에서는 20/50/100일 단위를 주로 쓰는데 아무래도 장이 일주일에 5일 동안만 열리기 때문인 것으로 보인다. 또한  **SMA_Crossover**라는 Calculated Field를 만들어 놓았는데, 이는 상대적으로 짧은 이동 구간의 **SMA_7Day**가 **SMA_28Day**를 추월하는 시점을 더 직관적으로 시각화하기 위함이다. 주식 트레이딩에서는 짧은 이동 구간의 SMA가 긴 이동 구간의 SMA를 추월하는 순간을 'Bullish Crossover', 그 반대를 'Bearish Crossover'라고 부른다고 한다. 여기서는 그 'Bullish Crossover'를 항공권 수요의 모멘텀이 시작되는 시점으로 이해하고 그 시점을 마케터가 파악하기 쉽게 시각화 하고자 한다.  

<img src="/assets/image/sma_crossover.PNG" width="100%" height="100%">&nbsp;&nbsp;  
**<Figure 1. SMA Crossover>**


&nbsp;&nbsp;빨간 선이 SMA_7Day, 파란 선은 SMA_28Day이고, 미리 만들어놓았던 SMA_Crossover 필드를 이용하여 7일 이동 평균이 상대적으로 더 낮은 기간은 희미한 컬러로 처리했다. 마케터들은 특정 항공권 수요에 대하여 뚜렷한 빨간 선이 시작하는 지점에 캠페인을 런칭하는 것을 고려해 볼 수 있을 것이다.

&nbsp;&nbsp;그런데 SMA는 일반적으로 그 후행(lagging)성이 한계점으로 지적된다. 그래서 최근 데이터에 더 큰 가중치를 부여하여 이동 평균을 구하는 방법이 같이 쓰이는데 바로 EMA(Exponential Moving Average, 지수 이동 평균)이라는 것이다. EMA를 쓰게 되면 SMA에 비해 상대적으로 덜 lagging하면서 더 빠르게(민감하게) 반응하는 이동 평균선을 그려볼 수 있다. 항공권 수요에 EMA를 적용하는 것은 얼마나 의미가 있을까? 태블로에서 직접 구현하여 알아보도록 하자. EMA를 구하는 공식은 위키피디아의 [지수 이동 평균][Wikipedia - Exponential Moving Average]과 [Data Distilled의 포스팅][Data Distilled Posting] 을 참고했다.


```sql
#EMA_7day 
  (2/7 #alpha coefficient)*sum([Sessions]) + (1-(2/7))*PREVIOUS_VALUE(sum([Sessions]))

#EMA_28day
  (2/28 #alpha coefficient)*sum([Sessions]) + (1-(2/28))*PREVIOUS_VALUE(sum([Sessions]))
  
#EMA_Crossover
  [EMA_7day] >= [EMA_28day]
```

<img src="/assets/image/ema_crossover.PNG" width="100%" height="100%">&nbsp;&nbsp; 
**<Figure 2. EMA Crossover>**


&nbsp;&nbsp;상대적으로 덜 Lagging하지만, Smoothing한 것이 의미가 없을 정도로 훨씬 더 민감하게 움직이는 EMA 선을 볼 수 있다. 아무래도 온라인 여행 서비스 트래픽 데이터의 요일별 패턴이 뚜렷하다보니 최신 정보에 더 많은 가중치를 주는 EMA가 의미있는 시사점을 제공해주지 못하는 것이 아닐까 한다(뇌피셜). 오히려 요일별로 가중치를 주는 이동 평균을 공식으로 만들어 적용하는게 더 적절할 것 같고, 트래픽 보다는 항공권 가격 추세를 살펴볼 때 EMA가 더 의미가 있을 것 같다는 생각도 든다. 


**NEXT**&nbsp;&nbsp; 

(1)EMA Crossover로 구간별 항공권 가격 추세 살펴보기

(2)EMA를 바탕으로 한 MACD(Moving Average Convergence Divergence)방법도 적용해보기

(3)파이썬 [Pandas][Pandas Built In Function]에는 이미 관련 함수들이 내장되어 있는데, 이를 활용하여 태블로로 시각화까지 해보기 


[Wikipedia - Moving Average]: https://en.wikipedia.org/wiki/Moving_average
[Wikipedia - Exponential Moving Average]:   https://en.wikipedia.org/wiki/Moving_average#Exponential_moving_average
[Data Distilled Posting]: https://medium.com/data-distilled/lets-get-recursive-previous-value-vs-lookup-in-tableau-c67f70392725
[Pandas Built In Function]: https://pandas.pydata.org/pandas-docs/stable/user_guide/computation.html#exponentially-weighted-windows
