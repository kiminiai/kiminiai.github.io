---
title: "로지스틱 회귀에서 회귀계수와 오즈비를 해석하는 방법: 채용·이직·은행 데이터 실습"
date: 2026-07-11
slug: logistic-regression-odds-ratio
categories:
  - Machine Learning
tags:
  - Logistic Regression
  - Odds Ratio
  - HR Analytics
  - Classification
  - Statsmodels
draft: false
---

# 로지스틱 회귀는 왜 계수를 그대로 해석하기 어려울까?

이번 수업에서는 은행 정기예금 가입, 공채 서류 합격, 타이타닉 생존, 직원 이직 여부를 로지스틱 회귀로 분석했다.

종속변수가 연속형이 아니라 0과 1이기 때문에 선형회귀 대신 로지스틱 회귀를 사용했다.

![로지스틱 회귀 해석 흐름](/img/statistics-day/12-logistic-workflow.png)

로지스틱 회귀의 계수는 확률의 직접적인 증가량이 아니라 **로그 오즈(log-odds)의 변화량**이다.

그래서 실무적으로 해석할 때는 계수에 지수함수를 적용한 오즈비를 자주 사용한다.

```python
odds_ratio = np.exp(model.params)
```

# 1. 대기업 공채 서류 합격 여부 예측

250명의 지원자 데이터를 이용해 서류 합격 여부를 예측했다.

```python
import statsmodels.formula.api as smf

formula = """
doc_pass
~ toeic_score
+ gpa
+ intern_count
+ cert_count
"""

model = smf.logit(formula=formula, data=df).fit(disp=False)
```

회귀계수와 p-value는 다음과 같았다.

| 변수 | 회귀계수 | p-value |
|---|---:|---:|
| toeic_score | 0.0041 | 0.0484 |
| gpa | 1.4085 | 0.0021 |
| intern_count | 0.6128 | 0.0000 |
| cert_count | 0.3361 | 0.0049 |

![공채 서류 합격 로지스틱 회귀계수](/img/statistics-day/13-recruit-logit-coef.png)

네 변수 모두 5% 유의수준에서 유의했다. 회귀계수만 보면 GPA가 가장 컸다.

GPA의 오즈비는 다음처럼 계산할 수 있다.

```python
import numpy as np

gpa_odds_ratio = np.exp(model.params["gpa"])
print(gpa_odds_ratio)
```

계수 1.4085의 오즈비는 약 4.09다. 다른 변수가 같다는 가정에서 GPA가 1점 증가하면 서류 합격 오즈가 약 4.09배가 된다고 해석할 수 있다.

여기서 **합격 확률이 4.09배가 된다는 뜻은 아니다.** 오즈와 확률은 서로 다르다.

```text
오즈 = 확률 / (1 - 확률)
```

모형의 정확도는 0.848이었다.

```python
from sklearn.metrics import accuracy_score

pred_prob = model.predict(df)
pred_class = (pred_prob >= 0.5).astype(int)

accuracy = accuracy_score(df["doc_pass"], pred_class)
```

다만 같은 데이터로 학습하고 평가한 정확도이므로 실제 일반화 성능보다 높게 나왔을 가능성이 있다. 취업용 프로젝트라면 train/test split 또는 교차검증을 추가하는 것이 더 적절하다.

# 2. 은행 정기예금 가입 예측

은행 고객의 나이, 잔액, 마케팅 접촉 횟수로 정기예금 가입 여부를 예측했다.

```python
import statsmodels.api as sm

X_train = train[["age", "balance", "campaign"]]
X_train = sm.add_constant(X_train)

y_train = train["y"]

model = sm.Logit(y_train, X_train).fit()
```

balance의 오즈비는 반올림했을 때 1.000이었다.

이 숫자만 보면 잔액이 가입에 전혀 영향을 주지 않는 것처럼 보일 수 있다. 하지만 balance가 1원 또는 1단위 증가할 때의 오즈비이기 때문에 단위가 너무 작아서 1에 가깝게 보일 수 있다.

예를 들어 1,000단위 증가 효과를 보고 싶다면 다음처럼 계산할 수 있다.

```python
beta_balance = model.params["balance"]

odds_ratio_1 = np.exp(beta_balance)
odds_ratio_1000 = np.exp(beta_balance * 1000)
```

모형의 잔차 이탈도는 25,732.546이었고 테스트 오분류율은 0.118이었다.

오분류율이 0.118이라는 것은 약 11.8%를 잘못 분류했다는 의미다.

# 3. 직원 이직 가능성 예측

IBM HR 데이터의 1,470명 직원 정보를 이용해 이직 여부를 분석했다.

사용한 변수는 다음과 같았다.

