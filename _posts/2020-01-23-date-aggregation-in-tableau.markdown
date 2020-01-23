---
layout: post
title:  "2.태블로에서 동적인 날짜(일/주/월 단위 Date Aggregation) 구현하기"
date:   2020-01-23
categories: Analytics
---
&nbsp;&nbsp; 대시보드를 만들 때 가장 많이 고민하는 부분은 'Aggregate & Disaggregate'의 균형을 어느 정도로 맞출 것이냐 하는 것이다. 이를테면 어떤 사람은 매출을 일 단위로 알아야 하겠지만, 다른 사람들은 주 단위 매출을 이해하는 것으로 충분할 수 있다. 또한 전년 대비(YoY) 성장률을 주 단위로 알고 싶을 때가 있는 한 편, 월 단위의 YoY가 필요한 상황도 빈번하다. 지표는 일/주/월 단위로 끊임없이 업데이트되고 사람들이 지표를 통해 얻고자 하는 정보는 그때마다 다르다. 그렇다고 그때그때 개별 니즈에 맞춘 새로운 대시보드를 만드는 것은 매우 비효율적인 일이다(<s>만드는 사람도 힘들고</s>). 그러나 태블로의 **Parameters**를 잘 활용하면 이러한 문제를 어느 정도 해결할 수 있다. 이번 포스팅에서는 가장 기본이 되는 일/주/월 단위의 'Data Aggregation'을 구현해보도록 하자. 먼저 다음과 같이 **Date Aggregation**이라는 파라미터를 만들자.

<img src="/assets/image/date_agg_parameter_1.PNG" width="40%" height="40%">&nbsp;&nbsp;  

&nbsp;&nbsp; Data type:으로는 "String"을, Allowable values:에서는 "List"를 선택한다. 그리고 List of values를 상단 스크린샷과 같이 입력해주면 하단 스크린샷과 같이 생성된 파라미터를 확인할 수 있다. 

<img src="/assets/image/date_agg_parameter_2.PNG" width="20%" height="20%">&nbsp;&nbsp; 

&nbsp;&nbsp;그럼 이제 List of values에서 입력한 'day','week','month'에 실제로 대응할 디멘션을 만들 차례다. 여기서는 데이터 소스에서 제공하는 날짜 디멘션을 'Date'로 가정하여 사용한다. 'Create Calculated Field'기능을 이용해 다음과 같이 **Date Aggregated**라는 디멘션을 생성해보도록 하자.

```SQL
#Date Aggregated 
  CASE [Date Aggregation]
    WHEN 'day' THEN DATETRUNC('day', [Date])
    WHEN 'week' THEN DATETRUNC('week', [Date])
    WHEN 'month' THEN DATETRUNC('month', [Date])
  END
```

&nbsp;&nbsp;CASE 구문을 활용하면 이미 만들어 놓은 **Date Aggregation**이라는 파라미터의 값에 따라 'Date' 디멘션이 일/주/월간 단위로 선택되도록 설정할 수 있다. 이렇게 만든 **Date Aggregated**라는 디멘션을 목적에 따라 Columns 또는 Rows로 배치한 뒤에, 다음 스크린샷과 같이 'Exact Date'를 선택하면 끝이다. 

<img src="/assets/image/exact_date.PNG" width="20%" height="20%">&nbsp;&nbsp; 

&nbsp;&nbsp;그러면 다음과 같이 파라미터를 조정하는대로 날짜의 Aggregation Level이 변하는 것을 볼 수 있을 것이다.
<img src="/assets/image/date_aggregation.gif" width="70%" height="70%">&nbsp;&nbsp; 

&nbsp;&nbsp;여기서 태블로가 제공하는 'Quick Table Calculation' > 'Percent Difference'를 이용하면 WoW와 MoM을 쉽게 구현할 수 있다. **Date Aggregation** 파라미터에서 'Weekly'를 선택하면 'Percent Difference'가 바로 WoW이고 'Monthly'를 선택하면 MoM이 되는 것이다. YoY는 어떨까? 필자의 경우에는 다음과 같은 Calculated Field를 통해 구현했다.  

```SQL
#Sessions prev year
  IF ([Date Aggregation] = 'day') THEN LOOKUP(zn(sum([Sessions])),-364)
  ELSEIF ([Date Aggregation] = 'week') THEN LOOKUP(zn(sum([Sessions])),-52)
  ELSEIF ([Date Aggregation] = 'month') THEN LOOKUP(zn(sum([Sessions])),-12)
  END
 
#Sessions YoY
  (sum([Sessions])/[Sessions prev year]) - 1
```
&nbsp;&nbsp;LOOKUP 구문을 활용하여 원하는 지표(여기서는 Sessions)의 전년도 값을 찾을 수 있고, 이렇게 만든 'Sessions prev year'라는 필드를 'Sessions'필드와 비교하는 'Sessions YoY' 필드를 추가적으로 생성하여 **Date Aggregation**파라미터에 동적으로 반응하는 YoY를 구현할 수 있다. 




