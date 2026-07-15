---
title: "axis, interpolate, np.where를 데이터 전처리 관점에서 이해하기"
slug: "pandas-axis-interpolate-numpy-where"
date: 2026-07-15
draft: false
categories:
  - Python
  - Pandas
tags:
  - axis
  - interpolate
  - np.where
  - preprocessing
description: "데이터 전처리에서 자주 만나는 axis, interpolate, np.where의 의미를 실제 코드 중심으로 정리합니다."
---

데이터 전처리 실습에서 자주 보이지만 매번 다시 검색하게 되는 개념이 있다.

```python
axis
interpolate()
np.where()
```

셋은 서로 다른 기능이지만 실제 전처리 코드에서 매우 자주 등장한다.

![axis interpolate where](/img/data-analysis-study-notes/axis-interpolate-where.png)

## axis: 어느 축을 기준으로 작업할 것인가?

예를 들어:

```python
df.drop('id', axis='columns')
```

은 `id`라는 열을 제거한다.

같은 의미로:

```python
df.drop('id', axis=1)
```

이라고 쓸 수 있다.

반대로 행을 제거할 때는:

```python
df.drop(index=3)
```

또는 상황에 따라 `axis=0`을 사용한다.

기본적으로:

```text
axis=0 또는 axis='index'
→ 행 축을 따라 연산

axis=1 또는 axis='columns'
→ 열 축을 따라 연산
```

다만 "행 방향", "열 방향"이라는 말은 처음에 오히려 더 헷갈릴 수 있다.

실전에서는 이렇게 외우는 편이 더 편했다.

```text
axis=0
→ 각 열별 결과가 나오는 경우가 많음

axis=1
→ 각 행별 결과가 나오는 경우가 많음
```

예를 들어:

```python
df.mean(axis=0)
```

은 각 열의 평균을 구한다.

```python
df.mean(axis=1)
```

은 각 행의 평균을 구한다.

## interpolate: 중간 결측값을 추정한다

```python
df['Age'] = df['Age'].interpolate()
```

기본 선형 보간의 직관은 다음과 같다.

```text
20, NaN, 40
```

이라면 중간을:

```text
30
```

으로 채운다.

이 방법은 다음처럼 값의 흐름이 연속적일 때 더 자연스럽다.

- 시간
- 온도
- 거리
- 센서값
- 시계열 관측값

반대로 범주형 데이터에는 일반적으로 적합하지 않다.

```text
서울, NaN, 부산
```

의 중간값은 정의하기 어렵기 때문이다.

## np.where: 조건에 따라 값을 선택한다

형식은 다음과 같다.

```python
np.where(조건, 참일 때 값, 거짓일 때 값)
```

예를 들어:

```python
df['adult'] = np.where(
    df['age'] >= 20,
    1,
    0
)
```

20세 이상이면 1, 아니면 0을 넣는다.

반복문 없이 전체 배열이나 Series에 조건을 적용할 수 있다는 점이 핵심이다.

## boolean indexing과 무엇이 다른가?

```python
array[array % 3 == 0]
```

은 조건을 만족하는 값만 골라낸다.

반면:

```python
np.where(array % 3 == 0, 'multiple', 'other')
```

는 모든 위치를 유지한 채 조건에 따라 값을 바꾼다.

즉:

```text
boolean indexing
→ 필터링

np.where
→ 조건부 변환
```

으로 이해하면 된다.

## 정리

### axis

```text
연산 또는 삭제의 기준 축
```

### interpolate

```text
연속적인 값의 중간 결측치를 추정
```

### np.where

```text
조건에 따라 값을 벡터화해서 선택
```

전처리 코드는 결국:

```text
무엇을 선택할지
어느 방향으로 계산할지
결측값을 어떻게 처리할지
조건에 따라 어떻게 변환할지
```

의 반복이다.

이 세 개념을 이해하면 pandas와 NumPy 전처리 코드를 읽는 속도가 훨씬 빨라진다.
