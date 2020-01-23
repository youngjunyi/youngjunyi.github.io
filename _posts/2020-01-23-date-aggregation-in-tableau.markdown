---
layout: post
title:  "2.태블로에서 동적인 날짜(일/주/월 단위 Date Aggregation) 구현하기"
date:   2020-01-23
categories: Analytics
---
&nbsp;&nbsp;필자가 태블로를 쓰기 시작한지 얼마 안됐을 때는 일/주/월간 단위의 비즈니스 지표(방문, 매출 등) 대시보드 페이지를 개별적으로 만들곤 했다. WoW/MoM/YoY등의 계산 또한 일/주/월 단위로 달라지기 때문에 이를 개별 페이지에서 구현한 것이다. 이렇게 필요한 Date Aggregation 단위에 따라 페이지를 넘나 들면서 데이터를 확인하는 것은 여간 귀찮은 일이 아니다. 그러나 태블로의 **Parameters**를 이용하면 이 모든 것을 한 페이지 안에서 구현할 수 있다. 먼저 필요한 Parameter를 만들어보자.

<img src="/assets/image/date_agg_parameter.PNG" width="100%" height="100%">&nbsp;&nbsp;  
**<Figure 1. Date Aggregation Parameter>**
&nbsp;&nbsp; Data type:으로 "String"을 선택하고, Allowable values:에서 "List"를 체크한다. 그리고 List of values를 스크린샷과 같이 입력해주면 된다. 그럼 다음과 같은 Parameter가 생성된 것을 볼 수 있다.

<img src="/assets/image/date_agg_parameter_2.PNG" width="100%" height="100%">&nbsp;&nbsp; 




```sql
#SMA_7day 
  WINDOW_AVG(SUM([Sessions]), -6, 0)

#SMA_28day
  WINDOW_AVG(SUM([Sessions]), -27, 0)
  
#SMA_Crossover
  [SMA_7day] >= [SMA_28day]
```
