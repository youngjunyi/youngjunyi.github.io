---
layout: post
title:  "3.탭파이(TabPy)로 데이터의 정규성(Normality) 검정하기"
date:   2020-01-26
categories: Analytics
---
&nbsp;&nbsp; 태블로에서는 여타 SQL에서 제공하는 것과 비슷한 종류의 여러가지 내장 함수(Built-in Functions)를 활용할 수 있다. 카테고리로 따지자면 흔히 'Logical', 'Aggregate', 'Table Calculations' 등의 함수를 자주 쓰게되는데 이를 통해 다양한 시각화 문제를 해결할 수 있다. 그러나 태블로는 이에 그치지 않고 외부 API를 활용하는 'External Service Connection'이란 기능을 제공하는데, 여기서 활용할 수 있는 서비스가 바로 [탭파이(TabPy)][TabPy Github]다. TabPy를 쓰게 되면 파이썬(Python) 스크립트를 태블로에서 직접 실행할 수 있기 때문에 그 활용 가능성이 무궁무진하다(R은 RServe라는 서비스를 쓰면 된다).

&nbsp;&nbsp; 이번 포스팅에서는 먼저 TabPy 설치 이후 로컬 서버에서의 실행 여부를 확인하고, NumPy와 SciPy의 내장함수를 통해 데이터의 왜도(Skewness), 첨도(Kurtosis), 그리고 정규성(Normality)을 알아보는 과정을 다루고자 한다. TabPy를 설치하는 방법은 [깃허브 블로그][TapPy Github Blog]에도 잘 나와있지만, 필자는 이 [포스팅][TapPy Installation]을 참고하여 Anaconda Powershell Prompt에서 설치를 진행했다. 설치한 패키지의 **startup.bat**파일을 Powershell에서 실행하면 다음 스크린샷과 같이 TabPy가 initialize된다. 
 
<img src="/assets/image/tabpy_init.PNG" width="50%" height="50%">&nbsp;&nbsp; 

&nbsp;&nbsp; 이 상태에서 태블로의 'Help > Settings and Performance > Manage External Service Connection' 메뉴로 들어가 **TabPy/External API**를 선택하고, Server는 'localhost', Port는 '9004'로 정의해주면 된다. 'Test Connection' 버튼을 눌러 다음 스크린샷과 같은 팝업 메세지가 나오면 연결된 것이다. 

<img src="/assets/image/tabpy_connected.PNG" width="50%" height="50%">&nbsp;&nbsp; 

&nbsp;&nbsp; 여기서 한 가지 주의할 점은 TabPy를 연결하여 사용하는 도중에 Localhost에 켜놓은 Powershell을 끄게되면 TabPy 연결이 끊어진다는 것이다. 별도의 서버에 TabPy가 initialize되어있다면 상관없겠지만, 여기서는 Localhost를 서버로 이용하고 있으니 Powershell을 끄면 연결이 끊어진다. 또한 태블로에서 파이썬 스크립트를 실행하면서 발생하는 Error Code들이 Shell에서 표시가 되니 TabPy 사용 중에 뭔가가 잘 안된다 싶으면 Shell을 확인해보도록 하자.

&nbsp;&nbsp; 이제 본격적으로 태블로에 분석할 데이터를 불러오고 TabPy를 이용해 파이썬 스크립트를 작성해보자. 필자는 모 구간의 항공권 가격 시계열 데이터를 준비했다. 먼저 테스트로 NumPy 라이브러리의 기본 함수인 mean과 median을 구현해보자. Calculated Field를 만들고 다음과 같은 스크립트를 작성한다.

```python
#price_mean 
  SCRIPT_REAL("import numpy as np
  return np.mean(_arg1)",
  sum([Price]))

#price_median 
  SCRIPT_REAL("import numpy as np
  return np.median(_arg1)",
  sum([Price]))
```

