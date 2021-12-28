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

- ```Dicky-Fuller 단위근 검정```을 실시한다.
  ```R
  adf.test(dif_14)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147567891-bd4167c8-361d-4106-a0e5-6d2e59f848ec.png)

- ```Dicky-Fuller 단위근 검정```에서 p-value가 0.05보다 크므로 귀무가설을 기각하였다. 따라서 정상성을 만족한다고 할 수 있다.
- ```AR(p)``` 모형의 적절한 p 매개변수를 추정하기 위해 spacf를 식별한다.
  ```R
  sacf <- acf[1,] ; spacf <- acf[2,]   # SACF, SPACF
  pacf(dif_14, lag.max = 36)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568158-32d09e07-102b-4590-ba0c-342a832bd5e6.png) 
  - SPACF 식별법은 다음과 같다.
  - ![image](https://user-images.githubusercontent.com/49339278/147568139-06d55911-9b36-4299-8e04-e6cc5f133816.png)

  ```R
  se1 <- 1.96 /sqrt(length(dif_14)) # S.E
  check1 <- ifelse(abs(spacf) <= se1 , '채택', '기각') 
  estimated.ar <- t(as.matrix(data.frame(spacf, check1))) ; estimated.ar
  ```
  - ![image](https://user-images.githubusercontent.com/49339278/147568235-fa1dc49d-ef32-4595-8022-c16d393530e9.png)
  - 결과 : V17 이후의 값은 표준오차보다 작기 때문에 ```AR(3)``` 과정을 적합하는 것을 고려해 보는 것으로 결정한다.

  - ```MA(q)``` 모형의 적절한 q 매개변수를 추정하기 위해 sacf를 식별한다.
  - ![image](https://user-images.githubusercontent.com/49339278/147568388-88e7cee4-bc82-4a82-934d-7cab91c6aa07.png)
  ```R
  se2 <- rep(0,36)
  se2[1] <- 1/length(dif_14)
  for (i in 1:36) {
    se2[i+1] <- se2[i] + (2*(sacf[i]^2))/length(dif_14)
  };se2 <- sqrt(se2[1:36]);se2 # S.E(rho_k.hat) 계산
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568420-f723cbd0-e633-4503-99d3-7fa9995ea8dd.png)

  ```R
  check2 <- ifelse(abs(sacf) >= 1.96 * se2, '기각', '채택') 
  estimated.ma <- t(as.matrix(data.frame(sacf, check2))) ; View(estimated.ma)
  acf(dif_14, lag.max=36)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568435-64465897-c8e5-4576-8a10-41615578e0e7.png)
  ![image](https://user-images.githubusercontent.com/49339278/147568437-6a041a61-cf5c-45da-8c51-8cb2c6c46fba.png)
  - 결과 : V5 이후로 0으로의 절단 형태가 나타나므로 ```MA(1)```를 적합하는 것을 고려해 보는 것으로 결정한다.

- 간결의 원칙을 고려하였을 때, 간단한 모형인 ```MA(1)```을 적합하는 것으로 결정하였다.

### 4. 모형의 추정
- 모형 1 : 로그변환한 ![image](https://user-images.githubusercontent.com/49339278/147568522-8a816920-a7fb-4492-baa3-4075fef26641.png) 적용하기
  ```R
  fit1 <- arima(lgfire, c(0,1,1), seasonal = list(order=c(0, 1, 0), period=4))
  fit
  coeftest(fit1, df=length(fit1))
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568598-dd35cdb7-b0cf-45e7-b54e-ed71080bb91d.png)

- 모형 2 : 로그변환 후 1차 차분하고 계절차분 실시한 ```ARIMA(3,1,0), (0,1,0)_4``` 적용
  ```R
  fit2 <- arima(lgfire, c(3,1,0), seasonal=list(order=c(0,1,0), period=4))
  fit2
  coeftest(fit2, df=length(fit2))
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568719-87ce1f54-aac7-47c6-8e27-c1b6df1e18c2.png)

- 결과 : 모형 1과 모형 2를 고려하였을 때, 모형 1에서 모수 t값은 ![image](https://user-images.githubusercontent.com/49339278/147568754-aefc5bc5-ae07-437b-aa04-edddb91edaeb.png)으로 유의하지만, 모형 2의 두 번째 모수와 세 번째 모수는 0.12583과 0.08261으로 유의하지 않다. 또한 분산을 비교해 보았을 때, 모형 1(0.9628)이 모형 2(1.118)보다 분산이 작은 것을 알 수 있다.

- 모형 적합 결과

|모수|추정값|표준오차|t-값|유의확률|
|:--:|:--:|:--:|:---:|:--:|
|![image](https://user-images.githubusercontent.com/49339278/147568874-62d738ee-81d7-4722-8edf-6a96caf7e2a1.png)|-0.999999|0.033316|-30.017|4.14e-14|

- 따라서 적합된 모형은 다음과 같다.
- ![image](https://user-images.githubusercontent.com/49339278/147568933-c19cfc14-3cab-4403-b552-272dc2c1a93e.png)


### 5. 모형의 진단
- 방법 1
  ```R
  # dev.off()
  par(mfrow=c(2,3))
  ts.plot(resid(fit1),main = '잔차 시계열그림', type='o');abline(h=0) # 시계열그림
  acf(resid(fit1), main = 'Residuals SACF');pacf(resid(fit), main = 'Residuals SPACF')
  qqnorm(resid(fit1), main = '잔차 정규성검정');qqline(resid(fit1), col='blue')

  pvalue <- rep(0,24)
  for (i in 1:24) {pvalue[i] <- Box.test(resid(fit1),i, type='Lj')$p.value} # 포트맨토검정 
  plot(1:24, pvalue, xlab = 'Time', ylim= c(-0.1, 1.0), main = '포트맨토 검정')
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568966-9961b7a3-411b-4df5-be31-99d264f80bc2.png)

  ```R
  jarque.bera.test(resid(fit1)) # Jarque-Bera 검정통계량 이용
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147568980-61e43d67-0dd9-45c6-8c1e-830d5937af66.png)

- 정규성검정 결과로는 뒷부분을 제외하고 거의 직선상에 위치하기 때문에 정규분포를 따른다고 볼 수 있다.
- Jarque-Bera 검정통계량 이용한 정규성검정에서 p-value가 0.05보다 큰 값이 나왔기 때문에 귀무가설을 기각할 수 없고, 따라서 정규분포라고 할 수 있다.
- 포트멘토검정 결과, 시차1, 2, 3에서 유의확률 0.05 이상이므로 오차들이 백색잡음과정이라는 귀무가설을 기각할 수 없어 ARIMA(0,1,1), (0,1,0)_4모형이 적합하다고 판단할 수 있다.
- 백색잡음 가정 확인을 위하여 잔차의 시계열그림과 잔차의 표본자기상관계수(residual SACF ; RSACF)와 잔차의 표본부분자기상관계수(residual SPACF ; RSPACF)을 확인해야 한다. 여기서 잔차시계열이 백색잡음과정을 따른다면 RSACF와 RSPACF가 모든 시차에서 0과 가까운 값을 가져야 하지만, 그렇지 않기 때문에 백색잡음과정에 해당하지 않는다. 그리고 잔차의 산점도를 보았을 때 회귀모형에 부가된 가정에 위배되지 않아 모형 선택이 적절하다.

- 방법 2
  ```R
  sarima(lgfire, 0, 1, 1, D=1, S=4)
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147569026-7de0d6f7-c3d5-4928-ae14-a3dda10e0976.png)
  ![image](https://user-images.githubusercontent.com/49339278/147569031-2edc9641-3fdc-4c72-ad28-ecd5d69c7a4c.png)
  ![image](https://user-images.githubusercontent.com/49339278/147569036-abf24b4d-c844-4e2c-95da-8da64989bcc1.png)
  - QQplot으로 정규성 검증을 다시 확인해 본 결과, 방법 1에서의 QQplot보다 더 자세한 결과가 출력이 되었다.


### 6. 과대적합
- 모형 3 : 로그변환한 ```ARIMA(0,1,2),(0,1,0)_4``` 적용하기 # 과대 적합
  ```R
  fit3 <- arima(lgfire, c(0,1,2), seasonal = list(order=c(0,1,0), period=4))
  fit3
  coeftest(fit3, df=length(fit3))
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147569142-20a13cf9-c1ce-4ce2-9fcd-1028146e0867.png)

- 모형 4 : 로그변환한 ```ARIMA(1,1,1), (0,1,0)_4``` 적용하기 # 과대 적합
  ```R
  fit4 <- arima(lgfire, c(1,1,1), seasonal = list(order=c(0,1,0), period=4))
  fit4 ; coeftest(fit4, df=length(fit4))
  ```
 ![image](https://user-images.githubusercontent.com/49339278/147569192-88fbf20d-b692-420f-84f1-03da9adc6395.png)
 
### 7. 결론
- AIC 값 비교하기
  ```R
  aic <- c(fit1$aic, fit2$aic, fit3$aic, fit4$aic) ; aic
  ```
  ![image](https://user-images.githubusercontent.com/49339278/147569229-e544eb6f-af2f-4758-b857-b7b849de9628.png)

- BIC 값 비교하기
  - 모형 1 : 2.855631, 모형 2 : 3.056111, 모형 3 : 2.891062, 모형 4 : 2.889166

- AIC와 BIC를 비교한 결과, 로그변환 후, ```ARIMA(0,1,1),(0,1,0)_4```을 적용한 모형이 가장 작았다.

- 따라서 최종적인 예측모형으로 ![image](https://user-images.githubusercontent.com/49339278/147569337-1c0e2899-5564-4187-bbab-767e01dd089a.png)을 선택하였다.

### 8. 참고자료
- SAS/ETS와 R을 이용한 시계열분석 (저자 : 조신섭)
- [대전CBS 고형석 기자, 강원도 고성 등 대형 산불 피해 복구 703억 투입  (검색 : 2020.06.07.)](https://www.nocutnews.co.kr/news/5353254)
-  자료 : 1997년부터 2019년 계절별 산불발생 현황(출처 : kosis 국가통계포털)
