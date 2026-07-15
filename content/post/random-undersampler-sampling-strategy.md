---
title: "RandomUnderSampler는 왜 가장 적은 클래스 수에 맞춰질까?"
slug: "random-undersampler-sampling-strategy"
date: 2026-07-15
draft: false
categories:
  - Study
tags:
  - RandomUnderSampler
  - undersampling
  - imbalanced-data
  - sampling_strategy
description: "RandomUnderSampler의 기본 동작과 sampling_strategy를 직접 지정하는 방법, 언더샘플링의 장단점을 정리합니다."
---

불균형 데이터 실습에서 다음 코드를 실행했다.

```python
from imblearn.under_sampling import RandomUnderSampler

sampler = RandomUnderSampler(random_state=42)
X_res, y_res = sampler.fit_resample(X, y)
```

그런데 각 클래스가 똑같은 개수로 줄어들었다.

나는 "5개로 맞추라"고 한 적이 없는데 결과가 5개씩 나왔다.

이유는 간단했다.

> 기본 설정에서는 다수 클래스의 샘플 수를 가장 적은 클래스에 맞추는 방향으로 언더샘플링한다.

![RandomUnderSampler](/img/data-analysis-study-notes/random-undersampling.png)

## 언더샘플링이란?

예를 들어 클래스 분포가 다음과 같다고 하자.

```text
class 0: 1000개
class 1: 100개
class 2: 20개
```

가장 적은 클래스는 20개다.

기본적인 랜덤 언더샘플링을 적용하면 다수 클래스에서 일부 데이터를 제거해 다음과 같이 맞출 수 있다.

```text
class 0: 20개
class 1: 20개
class 2: 20개
```

## 왜 이런 방식이 필요한가?

불균형 데이터에서는 모델이 다수 클래스만 잘 맞춰도 높은 정확도를 얻을 수 있다.

예를 들어:

```text
정상: 95%
이상: 5%
```

인 데이터에서 모든 샘플을 정상이라고 예측해도 정확도는 95%다.

하지만 이상 탐지 모델로서는 사실상 실패다.

언더샘플링은 다수 클래스의 비중을 줄여 소수 클래스를 더 강하게 보게 만든다.

## sampling_strategy를 직접 지정할 수 있다

각 클래스의 목표 개수를 직접 정하고 싶다면 딕셔너리를 사용할 수 있다.

```python
sampling = {
    3: 50,
    4: 50,
    5: 50,
    6: 50,
    7: 50
}

sampler = RandomUnderSampler(
    random_state=42,
    sampling_strategy=sampling
)
```

이제 각 클래스는 지정한 수에 맞춰 샘플링된다.

## 가장 큰 단점은 정보 손실이다

언더샘플링은 빠르고 단순하지만 다수 클래스 데이터를 실제로 버린다.

```text
1000개 → 20개
```

가 된다면 980개의 정보가 사라진다.

특히 데이터가 충분하지 않은 상황에서는 손실이 매우 클 수 있다.

그래서 다음과 같은 경우에 더 신중해야 한다.

- 전체 데이터가 작을 때
- 다수 클래스 안에서도 다양한 패턴이 있을 때
- 중요한 드문 패턴이 다수 클래스 안에 섞여 있을 때

## 언제 유용할까?

- 데이터가 매우 클 때
- 학습 시간을 줄이고 싶을 때
- 다수 클래스가 과도하게 압도적일 때
- 빠른 베이스라인을 만들 때

## 언더샘플링과 오버샘플링 중 무엇이 더 좋은가?

정답은 없다.

언더샘플링은:

```text
빠름
↓
하지만 정보 손실
```

오버샘플링은:

```text
정보를 버리지 않음
↓
하지만 과적합 또는 인공 샘플 품질 문제
```

가 있다.

따라서 실전에서는 원본 모델, class_weight, 언더샘플링, 오버샘플링을 모두 비교해보는 것이 좋다.

## 정리

`RandomUnderSampler()`의 결과가 가장 적은 클래스 수에 맞춰진 것은 우연이 아니다.

기본 전략이 다수 클래스의 데이터를 줄여 클래스 간 균형을 맞추기 때문이다.

그리고 필요하다면:

```python
sampling_strategy={...}
```

를 통해 원하는 클래스별 샘플 수를 직접 지정할 수 있다.

다만 언더샘플링은 성능을 높여주는 마법의 기법이 아니라 **정보를 버리는 대가를 치르고 균형을 얻는 방법**이라는 점을 기억해야 한다.
