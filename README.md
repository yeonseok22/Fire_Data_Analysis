# Fire-Data-Analysis
- 1997년 ~ 2019년 계절별 산불 발생 현황 시계열 분석

### Stacks & Skills
<img alt="R" src ="https://img.shields.io/badge/R-276DC3.svg?&style=for-the-badge&logo=R&logoColor=white"/> 

### 목차
---
[R코드 전문](https://github.com/yeonseoksong/Fire_Data_Analysis/blob/main/Fire_data_analysis/Fire_data_analysis.R)
1. 시계열 분석 배경
2. 시계열 자료 변환 및 시계열 그림 그리기
3. 모형의 식별
4. 모형의 추정
5. 모형의 진단
6. 과대적합
7. 결론
8. 참고자료
---

### 1. 시계열 분석 배경
- 시계열 분석할 자료 : 1997년부터 2019년 계절별 산불발생 현황(출처 : kosis 국가통계포털)
- 자료 선정 이유
  - 지난 4월에 경북 안동에서 대형 산불이 발생하였고, 5월에는 강원 고성에서 대형 산불이 발생하여 큰 피해가 일어났다. 특히 강원 고성에서 발생한 산불은 한 주택에서 붙은 불이 강풍을 타고 인근 야산에 옮겨붙게 되어 크게 확산되었고, 인명피해는 없었지만 24억 5900만원에 해당하는 산림피해가 발생하였고 주택 7채, 비닐하우스 6동, 창고 2동 등의 재산피해가 발생하였다.
  - 산불이 일어나게 되면 장비와 급수, 지형적인 요소와 인원을 동원하는 등의 악조건이 있기 때문에 짧은 시간에 효과적으로 산불을 진화하기 어렵고 바람에 의하여 넓은 면적으로 확대되는 경우가 많다. 산불을 진화하는 것이 쉽지 않기 때문에 산불을 예방하는 것이 매우 중요하다.

  - 그래서 어느 계절에 산불이 많이 발생하는지 알아보면 효과적으로 산불을 예방하는 데 도움이 될 것이다.

  - 따라서 시계열 분석 및 실습 강의에서 배운 내용을 바탕으로 자료를 분석하여 산불을 예방하는 데 유용한 자료로 쓸 수 있도록 국가통계포털에서 계절별 산불발생 현황 자료를 선정하였다.

  - [대전CBS 고형석 기자, 강원도 고성 등 대형 산불 피해 복구 703억 투입  (검색 : 2020.06.07.)](https://www.nocutnews.co.kr/news/5353254)


### 2. 시계열 자료 변환 및 시계열 그림 그리기
- 참고 : 국가통계포털에서 다운받은 계절별 산불발생 현황 자료에서 계절별 합계와 '10년 평균'에 대한 내용는 분석할 때 사용하지 않으므로 미리 제거하였다.

```R
#install.packages('astsa')
#install.packages('lmtest')
#install.packages('tseries')   

library(astsa)
library(lmtest)
library(tseries)

dir <- choose.dir()   # 작업 디렉터리 지정
setwd(dir)
data <- read.table('계절별 산불발생 현황.txt', header = T, fileEncoding = 'utf-8') # raw data
```
![image](https://user-images.githubusercontent.com/49339278/147550153-a9e121ac-121b-4383-b5e1-296c95ea3c12.png)

- 이후 계절별, 기간별 표시 제거하고, 시계열 자료로 변환한다.

```R
data <- data[, -2:-1]  # 계절별, 기간별 표시 제거
data <- unlist(data, use.names	=FALSE); View(data)  # 시계열자료로 변환 위해 데이터 변환 실시
fire <- ts(data, start=c(1997,1), end=c(2019,4), frequency = 4)  # 시계열 자료로 변환
```

### 3. 모형의 식별
- 먼저 시계열 그림과 표본상관도표를 이용한다.
  - 시계열 그림 그리기
  ```R
  ts.plot(fire, xlim=c(1997,2020), xlab = 'year')  # 시계열 그림 그리기
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147550483-780fdce4-8c9c-45af-b8d6-cad47c0b3443.png)

  - 표본상관도표 그리기
  ```R
  acf2(fire)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147550492-be2d26d7-fb62-4d39-a80d-694876066790.png)
  ![image](https://user-images.githubusercontent.com/49339278/147550511-0d3c9837-6792-4eb5-a2f6-b2f3dc23b482.png)

- 시계열 그림과 표본상관도표를 분석한 결과, 변동폭이 크기 때문에 분산안정화 변환이 필요하다.
  - 분산안정화 변환을 하기 위해 로그변환을 실시한다.
  ```R
  lgfire <- log(fire)  # 로그 변환
  ts.plot(lgfire, xlim=c(1997,2020), xlab='year')  # 시계열 그림 그리기
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147550726-9247a114-0caa-4522-a837-9ae3b3533652.png)

  ```R
  acf2(lgfire, max.lag=36)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147550748-584cb24a-9e31-45a9-a42d-1f4df3539e2d.png)
  ![image](https://user-images.githubusercontent.com/49339278/147550756-5f20e466-24c1-45ab-8e8f-9be8202c44b5.png)

- 추세를 구분하기 위해서 ```Dicky-Fuller 단위근 검정```을 실시하였다.
  - 이를 위해 ```adf.test()``` 함수를 사용하였다.
  ```R
  adf.test(lgfire)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147550917-bbafd05b-fa77-4a73-a709-fe60d99dd75f.png)

- ```Dicky-Fuller 단위근 검정```에서 p-value가 0.05보다 크므로 귀무가설을 채택하였다. 따라서 정상성을 가질 수 없다.
  - 그러므로 분석을 위해 차분을 실시하기로 하였다.
  ```R
  dif_1 <- diff(lgfire)  # 1차 차분 실시
  ts.plot(dif_1, xlab='year')  # 시계열 그림 그리기
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147551092-f1a24706-7620-4900-965a-2b45f2a02193.png)

  ```R
  acf2(dif_1, max.lag=36)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147551132-5de289ac-b5c8-425e-919a-671fff743bfb.png)
  ![image](https://user-images.githubusercontent.com/49339278/147551137-a99aceb3-e5a3-45c7-b57c-6cf6267d274a.png)

- 표본상관도표를 분석한 결과, 주기가 4인 계절성분을 가지고 있는 것으로 판단되어 계절차분을 실시한다.
  ```R
  dif_14 <- diff(dif_1, lag=4)  # 주기가 4인 계절 차분 실시
  ts.plot(dif_14, xlab='year')  # 시계열 그림 그리기
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147551412-e5afabc1-4b85-401c-8794-fec3532f726f.png)

  ```R
  acf <- acf2(dif_14, max.lag=36); acf
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147551569-db542a78-cbd0-4893-944d-912638e54b31.png)
  ![image](https://user-images.githubusercontent.com/49339278/147551448-3c493ab7-75c2-43ff-a345-880b16f37915.png)


### 4. 모형의 추정

### 5. 모형의 진단

### 6. 과대적합

### 7. 결론

### 8. 참고자료
