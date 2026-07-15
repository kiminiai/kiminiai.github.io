---
title: "RSS는 왜 직접 구현할까? 잔차제곱합을 코드로 이해하기"
slug: "rss-residual-sum-of-squares"
date: 2026-07-15
draft: false
categories:
  - Study
tags:
  - RSS
  - regression
  - residual
  - evaluation
description: "회귀분석의 RSS를 for문으로 직접 구현하며 잔차, 제곱, 합의 의미를 이해합니다."
---

회귀 실습에서 다음 함수를 직접 작성했다.

```python
def return_RSS(test_y, predicted):

    RSS = 0

    for i in range(len(test_y)):
        RSS += (test_y[i] - predicted[i]) ** 2

    return RSS
```

처음에는 왜 이미 있는 평가지표를 쓰지 않고 직접 구현하는지 궁금했다.

하지만 코드를 한 줄씩 보니 RSS 수식을 그대로 옮긴 것이었다.

## RSS란?

RSS는 Residual Sum of Squares의 약자다.

한국어로는 잔차제곱합이다.

```text
RSS = Σ(실제값 - 예측값)^2
```

여기서 잔차는:

```text
실제값 - 예측값
```

이다.

![RSS 오차](/img/data-analysis-study-notes/rss-errors.png)

## 왜 제곱할까?

잔차를 그냥 더하면 양수와 음수가 상쇄될 수 있다.

예를 들어:

```text
오차 1: +10
오차 2: -10

합 = 0
```

하지만 예측이 완벽한 것은 아니다.

그래서 제곱한다.

```text
10^2 + (-10)^2 = 200
```

## 코드 한 줄씩 보기

```python
RSS = 0
```

누적할 값을 0으로 시작한다.

```python
for i in range(len(test_y)):
```

모든 테스트 샘플을 순회한다.

```python
test_y[i] - predicted[i]
```

i번째 실제값과 예측값의 차이를 구한다.

```python
(test_y[i] - predicted[i]) ** 2
```

잔차를 제곱한다.

```python
RSS += ...
```

각 샘플의 잔차제곱을 계속 더한다.

결국 수식 그대로다.

## numpy로 더 짧게 쓸 수도 있다

```python
rss = np.sum((test_y - predicted) ** 2)
```

두 코드는 본질적으로 같은 일을 한다.

직접 만든 함수는 원리를 학습하기 좋고, NumPy 방식은 실제 코드에서 더 간결하다.

## MSE와 무엇이 다른가?

MSE는 RSS를 데이터 개수로 나눈 값이다.

```text
MSE = RSS / n
```

즉:

```text
RSS
= 전체 오차 규모

MSE
= 샘플 1개당 평균적인 제곱오차
```

데이터 개수가 다른 모델 결과를 비교할 때는 MSE가 더 해석하기 편한 경우가 많다.

## 함수 인자 이름은 바꿔도 될까?

다음처럼 작성해도 문법적으로는 가능하다.

```python
def return_RSS(df1, df2):
```

하지만 좋은 이름은 아니다.

실제 역할을 드러내는 것이 낫다.

```python
def return_RSS(y_true, y_pred):
```

함수의 입력만 봐도 의미가 드러난다.

## 정리

RSS는 별도의 복잡한 개념이 아니다.

```text
실제값과 예측값의 차이
↓
제곱
↓
모든 샘플에 대해 합
```

이다.

직접 구현해보면 MSE, RMSE도 같은 계열의 지표라는 점이 자연스럽게 연결된다.

```text
RSS
↓ / n
MSE
↓ square root
RMSE
```

평가지표의 이름을 외우는 것보다 이 관계를 이해하는 것이 더 중요했다.
