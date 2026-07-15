---
title: "enumerate, zip, idxmax가 헷갈릴 때 한 번에 정리하기"
slug: "python-enumerate-zip-pandas-idxmax"
date: 2026-07-15
draft: false
categories:
  - Python
  - Pandas
tags:
  - enumerate
  - zip
  - idxmax
  - pandas
description: "데이터 전처리 코드에서 자주 만나는 enumerate, zip, idxmax의 역할을 실제 예제로 비교합니다."
---

데이터 전처리 실습을 하다 보면 코드 한 줄은 짧은데 역할이 헷갈리는 함수들이 있다.

내가 자주 멈췄던 것은 다음 세 가지였다.

```python
enumerate()
zip()
idxmax()
```

셋 다 반복과 인덱스에 관련되어 보이지만 역할은 완전히 다르다.

![enumerate zip idxmax](/img/data-analysis-study-notes/enumerate-zip-idxmax.png)

## enumerate: 인덱스와 값을 같이 꺼낸다

기본 반복문은 값만 준다.

```python
nums = [10, 20, 30]

for value in nums:
    print(value)
```

결과:

```text
10
20
30
```

인덱스도 필요하다면:

```python
for i, value in enumerate(nums):
    print(i, value)
```

결과:

```text
0 10
1 20
2 30
```

즉:

```text
enumerate(iterable)
→ (index, value)
```

를 하나씩 준다.

## zip: 같은 위치의 값들을 묶는다

```python
names = ['A', 'B', 'C']
scores = [90, 80, 70]

for name, score in zip(names, scores):
    print(name, score)
```

결과:

```text
A 90
B 80
C 70
```

`zip()`은 여러 iterable을 같은 위치끼리 묶는다.

택시 데이터에서는 다음처럼 여러 좌표를 동시에 비교할 수 있다.

```python
zipped = zip(
    pickup_longitude,
    pickup_latitude,
    dropoff_longitude,
    dropoff_latitude
)
```

그리고:

```python
for x, y, dx, dy in zipped:
    if x == dx and y == dy:
        ...
```

처럼 사용할 수 있다.

## idxmax: 가장 큰 값의 index label을 반환한다

다음 DataFrame이 있다고 하자.

```python
df = pd.DataFrame({
    'name': ['Kim', 'Lee', 'Park'],
    'age': [25, 41, 33]
})
```

가장 나이가 많은 사람을 찾고 싶다면:

```python
index = df['age'].idxmax()
```

결과는:

```text
1
```

이다.

중요한 점은 `41`을 반환하는 것이 아니라 **41이 있는 인덱스**를 반환한다는 것이다.

그 다음:

```python
name = df.loc[index, 'name']
```

으로 실제 이름을 가져올 수 있다.

## argmax와 idxmax의 차이

NumPy에서:

```python
arr.argmax()
```

는 가장 큰 값의 **위치 번호**를 반환한다.

Pandas에서:

```python
series.idxmax()
```

는 가장 큰 값의 **index label**을 반환한다.

인덱스가 다음과 같다면:

```text
['a', 'b', 'c']
```

`idxmax()`는 `b` 같은 라벨을 반환할 수 있다.

## 한 줄 정리

```text
enumerate
→ 몇 번째인지 + 값

zip
→ 여러 데이터의 같은 위치 묶기

idxmax
→ 최댓값이 있는 index label
```

이 세 가지를 구분하고 나니 반복문과 DataFrame 인덱싱 코드를 읽는 속도가 훨씬 빨라졌다.
