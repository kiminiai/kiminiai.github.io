---
title: "다중선형회귀를 실제 데이터에 적용하며 알게 된 것: 계수, p-value, R²와 예측 오차"
date: 2026-07-11
slug: multiple-linear-regression-practice
categories:
  - Machine Learning
tags:
  - Regression
  - OLS
  - Statsmodels
  - R-squared
  - Python
draft: false
---

# 회귀분석은 예측만 하는 모델이 아니었다

머신러닝에서 회귀를 배울 때는 RMSE나 R²가 높은지부터 확인했다.

이번에는 statsmodels를 이용해 회귀계수와 p-value를 함께 확인하면서 조금 다른 관점으로 접근했다.

> 예측 성능뿐 아니라 어떤 변수가 결과에 어떤 방향으로 영향을 주며, 그 영향이 통계적으로 유의한지도 확인할 수 있었다.

![다중선형회귀 분석 흐름](/img/statistics-day/09-regression-workflow.png)

# 1. 영업 사원의 월매출에 영향을 주는 요인

200명의 영업 사원 데이터를 이용해 월매출을 예측했다.

독립변수는 다음 네 가지였다.

- 월간 영업 콜 횟수
- 고객 방문 횟수
- 영업 경력
- 교육 점수

```python
import statsmodels.formula.api as smf

formula = """
monthly_sales
~ call_count
+ visit_count
+ work_years
+ training_score
"""

model = smf.ols(formula=formula, data=df).fit()
print(model.summary())
```

회귀계수는 다음과 같았다.

| 변수 | 회귀계수 | p-value |
|---|---:|---:|
| call_count | 8.6052 | 0.000 |
| visit_count | 15.7719 | 0.000 |
| work_years | 29.1144 | 0.000 |
| training_score | 4.2019 | 0.000 |

![영업 사원 월매출 회귀계수](/img/statistics-day/10-sales-coefficients.png)

모든 독립변수의 p-value가 0.05보다 작았다. 회귀계수만 보면 영업 경력 1년 증가가 월매출과 가장 큰 양의 관계를 보였다.

하지만 여기서 단순히 계수의 숫자만 비교하면 주의해야 한다. 각 변수의 단위가 다르기 때문이다.

- 경력은 1년 단위
- 콜 횟수는 1회 단위
- 교육 점수는 1점 단위

단위가 다른 변수의 영향력을 직접 비교하려면 표준화 회귀계수나 변수 범위를 함께 확인하는 것이 더 적절하다.

모형의 R²는 0.940, 수정된 R²는 0.939였다. 네 변수가 월매출 변동의 약 94%를 설명했다.

```python
print("R²:", model.rsquared)
print("Adjusted R²:", model.rsquared_adj)
```

R²와 수정된 R² 차이가 크지 않았기 때문에 불필요한 변수를 무리하게 추가한 모형은 아니라고 볼 수 있었다.

# 2. 팁 금액은 식사 금액만으로 설명할 수 있을까?

레스토랑의 244건 식사 기록을 이용해 팁 금액을 분석했다.

```python
formula = """
tip
~ total_bill
+ size
+ C(sex)
+ C(smoker)
+ C(day)
+ C(time)
"""

tip_model = smf.ols(formula=formula, data=tips).fit()
```

`C()`를 사용한 변수는 범주형 변수로 처리된다. 기준 범주와 비교한 더미변수의 회귀계수가 생성된다.

이 모형의 R²는 0.470이었다.

즉, 사용한 변수들이 팁 금액 변동의 약 47%를 설명했다. 47%가 낮다고 무조건 나쁜 모델은 아니다. 팁에는 서비스 만족도, 직원의 응대, 고객 성향처럼 데이터에 포함되지 않은 요인이 많기 때문이다.

# 3. 의류 제조업 생산성 예측

이번에는 수치형 변수와 범주형 변수가 함께 있는 데이터를 사용했다.

```python
formula = """
productivity
~ hours_worked
+ age
+ experience
+ C(department)
+ C(day)
+ C(quarter)
+ C(education)
+ C(designation)
"""

model = smf.ols(formula=formula, data=train).fit()
```

절편을 제외한 회귀계수 절댓값 중 두 번째로 큰 변수는 `C(quarter)[T.Quarter4]`였다.

p-value가 0.05보다 큰 비유의 변수는 12개였고, 테스트 데이터 MSE는 0.0273이었다.