- Age
- Gender
- Education
- MonthlyIncome
- OverTime
- DistanceFromHome
- YearsAtCompany

문자열로 되어 있던 Attrition을 0과 1로 변환했다.

```python
df["Attrition_bin"] = (
    df["Attrition"].astype(str).str.strip() == "Yes"
).astype(int)
```

범주형 변수는 formula API의 `C()`를 사용했다.

```python
formula = """
Attrition_bin
~ Age
+ Education
+ MonthlyIncome
+ DistanceFromHome
+ YearsAtCompany
+ C(Gender)
+ C(OverTime)
"""

model = smf.logit(
    formula=formula,
    data=df
).fit(disp=False, maxiter=200)
```

Education이 1단계 증가할 때의 오즈비는 1.033이었다.

다른 조건이 같을 때 교육 수준이 1단계 증가하면 이직 오즈가 약 3.3% 높아지는 방향의 관계가 나타났다는 의미다.

다음 조건을 가진 직원의 이직 확률은 0.388로 예측됐다.

- 나이: 40세
- 성별: 남성
- 교육 수준: 3
- 월 급여: 3,000
- 초과 근무: Yes
- 출퇴근 거리: 10
- 근속 연수: 2년

```python
new_employee = pd.DataFrame({
    "Age": [40],
    "Gender": ["Male"],
    "Education": [3],
    "MonthlyIncome": [3000],
    "OverTime": ["Yes"],
    "DistanceFromHome": [10],
    "YearsAtCompany": [2]
})

attrition_probability = model.predict(new_employee).iloc[0]
```

이 결과를 실제 HR 의사결정에 사용한다면 0.388이라는 값만으로 직원을 위험군으로 단정하면 안 된다.

임계값을 0.5로 할지, 이직자를 놓치는 비용을 고려해 더 낮출지 결정해야 하며 정밀도와 재현율도 함께 확인해야 한다.

# 4. 구직자 데이터에서 정확도와 AUC 함께 보기

180명의 구직자 데이터로 최종 합격 여부를 예측한 실습에서는 다음 결과가 나왔다.

- age 계수: 0.0341, p-value 0.5226
- gpa 계수: 1.3748, p-value 0.0017
- toeic_score 계수: 0.0072, p-value 0.0003
- 정확도: 0.672
- AUC: 0.722

나이는 5% 유의수준에서 유의하지 않았고 GPA와 토익은 유의했다.

정확도와 AUC는 서로 다른 내용을 본다.

- 정확도: 정해진 임계값에서 전체 예측 중 맞힌 비율
- AUC: 임계값을 바꾸었을 때 양성과 음성을 구분하는 전반적인 능력

![로지스틱 회귀 실습 결과](/img/statistics-day/14-logistic-performance.png)

불균형 데이터에서는 정확도만 보면 문제가 생길 수 있다. 예를 들어 이직자가 16%라면 모든 직원을 잔류로 예측해도 정확도가 84%가 될 수 있다.

따라서 이직 예측에서는 다음 지표도 함께 봐야 한다.

```python
from sklearn.metrics import (
    confusion_matrix,
    classification_report,
    roc_auc_score
)

print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
print(roc_auc_score(y_test, pred_prob))
```

# 이번 실습에서 가장 헷갈렸던 부분

## 회귀계수와 확률 증가량은 다르다

로지스틱 회귀계수는 확률의 변화량이 아니다. 로그 오즈의 변화량이다.

## 오즈비 1은 변화가 거의 없다는 뜻이다

- 오즈비 > 1: 사건 발생 오즈 증가
- 오즈비 < 1: 사건 발생 오즈 감소
- 오즈비 = 1: 변화 없음

하지만 변수의 단위가 너무 작다면 오즈비가 반올림 때문에 1.000으로 보일 수 있다.

## p-value와 예측 성능은 다른 문제다

변수가 통계적으로 유의하더라도 분류 성능 향상 폭은 작을 수 있다. 반대로 p-value가 큰 변수도 다른 변수와 결합했을 때 예측에 도움이 될 수 있다.

# 마무리

이번 로지스틱 회귀 실습을 통해 분류 모델에서도 단순히 정확도만 보는 것이 아니라 변수의 영향력을 해석할 수 있다는 것을 알게 됐다.

앞으로 실제 프로젝트에서는 다음을 추가해야겠다.

1. 학습·검증·테스트 데이터 분리
2. 클래스 불균형 확인
3. 혼동행렬, Precision, Recall, F1, ROC-AUC 평가
4. 임계값 조정
5. 회귀계수와 오즈비 해석
6. 데이터 기반 의사결정에서 생길 수 있는 편향 점검
