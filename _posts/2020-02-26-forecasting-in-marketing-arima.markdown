---
layout: post
title:  "6.마케팅과 예측(forecasting) - 1.ARIMA"
date:   2020-02-26
categories: Analytics
---
Why Forecasting in marketing?
 (1)CausalImpact
 (2)Tableau Built-in(Exponential Smoothing), Prophet
 (3)ARIMA(Seasonal)
 
ARIMA in Python
 (1)Get the Data(KR Air Departure)
 (2)Decomposition
 (3)Stationarity Test
 (4)Differencing Method
 (5)ACF/PACF Interpretation
 (6)Auto-Arima
 (7)Seasonal Arima
 (8)Model Fitting and Forecasting
 
&nbsp;&nbsp; 마케팅에서 예측(forecasting)은 수요나 매출 예측 등 거시적인 목적으로도 많이 쓰이지만, 개인적으로는 단기적인 마케팅 캠페인의 성과를 측정하기 위한 여러가지 방법을 고민하면서 forecasting 기법에 대해 더 깊은 관심을 갖게 되었다. 예측을 활용한 성과 측정 방식의 논리를 거칠게 말하자면 forecasting 기법을 통해 'synthetic control'이라는 조건법적(counterfactual) 결과를 만들어 실제 결과와 비교하는 것이다(e.g., 이 캠페인을 하지 않았더라면 vs. 실제로 이 캠페인을 했더니).  

<img src="/assets/image/synthetic_control.png" width="100%" height="100%">&nbsp;&nbsp;  
<Figure 1. Synthetic Control _source:_ [_World Bank Blog_][World Bank Blog]>
  
&nbsp;&nbsp; 이와 관련하여 마케터들에게 가장 널리 알려진 방법론은 2015년에 구글에서 발표한 [CausalImpact][CausalImpact]일 것이다. 이 방법론은 'Bayesian structural time-series models'에 기반하여 synthetic control을 만들어 주는데, 원리를 정확히 이해하지 못하더라도 배포되어있는 R패키지를 이용하여 누구나 결과값을 만들어 낼 수 있다. 현재 근무 중인 회사에서도 이 패키지를 Shiny 웹앱으로 배포하여 누구나 사용할 수 있도록 하고 있는데, 이 방법론에 대한 생각은 (모델에 대한 공부를 더 한 이후에..) 추후 별도 포스팅에서 정리하고자 한다. 그 다음으로는 'Forecasting at scale'을 주창하며 페이스북에서 2017년에 발표한 [Prophet][Prophet]이 있을 것이다. Prophet은 시계열 데이터를 'Growth + Seasonality + Holidays + Error'의 additive한 모델로 인식하여 필자와 같이 통계적 지식이 부족한 사용자도 파라미터들을 보다 직관적으로 이해하고 조정하면서 사용할 수 있도록 만들어놨다. 실제로 회사에서도 많은 사람들이 Prophet 패키지를 애용하고 있다. 결과값을 만들기 가장 쉬운 방법으로는 [태블로(Tableau)의 built-in forecasting][Tableau Forecasting]이 있을 것 같다. 이 방법은 'Exponential Smoothing(지수평활법)'에 근거하는데 그 성능과 설명의 용이성을 차치하고 클릭 몇 번으로 결과값을 제조(?)할 수 있는 것이 가장 큰 장점이다. 마지막으로 YoY를 이용하는 가장 원시적인 방법이 있는데 전년 데이터가 없을 경우 사용할 수 없고 '어느 시점의 어떤 YoY'를 이용하는 것이 가장 적절한지 평가하기 어렵다는 치명적인 단점을 갖고 있다. 

&nbsp;&nbsp; 그러나 이와 같이 접근이 상대적으로 용이한 방법론들을 살펴보기 전에, 본 포스팅에서는 먼저 ARIMA(Auto-Regressive Integrated Moving Average)라는 그 유래가 1930년대까지 거슬러 올라가는 클래식(?)한 시계열 예측 모델을 파이썬으로 적용하고 구현하는 방법에 대해서 정리하고자 한다.   



[World Bank Blog]: https://blogs.worldbank.org/impactevaluations/evaluating-regulatory-reforms-using-the-synthetic-control-method
[CausalImpact]: https://google.github.io/CausalImpact/CausalImpact.html
[Prophet]: https://facebook.github.io/prophet/
[Tableau Forecasting] : https://help.tableau.com/current/pro/desktop/en-us/forecast_how_it_works.htm