```python
from sklearn.metrics import mean_squared_error

y_pred = model.predict(test)
mse = mean_squared_error(test["productivity"], y_pred)

print(round(mse, 4))
```

이 결과를 보며 변수를 많이 넣는다고 모든 변수가 의미를 갖는 것은 아니라는 것을 확인했다.

다만 p-value가 크다는 이유만으로 변수를 바로 삭제하면 안 된다. 예측 성능이 목적이라면 교차검증 결과와 도메인 지식도 함께 봐야 한다.

# 4. 콘크리트 압축 강도 예측

1,030개의 콘크리트 샘플을 학습 데이터와 테스트 데이터로 나누어 다중선형회귀를 적용했다.

```python
import statsmodels.api as sm
from sklearn.metrics import mean_squared_error
import numpy as np

X_train = sm.add_constant(train[feature_cols])
y_train = train["strength"]

model = sm.OLS(y_train, X_train).fit()

train_pred = model.predict(X_train)

X_test = sm.add_constant(test[feature_cols], has_constant="add")
test_pred = model.predict(X_test)

rmse = np.sqrt(mean_squared_error(test["strength"], test_pred))
```

결과는 다음과 같았다.

- 통계적으로 유의한 독립변수: 7개
- 학습 데이터 실제값과 예측값의 피어슨 상관계수: 0.780
- 테스트 RMSE: 8.56 MPa

학습 데이터의 상관계수가 높아도 테스트 데이터 오차가 작다는 보장은 없다. 그래서 학습 적합도와 테스트 성능을 따로 확인해야 한다.

# 5. 주택 가격 예측에서 유의한 변수만 다시 사용하기

Ames 주택 데이터에서 10개 변수를 사용해 SalePrice를 예측했다.

첫 번째 모형에서 p-value가 0.05보다 작은 변수만 선택한 뒤 두 번째 모형을 다시 적합했다.

```python
import statsmodels.api as sm

X = sm.add_constant(df[features])
y = df["SalePrice"]

model1 = sm.OLS(y, X).fit()

p_values = model1.pvalues.drop("const")
significant_features = p_values[p_values < 0.05].index.tolist()

X2 = sm.add_constant(df[significant_features])
model2 = sm.OLS(y, X2).fit()
```

유의한 변수는 다음과 같았다.

- OverallQual
- OverallCond
- GrLivArea
- TotalBsmtSF
- 1stFlrSF
- GarageArea
- BedroomAbvGr
- TotRmsAbvGrd
- YearBuilt

FullBath는 5% 유의수준에서 제외됐다.

유의한 변수만 사용한 모형의 R²는 0.781이었고, 예시 조건의 주택 가격 예측값은 204,844.16달러였다.

![회귀 실습 성능 비교](/img/statistics-day/11-regression-performance.png)

# p-value가 작은 변수만 남기면 좋은 모델일까?

통계적 설명이 목적이라면 유의한 변수에 집중하는 것이 의미가 있다.

하지만 예측이 목적이라면 p-value만을 기준으로 변수를 제거하는 방식에는 한계가 있다.

- 변수 사이의 다중공선성
- 표본 수
- 범주형 변수의 기준 범주
- 비선형 관계
- 상호작용
- 테스트 데이터 일반화 성능

이런 요소를 함께 고려해야 한다.

머신러닝에서는 교차검증과 규제를 사용하고, 통계 모델에서는 계수의 신뢰구간과 가정을 확인한다. 둘 중 하나가 무조건 더 좋은 것이 아니라 분석 목적에 따라 강조점이 달랐다.

# 이번 실습에서 정리한 회귀 결과

| 데이터 | 주요 결과 |
|---|---|
| 레스토랑 팁 | R² 0.470 |
| 영업 사원 월매출 | R² 0.940, Adj. R² 0.939 |
| 의류 생산성 | 테스트 MSE 0.0273 |
| 콘크리트 강도 | 테스트 RMSE 8.56 |
| 주택 가격 | 유의 변수 모형 R² 0.781 |

이번 수업을 통해 회귀분석 결과를 볼 때 최소한 다음은 함께 확인해야 한다고 느꼈다.

1. 회귀계수의 방향과 단위
2. 변수별 p-value
3. R²와 수정된 R²
4. 학습 데이터 적합도
5. 테스트 데이터 오차
6. 모델의 가정과 해석 범위
