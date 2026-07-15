---
title: "fit_transform(X[['Fare']])에서 대괄호가 두 번 들어가는 이유"
slug: "fit-transform-double-brackets-dataframe-series"
date: 2026-07-15
draft: false
categories:
  - Study
tags:
  - pandas
  - scikit-learn
  - DataFrame
  - Series
  - MinMaxScaler
description: "X['Fare']와 X[['Fare']]의 차이부터 scikit-learn이 왜 2차원 입력을 요구하는지까지 정리합니다."
keywords:
  - fit_transform 대괄호 두번
  - X 이중 대괄호
  - DataFrame Series 차이
  - MinMaxScaler 2차원
  - sklearn expected 2D array
---

머신러닝 실습 중 다음 코드를 보고 한동안 멈췄다.

```python
scaler = MinMaxScaler()
X['Fare'] = scaler.fit_transform(X[['Fare']])
```

`Fare`라는 열 하나만 선택하는데 왜 `X['Fare']`가 아니라 `X[['Fare']]`처럼 대괄호를 두 번 쓰는 걸까?

처음에는 단순한 문법 차이라고 생각했지만, 실제로는 **pandas의 Series와 DataFrame 차이**, 그리고 **scikit-learn이 데이터를 바라보는 방식**이 연결되어 있었다.

---

## 핵심부터 말하면

```python
X['Fare']
```

는 **1차원 Series**를 반환하고,

```python
X[['Fare']]
```

는 **2차원 DataFrame**을 반환한다.

scikit-learn의 `MinMaxScaler`, `StandardScaler` 같은 변환기는 입력 데이터를 기본적으로 다음과 같은 2차원 형태로 기대한다.

```text
(샘플 수, 특성 수)
```

즉, 열이 하나뿐이어도 다음처럼 표현되어야 한다.

```text
(891, 1)
```

그래서 `fit_transform()`에 열 하나를 넣을 때도 DataFrame 형태를 유지하기 위해 대괄호를 두 번 사용한다.

![Series와 DataFrame 차이](/img/study-note-01/series-vs-dataframe.png)

---

## X['Fare']와 X[['Fare']]는 실제로 무엇이 다른가?

예를 들어 다음과 같은 데이터프레임이 있다고 하자.

```python
import pandas as pd

X = pd.DataFrame({
    'Age': [22, 38, 26],
    'Fare': [7.25, 71.28, 7.92]
})
```

### 대괄호 한 번

```python
fare_series = X['Fare']

print(type(fare_series))
print(fare_series.shape)
```

결과는 다음과 같다.

```text
<class 'pandas.core.series.Series'>
(3,)
```

`(3,)`은 데이터가 3개 있는 **1차원 구조**라는 뜻이다.

---

### 대괄호 두 번

```python
fare_df = X[['Fare']]

print(type(fare_df))
print(fare_df.shape)
```

결과는 다음과 같다.

```text
<class 'pandas.core.frame.DataFrame'>
(3, 1)
```

이번에는 3개의 행과 1개의 열을 가진 **2차원 구조**다.

같은 `Fare` 열을 선택했지만 반환되는 객체의 차원이 다르다.

| 코드 | 반환 타입 | shape |
|---|---|---|
| `X['Fare']` | Series | `(3,)` |
| `X[['Fare']]` | DataFrame | `(3, 1)` |

---

## 왜 scikit-learn은 굳이 2차원을 요구할까?

머신러닝 모델에서 입력 데이터 `X`는 일반적으로 **특성 행렬(feature matrix)**이다.

예를 들어 승객 3명의 `Age`, `Fare`를 사용한다면 데이터는 다음처럼 생각할 수 있다.

```text
        Age    Fare
승객1    22     7.25
승객2    38    71.28
승객3    26     7.92
```

행은 샘플이고 열은 특성이다.

따라서 입력 구조는 항상 다음과 같이 표현된다.

```text
(n_samples, n_features)
```

