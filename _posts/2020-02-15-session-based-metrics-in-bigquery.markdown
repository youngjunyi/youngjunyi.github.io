---
layout: post
title:  "5.빅쿼리(BigQuery)와 세션 지표(Session-based Metrics)"
date:   2020-02-15
categories: Analytics
---
&nbsp;&nbsp; 구글 애널리틱스(Google Analytics)에서 볼 수 있는 데이터를 더 세부적으로 보고 싶을 때 빅쿼리(BigQuery)를 찾게 된다. 구글 애널리틱스에서는 데이터가 이미 날짜나 채널 등의 여러가지 단위로 요약이 되어있지만, 빅쿼리에서는 user, visitor, session, visit, hit 등의 레벨로 원하는 만큼 데이터를 파고 들어갈 수 있다. 그렇다면 보통 어떤 상황에서 이렇게 hit 레벨까지 들어가 디테일한 내역을 확인하게 되는 것일까? 마케팅 관점의 케이스로는 Product Activation 지표를 더 세밀하게 보고자 하는 경우가 있을 것이다. 예를 들어 어떤 이커머스 웹사이트에서 '검색'이라는 activation이 일어났다고 할 때, 그 activation에 묶여있는 '페이지 내 특정 버튼 클릭 여부'와 같은 세부적인 이벤트를 보고자 하는 것이다. 

&nbsp;&nbsp; 이번 포스팅에서는 빅쿼리에서 이러한 세부 이벤트를 불러와 세션 기반의 지표(Session-based Metrics)로 표현하는 방법에 대해 정리해보고자 한다. 여기서는 일반적으로 쓰이는 **"ga_sessions_YYYYMMDD"** 형식의 [BigQuery Export Schema][BigQuery Export Schema]내 테이블을 사용하는 것을 전제로 한다. 

&nbsp;&nbsp; 빅쿼리에서 이미 flatten되어있는 정규형 테이블만 쓰다가 ga_sessions_YYYYMMDD 테이블을 처음으로 썼을 때 **sessionId**라는 필드가 없어서 당황했던 기억이 난다. 그러나 sessionId는 fullVisitorId와 visitId의 조합일 뿐이니 하기와 같이 sessionId 필드를 만들어주면 된다. 

```sql
CONCAT(fullvisitorId,STRING(visitId)) AS sessionId
```

&nbsp;&nbsp; 그 다음으로는 **hits.type** 필드를 먼저 살펴보도록 하자. 이 필드에는 ["PAGE", "TRANSACTION", "ITEM", "EVENT", "SOCIAL", "APPVIEW", "EXCEPTION"] 중 하나의 값이 들어있다. 그 값이 "PAGE"라는 것은 말그대로 특정 PAGE를 hit했다는 것이고, "EVENT"라는 것은 어떤 페이지 내의 특정 EVENT를 hit했다는 의미이다. 예를 들어 어떤 이커머스 웹사이트에서 '검색'이라는 activation을 '/search/'라는 URL path를 가진 페이지에서만 일어나는 것으로 규정했다고 가정할 때, 다음과 같이 해당 activation을 구성하는 논리를 표현해 볼 수 있다.

```sql
hits.type = 'PAGE' AND REGEXP_MATCH(hits.page.pagePath, '/search/')
```
&nbsp;&nbsp; 또 다른 예로 해당 이커머스 웹사이트의 검색 결과 페이지에서 필터를 사용할 경우 'FilterApply'라는 이벤트 값을 기록하도록 규정했다고 가정할 때, 해당 논리는 다음과 같이 표현해 볼 수 있을 것이다. 

```sql
hits.type = 'EVENT' AND hits.event.eventCategory = 'FilterApply')
```

&nbsp;&nbsp; 이처럼 **hits.**라는 이름의 필드를 활용하면 웹사이트 내에 규정되어있는 여러가지 이벤트들을 포착할 수 있는데, 기본적으로 제공되는 컬럼들은 [BigQuery Export Schema 문서][BigQuery Export Schema]에 잘 정리가 되어있다. 이벤트 값들이 어떻게 기록되고 있는지 확실하지 않을 때는 [Google Analytics Debugger][GA Debugger]를 설치한 후 하단 스크린샷과 같이 직접 페이지와 이벤트를 로드해보면 해당 값들을 확인할 수 있다. 


<img src="/assets/image/ga_debugger.png" width="80%" height="80%">


&nbsp;&nbsp; 그럼 이제 이 세부 이벤트를 세션 기반의 지표(Session-based Metrics)로 표현하는 방법에 대해 알아보도록 하자. 여기서 우리는 (1)한 번의 세션에서 특정 이벤트가 '몇 번이나 일어났는지'가 궁금할 수도 있고(activation의 횟수), (2)이 세션이 해당 이벤트가 일어난 세션인지 아닌지의 여부(activation의 여부)가 궁금할 수도 있다. '검색'을 예로 들자면 (1)은 **Searches per Session**과 같은 지표로, (2)는 **Sessions with Search**같은 지표로 표현할 수 있는 것이다. 이 지표들을 만들기 위해 먼저 서브쿼리에 IF절을 이용해 다음과 같은 필드를 만들어 보자. 

```sql
IF(hits.type = 'PAGE' AND REGEXP_MATCH(hits.page.pagePath, '/search/'),1,0) AS Search
IF(hits.type = 'EVENT' AND hits.event.eventCategory = 'FilterApply',1,0) AS Filter
```

&nbsp;&nbsp; 그리고 이렇게 만든 서브쿼리의 필드들을 활용하여 하기와 같이 메인쿼리의 필드를 만들 수 있다. 

```sql
#(1) activation의 횟수

SUM(Search) AS SearchCount
SUM(Filter) AS FilterCount

#(2) activation의 여부

CASE WHEN SUM(Search) > 0 THEN 1 ELSE 0 AS Search
CASE WHEN SUM(Filter) > 0 THEN 1 ELSE 0 AS Filter
```

&nbsp;&nbsp; 이렇게 만든 메인쿼리의 필드를 활용하여 원하는 세션 기반의 지표를 만들어 보자. SearchCount를 전체 Session으로 나눠주면 Searches per Session라는 지표가, 그리고 'Search = 1'인 Session들이 바로 Sessions with Search라는 지표가 된다. 




[BigQuery Export Schema]: https://support.google.com/analytics/answer/3437719?hl=en
[Count of GA Sessions]: https://medium.com/@asafonau/count-of-google-analytics-sessions-using-google-bigquery-how-to-show-count-of-sessions-in-a-way-166337e2a458
[GA Debugger]: https://chrome.google.com/webstore/detail/google-analytics-debugger/jnkmfdileelhofjcijamephohjechhna?hl=en
