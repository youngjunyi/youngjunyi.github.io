---
layout: post
title:  "4.태블로에서 공간(Spatial)데이터 연결(Join)하기"
date:   2020-01-30
categories: Analytics
---
&nbsp;&nbsp; 태블로에서 지도에 데이터 시각화를 하다보면 다양한 타입의 공간 데이터를 다루게 된다. 그 중 가장 흔한 것이 아마 위도(latitude)와 경도(longitude)일테고, 더 나아가서는 Point, LineString, Polygon, MultiPolygon 등이 있다. 이번 포스팅에서는 특정 지점의 위치를 규정하는 위도와 경도 를 태블로에서 'Point' 타입으로 만들어 'MultiPolygon'과 Join하는 방법에 대해 간단하게 정리하고자 한다.  

&nbsp;&nbsp; 예시 데이터로 [Inside Airbnb][Inside Airbnb]라는 에어비앤비 리스팅 및 관련 지역 정보를 제공하는 사이트의 데이터를 가져왔다. Inside Airbnb에서는 여러 종류의 데이터를 제공하고 있지만, 포스팅의 목적에 맞게 (1)특정 도시 내 숙소의 위도와 경도 데이터를 가져올 **listings.csv** 파일, 그리고 (2)해당 도시의 구획 정보를 MultiPolygon 타입의 데이터로 불러올 **neighbourhoods.geojson**파일만 다운로드 받으면 된다. 필자는 코펜하겐을 대상 도시로 고르고 데이터를 다운로드 받았다. 그럼 이제 태블로에 데이터를 업로드하자. 

<img src="/assets/image/listings_csv.PNG" width="100%" height="100%">&nbsp;&nbsp; 

&nbsp;&nbsp; 첫 번째 파일은 단순하다. 숙소명과 id를 기준으로 위도와 경도값 등이 일반적인 테이블 형태로 저장되어있는 것을 볼 수 있다.

<img src="/assets/image/neighbourhoods.PNG" width="50%" height="50%">&nbsp;&nbsp; 

&nbsp;&nbsp; 두 번째 파일은 일단 csv파일이 아니라 [GeoJson][GeoJson]파일이다. 태블로에서 미리보기를 하면 테이블에 구획 이름(Neighbourhood)을 기준으로 'Geometry'컬럼에 'MULTIPOLYGON'이라고만 표시되고 실제 값은 보이지 않는다. 하지만 이 GeoJson파일을 메모장으로 열어보면 다음 스크린샷과 같이 보이는데, 여러 경도와 위도값들이 Json형식으로 nested되어 있는 것을 알 수 있다. 이 값을 태블로는 자동으로 'MultiPolygon' 데이터 타입으로 인식한다. 

<img src="/assets/image/geojson.PNG" width="50%" height="50%">&nbsp;&nbsp; 

&nbsp;&nbsp; 이제 태블로에서 첫 번째 파일의 숙소 '위치(위도와 경도)'를 두 번째 파일의 '구획'과 연결해보자. 연결하는 원리는 구획이라는 'MultiPolygon'타입 데이터에 위도와 경도값이 intersect할 수 있도록 'Point'타입의 데이터로 만들어주는 것이다. 태블로에서는 이를 구현할 수 있도록 **MAKEPOINT**라는 내장 함수를 제공한다.  

<img src="/assets/image/spatial_join.PNG" width="50%" height="50%">&nbsp;&nbsp; 

&nbsp;&nbsp; 첫 번째 파일에서 Join하고 싶은 컬럼을 고르는 드롭다운 버튼을 누르면 'Create Join Calculation'이라는 옵션이 있다. 이를 눌러서 다음과 같이 MAKEPOINT 함수를 쓰자. 

```sql
#Join Calculation
  MAKEPOINT([Latitude],[Longitude])
```

&nbsp;&nbsp; 이렇게 만든 'Join Calculation'을 두 번째 파일의 'Geometry' 컬럼과 연결하면 끝이다. 연결할 때 comparison operator를 '='이 아닌 'Intersects'로 선택하는 것을 잊지 말도록 하자. 


<img src="/assets/image/copenhagen.PNG" width="100%" height="100%">&nbsp;&nbsp; 

&nbsp;&nbsp; 예시로 코펜하겐에 리스팅 된 숙소들의 구획별 평균 가격(파란색 density)과 개별 숙소별 평균 가격(빨간색 density)을 시각화 한 모습이다. MAKEPOINT함수를 이용해 위도와 경도값을 MultiPolygon에 연결함으로써 특정 구획과 그 구획에 속하는 개별 숙소들의 데이터를 함께 다룰 수 있게 되었다.



[Inside Airbnb]: http://insideairbnb.com/get-the-data.html
[GeoJson]: https://geojson.org/


