---
title: "경사하강법에서 왜 gradient의 반대 방향으로 움직일까?"
slug: "gradient-descent-beta-update"
date: 2026-07-15
draft: false
categories:
  - Study
tags:
  - gradient-descent
  - linear-regression
  - optimization
  - numpy
description: "선형회귀를 직접 구현하며 beta_0와 beta_1 업데이트 식, gradient의 반대 방향으로 이동하는 이유를 직관적으로 설명합니다."
---

선형 회귀를 직접 구현하는 실습에서 가장 이해하기 어려웠던 부분은 다음 코드였다.

```python
delta_0 = -(lr * (2 / len(X)) * np.sum(y - y_pred))

delta_1 = -(lr * (2 / len(X)) *
            np.dot(X.T, (y - y_pred)))
```

그리고 이어지는 업데이트:

```python
beta_0 -= delta_0
beta_1 -= delta_1
```

처음에는 부호가 너무 많아 어디로 움직이는지조차 헷갈렸다.

## 먼저 우리가 원하는 것

선형 회귀식은 다음과 같다.

```text
y_pred = beta_0 + beta_1 * X
```

여기서:

- `beta_0`: 절편
- `beta_1`: 기울기

모델의 목표는 실제값과 예측값의 차이를 줄이는 것이다.

대표적인 손실 함수가 MSE다.

```text
MSE = 평균((y - y_pred)^2)
```

## gradient는 무엇인가?

gradient는 현재 위치에서 **손실이 가장 빠르게 증가하는 방향**을 알려준다.

따라서 손실을 줄이고 싶다면 반대로 가야 한다.

```text
새 파라미터
=
현재 파라미터
-
학습률 × gradient
```

![경사하강법](/img/data-analysis-study-notes/gradient-descent-flow.png)

### gradient가 양수일 때

오른쪽으로 갈수록 손실이 커진다는 뜻이다.

따라서 왼쪽으로 가야 한다.

### gradient가 음수일 때

오른쪽으로 갈수록 손실이 작아지는 방향이다.

따라서 파라미터 값은 증가해야 한다.

이 두 경우를 한 번에 처리하는 식이:

```text
parameter = parameter - lr * gradient
```

이다.

## 왜 beta_0는 오차의 합을 보는가?

`beta_0`는 절편이다.

절편을 바꾸면 회귀선 전체가 위아래로 이동한다.

따라서 전체 오차가 어느 방향으로 치우쳐 있는지가 중요하다.

```python
np.sum(y - y_pred)
```

실제값이 예측값보다 전반적으로 크다면 회귀선을 위로 올려야 한다.

반대라면 아래로 내려야 한다.

## 왜 beta_1에는 X가 곱해지는가?

`beta_1`은 기울기다.

기울기는 X 값이 큰 구간에서 더 큰 영향을 준다.

그래서 기울기 업데이트에는 오차뿐 아니라 입력 X도 함께 들어간다.

```python
np.dot(X.T, (y - y_pred))
```

직관적으로는:

```text
X가 큰 곳에서 발생한 오차
→ 기울기 수정에 더 큰 영향
```

이라고 이해할 수 있다.

## 학습률 lr은 왜 필요한가?

gradient 방향만 따라 한 번에 너무 크게 움직이면 최소점을 지나칠 수 있다.

그래서 한 번에 얼마나 움직일지 조절하는 값이 학습률이다.

```python
lr = 1e-4
```

### 너무 크면

- 발산할 수 있음
- 최소점을 계속 지나칠 수 있음

### 너무 작으면

- 학습이 매우 느림
- 충분한 반복 횟수가 필요함

## 직접 구현에서 얻은 가장 큰 이해

라이브러리를 쓰면 다음 한 줄이면 끝난다.

```python
model.fit(X, y)
```

하지만 직접 구현해보니 `fit()` 안에서는 결국:

```text
예측
↓
오차 계산
↓
gradient 계산
↓
파라미터 업데이트
↓
반복
```

이 일어난다는 것을 볼 수 있었다.

## 정리

경사하강법의 핵심은 어렵지 않다.

```text
gradient
= 가장 빠르게 올라가는 방향

-gradient
= 내려가는 방향
```

그래서 손실을 줄이기 위해:

```text
현재 값 - 학습률 × gradient
```

형태로 파라미터를 반복해서 수정한다.

`beta_0`는 전체 회귀선을 위아래로 움직이고, `beta_1`은 기울기를 바꾸기 때문에 두 파라미터의 gradient 식도 다르게 나타난다.