특성이 2개라면:

```text
(3, 2)
```

특성이 1개라면:

```text
(3, 1)
```

scikit-learn은 열이 하나라고 해서 갑자기 입력 구조를 1차원으로 바꾸지 않는다.  
여러 개의 특성을 받는 동일한 인터페이스를 유지하기 위해 **특성 하나도 2차원 행렬로 전달하는 것**이 기본이다.

---

## 실제로 Series를 넣으면 어떤 일이 생길까?

다음 코드는 문제가 발생할 수 있다.

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
scaler.fit_transform(X['Fare'])
```

대표적으로 다음과 비슷한 오류를 볼 수 있다.

```text
Expected 2D array, got 1D array instead
```

scikit-learn 입장에서는 `(3,)`만 보고 이것이 다음 중 무엇인지 확실히 구분하기 어렵다.

- 샘플 3개, 특성 1개
- 샘플 1개, 특성 3개

따라서 입력을 명확한 2차원 구조로 전달해야 한다.

```python
scaler.fit_transform(X[['Fare']])
```

이제 shape은 `(3, 1)`이다.

- 샘플: 3개
- 특성: 1개

구조가 명확하다.

---

## fit_transform은 무엇을 하는가?

`fit_transform()`은 이름 그대로 두 단계를 한 번에 수행한다.

```mermaid
flowchart LR
    A["X[['Fare']]"] --> B["fit()"]
    B --> C["최솟값·최댓값 학습"]
    C --> D["transform()"]
    D --> E["스케일링된 2차원 배열"]
```

`MinMaxScaler`를 예로 들면 `fit()` 단계에서 학습 데이터의 최솟값과 최댓값을 기억한다.

그 다음 `transform()` 단계에서 다음 방식으로 데이터를 변환한다.

```text
x_scaled = (x - min) / (max - min)
```

따라서 다음 코드는:

```python
scaler.fit_transform(X[['Fare']])
```

개념적으로 다음과 같다.

```python
scaler.fit(X[['Fare']])
scaled_fare = scaler.transform(X[['Fare']])
```

![fit_transform 흐름](/img/study-note-01/fit-transform-flow.png)

---

## 그런데 왼쪽에는 왜 X['Fare']를 쓰는가?

여기서 또 하나 헷갈리는 부분이 있다.

```python
X['Fare'] = scaler.fit_transform(X[['Fare']])
```

오른쪽은 `X[['Fare']]`인데 왼쪽은 `X['Fare']`다.

이유는 역할이 다르기 때문이다.

오른쪽의 `X[['Fare']]`는 **scikit-learn에 넣는 입력 데이터**다.  
따라서 2차원 구조가 필요하다.

반면 왼쪽의 `X['Fare']`는 **DataFrame의 한 열에 결과를 저장하는 대상**이다.

즉 다음처럼 이해하면 된다.

```text
X[['Fare']]
→ 변환기에 넣기 위한 2차원 입력

X['Fare']
→ 변환 결과를 저장할 단일 열
```

---

## 더 깔끔한 방법은 없는가?

열 하나만 변환할 때는 다음처럼 작성할 수도 있다.

```python
X[['Fare']] = scaler.fit_transform(X[['Fare']])
```

입력과 출력 모두 DataFrame의 2차원 형태를 유지하므로 의미가 더 명확하다.

여러 열을 동시에 스케일링할 수도 있다.

```python
cols = ['Age', 'Fare']

X[cols] = scaler.fit_transform(X[cols])
```

이 경우 입력 shape은 다음과 같다.

```text
(n_samples, 2)
```

실무에서는 전처리할 열이 많아지면 `ColumnTransformer`와 `Pipeline`을 사용하는 경우가 많다.

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import MinMaxScaler

preprocessor = ColumnTransformer(
    transformers=[
        ('scale', MinMaxScaler(), ['Fare'])
    ],
    remainder='passthrough'
)
```