&nbsp;&nbsp; 여기서 **SCRIPT_REAL**이라는 것은 말 그대로 필드 내의 스크립트를 인식시켜주는 태블로의 function이고, 'REAL'은 이 스크립트가 불러오는 결과값의 Data Type이 'Float'이라는 것을 의미한다. 따라서 필요한 Data Type에 따라 'SCRIPT_BOOL', 'SCRIPT_INT', 'SCRIPT_STR' 등을 골라서 쓰면 된다. 또한 스크립트 내의 '(_arg1)'은 바로 NumPy 함수에 적용할 argument를 의미하는데, 그래서 스크립트의 끝에 'sum([Price])'를 해당 argument를 지정해 준 것을 볼 수 있다. 만약 2개의 arguments를 필요로 하는 함수라면 스크립트에서 '(_arg1, _arg2)'로 규정하면 된다.

&nbsp;&nbsp; NumPy의 mean, median함수를 써서 구한 값과 태블로의 내장 함수를 써서 구한 mean, median값이 같은지 확인해보자. 만약 다르다면, 'Edit Table Calculation' 메뉴를 통해 NumPy 함수가 적용된 필드가 내가 원하는 테이블의 범위를 참조하고 있는지 확인하여 그 이유를 알아낼 수 있다.

&nbsp;&nbsp; 다음으로는 [Skewness][SciPy Skewness]와 [Kurtosis][SciPy Kurtosis]를 알아보도록 하자. 이번에는 SciPy 라이브러리를 불러오면 된다.

```python
#price_skewness 
  SCRIPT_REAL("import scipy.stats as stats
  return stats.skew(_arg1)",
  sum([Price]))

#price_kurtosis
  SCRIPT_REAL("import scipy.stats as stats
  return stats.kurtosis(_arg1)",
  sum([Price]))
```

&nbsp;&nbsp; 물론 [태블로 커뮤니티][Tableau Comm]를 뒤져보면 태블로의 내장 함수를 이용해 Skewness와 Kurtosis를 계산하는게 가능하다는 것을 알 수 있다. 그리고 사실 태블로의 Summary를 이용하면 다음의 스크린샷과 같이 왠만한 통계치들은 빠르게 확인이 가능하다. 그러나 마치 Excel에서도 'SKEW'나 'KURT' 함수를 쓰는 것처럼, SciPy의 내장 함수를 이용하면 길고 복잡한 식을 작성할 필요가 없어 훨씬 더 간단하다. 또한 태블로의 Summary는 workbook에 구현된 테이블의 모든 범위를 대상으로 계산을 해주기 때문에 내가 원하는 대로 통계치를 계산하기 위해서는 TabPy를 활용하는 것이 더 유리하다.

<img src="/assets/image/tableau_summary.PNG" width="20%" height="20%">&nbsp;&nbsp; 

&nbsp;&nbsp; 그럼 이제 마지막으로 정규성 검정(Normality Test)을 위한 필드를 만들어보자. 정규성 검정은 태블로의 내장 함수만을 이용해서는 구현하기 불가능한 영역이기도 하다. 여기서는 SciPy 라이브러리에 있는 Jarque-Bera Test[Jarque-Bera Test]와 Sharpiro-Wilk Test[Shapiro-Wilk Test] 함수를 이용하여 귀무가설('데이터가 정규분포를 따른다')을 채택 또는 기각하기 위한 p-value를 출력하는 스크립트를 다음과 같이 만들었다.

```python
#jarque_bera_test
  SCRIPT_REAL("from scipy import stats
  return stats.jarque_bera(_arg1)[1]",
  sum([Price]))

#shapiro_wilk_test
  SCRIPT_REAL("from scipy import stats
  return stats.shapiro(_arg1)[1]",
  sum([Price]))
```

**NEXT**&nbsp;&nbsp; 

(1)Correlation에 관련된 함수를 태블로에서 구현하기

(2)Scaling/Normalizing에 관련된 함수를 태블로에서 구현하기

(2)파이썬으로 구현된 Forecasting 모델을 적용하여 태블로에서 시각화하기






[TabPy Github]: https://github.com/tableau/TabPy
[TapPy Github Blog]: https://tableau.github.io/TabPy/docs/server-install.html
[TapPy Installation]: https://www.theinformationlab.co.uk/2019/04/09/how-to-set-up-tabpy-in-tableau/
[SciPy Skewness]: https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.skew.html
[SciPy Kurtosis]: https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.kurtosis.html
[Tableau Comm]: https://community.tableau.com/thread/249623
[Jarque-Bera Test]: https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.jarque_bera.html
[Shapiro-Wilk Test]: https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.shapiro.html



