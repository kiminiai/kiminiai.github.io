---
title: "SMOTE의 k_neighbors는 왜 필요할까? 오버샘플링 원리부터 데이터 누수까지"
slug: "smote-k-neighbors-oversampling"
date: 2026-07-15
draft: false
categories:
  - Machine Learning
tags:
  - SMOTE
  - imbalanced-data
  - oversampling
  - imbalanced-learn
description: "SMOTE가 소수 클래스의 이웃을 이용해 새로운 샘플을 만드는 원리와 k_neighbors의 의미, 실무에서 주의할 점을 정리합니다."
---

불균형 데이터 실습에서 다음 코드를 봤다.

```python
from imblearn.over_sampling import SMOTE

sampler = SMOTE(k_neighbors=4)
X_res, y_res = sampler.fit_resample(X, y)
```

처음 든 질문은 단순했다.

> 왜 하필 `k_neighbors=4`일까?

하지만 이 값을 이해하려면 먼저 SMOTE가 무엇을 하는지부터 알아야 했다.

## SMOTE는 데이터를 복사하는 방법이 아니다

단순 오버샘플링은 소수 클래스의 기존 데이터를 복제할 수 있다.

```text
기존 소수 클래스 샘플
A, B, C

단순 복제
A, A, B, B, C, C
```

SMOTE는 다르다.

기존 소수 클래스 샘플과 그 주변의 소수 클래스 이웃 사이에서 **새로운 가상 샘플**을 만든다.

![SMOTE 직관](/img/data-analysis-study-notes/smote-intuition.png)

개념적으로는 다음과 같다.

```text
새 샘플 = 원본 샘플 + 랜덤 비율 × (이웃 샘플 - 원본 샘플)
```

따라서 완전히 동일한 데이터를 복제하는 것이 아니라, 기존 소수 클래스가 존재하는 공간 안에서 새로운 점을 생성한다.

## k_neighbors의 의미

```python
SMOTE(k_neighbors=4)
```

는 각 소수 클래스 샘플을 기준으로 **가장 가까운 소수 클래스 이웃 4개를 후보로 보겠다**는 뜻이다.

예를 들어 A 주변에 가장 가까운 소수 클래스 샘플이 다음 순서라면:

```text
B, C, D, E, F
```

`k_neighbors=4`일 때 후보는:

```text
B, C, D, E
```

중 하나가 된다.

이웃 하나를 선택한 뒤 A와 그 이웃 사이에서 새로운 점을 만든다.

## k가 너무 작거나 너무 크면?

### 너무 작을 때

```python
k_neighbors=1
```

가까운 한 점만 반복적으로 참고하게 된다.

데이터 다양성이 충분히 늘어나지 않을 수 있다.

### 너무 클 때

너무 먼 소수 클래스 점까지 이웃으로 포함할 수 있다.

특히 클래스 내부에 여러 하위 패턴이 있다면 서로 다른 영역을 억지로 연결해 이상한 샘플을 만들 위험이 있다.

그래서 실전에서는 보통 하나의 정답을 외우기보다 여러 값을 비교한다.

```python
k_neighbors = 3
k_neighbors = 5
k_neighbors = 7
```

그리고 교차검증으로 성능을 확인한다.

## 가장 중요한 실무 주의점: split 전에 하면 안 된다

다음 순서는 위험하다.

```python
X_res, y_res = smote.fit_resample(X, y)

X_train, X_test, y_train, y_test = train_test_split(
    X_res, y_res
)
```

SMOTE로 만든 데이터가 학습과 테스트 양쪽에 섞일 수 있기 때문이다.

더 안전한 순서는 다음과 같다.

```text
원본 데이터
↓
train / test 분리
↓
train에만 SMOTE
↓
모델 학습
↓
원본 test로 평가
```

즉:

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    stratify=y,
    random_state=42
)

X_train_res, y_train_res = smote.fit_resample(
    X_train,
    y_train
)
```

## Pipeline과 함께 쓰는 이유

교차검증을 할 때는 각 fold의 학습 데이터에만 SMOTE가 적용되어야 한다.

그래서 `imblearn.pipeline.Pipeline`을 사용하는 것이 안전하다.

```python
from imblearn.pipeline import Pipeline

pipeline = Pipeline([
    ('smote', SMOTE(k_neighbors=5)),
    ('model', classifier)
])
```

이렇게 하면 각 fold 안에서:

```text
학습 fold → SMOTE → 모델 학습
검증 fold → 원본 상태로 평가
```

가 유지된다.

## 정리

`k_neighbors`는 단순히 "몇 개를 늘릴지"가 아니다.

> 새로운 소수 클래스 샘플을 만들 때 주변의 몇 개 이웃을 후보로 볼 것인가?

를 정하는 값이다.

그리고 SMOTE에서 더 중요한 것은 `k=4`냐 `k=5`냐보다도 **오버샘플링을 반드시 학습 데이터 안에서만 수행하는 것**이다.

이 원칙을 놓치면 점수는 높아 보여도 실제 성능은 과대평가될 수 있다.