이 방식은 어떤 열에 어떤 전처리를 적용했는지 관리하기 쉽고, 학습 데이터와 테스트 데이터의 전처리 규칙을 일관되게 유지하는 데 유리하다.

---

## 내가 처음 헷갈렸던 이유

처음에는 `[['Fare']]`를 보고 단순히 "대괄호를 두 번 써야 하는 문법"이라고 생각했다.

하지만 실제로 중요한 것은 대괄호의 개수가 아니라 **반환되는 자료구조와 차원**이었다.

```python
X['Fare']
```

는 열 하나를 Series로 꺼낸다.

```python
X[['Fare']]
```

는 열 목록을 전달한다.

여기서 내부의 `['Fare']`는 "선택할 열의 목록"이다.

즉 다음과 같은 코드도 같은 원리다.

```python
X[['Age', 'Fare']]
```

열 두 개를 리스트로 전달하고 DataFrame을 반환한다.

열이 하나여도 리스트로 전달하면 DataFrame이 유지된다.

```python
X[['Fare']]
```

따라서 "대괄호를 두 번 쓴다"기보다 더 정확하게는:

> DataFrame에 열 이름 문자열 하나를 전달하면 Series가 나오고, 열 이름 리스트를 전달하면 DataFrame이 나온다.

라고 이해하는 것이 좋다.

---

## 실무에서 주의할 점

### 1. 스케일러는 학습 데이터에만 fit 해야 한다

다음처럼 전체 데이터에 먼저 `fit_transform()`을 적용한 뒤 학습/테스트 데이터를 나누면 데이터 누수가 발생할 수 있다.

```python
X['Fare'] = scaler.fit_transform(X[['Fare']])

X_train, X_test = train_test_split(X)
```

더 안전한 흐름은 다음과 같다.

```python
X_train, X_test = train_test_split(X)

scaler.fit(X_train[['Fare']])

X_train[['Fare']] = scaler.transform(X_train[['Fare']])
X_test[['Fare']] = scaler.transform(X_test[['Fare']])
```

테스트 데이터의 정보를 학습 과정에 사용하지 않기 위해서다.

---

### 2. 가능하면 Pipeline을 사용한다

전처리와 모델 학습을 따로 관리하면 실수하기 쉽다.

```python
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('scaler', MinMaxScaler()),
    ('model', DecisionTreeClassifier())
])
```

물론 실제로는 숫자형·범주형 열을 분리하기 위해 `ColumnTransformer`를 함께 사용하는 경우가 많다.

---

## 면접에서 설명한다면

**Q. `X['Fare']`와 `X[['Fare']]`의 차이는 무엇인가요?**

`X['Fare']`는 pandas Series를 반환하기 때문에 1차원이고, `X[['Fare']]`는 DataFrame을 반환하기 때문에 2차원입니다. scikit-learn의 transformer는 일반적으로 `(n_samples, n_features)` 형태의 2차원 입력을 기대하므로, 열 하나를 넣더라도 DataFrame 형태를 유지하기 위해 이중 대괄호를 사용합니다.

---

## 정리

```python
X['Fare']
```

- Series
- 1차원
- shape: `(n_samples,)`

```python
X[['Fare']]
```

- DataFrame
- 2차원
- shape: `(n_samples, 1)`

그리고 scikit-learn은 기본적으로 다음 구조를 기대한다.

```text
(샘플 수, 특성 수)
```

그래서 다음 코드에서:

```python
X['Fare'] = scaler.fit_transform(X[['Fare']])
```

오른쪽의 이중 대괄호는 단순한 문법 습관이 아니라 **하나의 특성을 2차원 특성 행렬로 유지하기 위한 선택**이다.

처음에는 대괄호 하나 차이처럼 보였지만, 이 차이를 이해하고 나니 `reshape(-1, 1)`, `X.shape`, Pipeline에서 발생하는 여러 오류도 훨씬 쉽게 이해할 수 있었다.
