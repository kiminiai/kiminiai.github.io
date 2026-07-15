---
title: "정규성을 만족하지 않으면 어떻게 할까? Anderson-Darling과 Wilcoxon 검정 실습"
date: 2026-07-11
slug: normality-and-nonparametric-test
categories:
  - Study
tags:
  - Statistics
  - Normality Test
  - Anderson Darling
  - Wilcoxon
  - Paired T-test
draft: false
---

# 무조건 t-test를 사용하면 안 되는 이유

처음 통계 문제를 풀 때는 평균을 비교하면 t-test를 사용하면 된다고 생각했다.

그런데 t-test 같은 모수 검정에는 데이터 분포와 관련된 가정이 있다. 이 가정이 크게 어긋난다면 비모수 검정을 고려해야 한다.

이번 실습에서는 PM2.5 데이터의 정규성을 확인하고, 생산성 전후 비교에는 Wilcoxon 부호 순위 검정을 사용했다.

![정규성에 따른 검정 선택](/img/statistics-day/06-normality-workflow.png)

# 1. PM2.5 일평균 데이터의 정규성 확인

원본 데이터는 시간 단위 PM2.5 측정값이었다. 문제에서는 100일 동안의 일평균 농도를 분석해야 했기 때문에 먼저 날짜별 평균을 계산했다.

```python
import pandas as pd
from statsmodels.stats.diagnostic import normal_ad

df = pd.read_csv("6_3_1.csv")

df["pm25"] = pd.to_numeric(df["pm25"], errors="coerce")
df["date"] = pd.to_datetime(df["date"], errors="coerce")
df["date_only"] = df["date"].dt.date

daily = (
    df.groupby("date_only", as_index=False)["pm25"]
      .mean()
      .dropna()
      .sort_values("date_only")
      .reset_index(drop=True)
)

start_date = pd.to_datetime(f"{int(df['year'].min())}-03-02")
end_date = start_date + pd.Timedelta(days=99)

dates = pd.to_datetime(daily["date_only"])
daily_100 = daily.loc[dates.between(start_date, end_date)].iloc[:100]

pm25 = daily_100["pm25"].astype(float)
```

여기서 바로 검정을 하지 않고 다음 내용을 확인했다.

- 실제로 100일이 선택되었는가?
- PM2.5가 숫자형으로 변환되었는가?
- 날짜 변환 실패나 결측치가 없는가?
- 시간별 데이터가 아니라 일평균 데이터가 맞는가?

통계 검정 이전에 분석 단위를 정확하게 만드는 전처리가 더 중요할 수 있다는 생각이 들었다.

## Anderson-Darling 검정

귀무가설과 대립가설은 다음과 같다.

- H0: PM2.5 일평균 데이터는 정규분포를 따른다.
- H1: PM2.5 일평균 데이터는 정규분포를 따르지 않는다.

```python
ad_stat, p_value = normal_ad(pm25.values)

if p_value < 0.05:
    decision = "귀무가설 기각"
else:
    decision = "귀무가설을 기각하지 못함"
```

실습에서는 100일의 데이터를 사용했고 표본 표준편차는 약 58.355였다. 검정 결과 p-value가 0.05보다 작아 정규성 귀무가설을 기각했다.

즉, 이 PM2.5 일평균 데이터는 정규분포를 따른다고 보기 어려웠다.

Anderson-Darling 검정은 분포의 꼬리 부분 차이에 비교적 민감하다. 미세먼지 데이터처럼 특정 시기에 매우 높은 값이 나타날 수 있는 환경 데이터에서는 이러한 특성이 중요할 수 있다.

# 2. 같은 학생의 1학기와 2학기 점수 비교

649명의 학생을 대상으로 새로운 학습 프로그램 도입 전후 점수를 비교했다.

같은 학생의 G1과 G2를 비교하기 때문에 대응표본 t-검정을 사용했다.

```python
from scipy import stats

diff = df["G2"] - df["G1"]

diff_mean = diff.mean()
diff_std = diff.std(ddof=1)

t_stat, p_value = stats.ttest_rel(
    df["G2"],
    df["G1"],
    alternative="greater"
)
```

결과는 다음과 같았다.

| 항목 | 결과 |
|---|---:|
| G2 - G1 평균 | 0.171 |
| 차이의 표준편차 | 1.479 |
| t-통계량 | 2.945 |
| 결론 | 귀무가설 기각 |

통계적으로는 프로그램 도입 후 점수가 높아졌다고 판단했다.

하지만 평균 상승 폭은 0.171점이었다. 표본 수가 649명으로 크기 때문에 작은 차이도 통계적으로 유의하게 나타날 수 있다.

따라서 **통계적 유의성과 실제 교육 효과의 크기는 따로 판단해야 한다.**

![학습 프로그램 점수 변화](/img/statistics-day/08-paired-score.png)

# 3. 정규성을 가정하기 어려운 전후 비교: Wilcoxon 검정

의류 제조 공장의 30개 팀을 대상으로 인센티브 프로그램 도입 전후 생산성을 비교했다.

이번에는 Wilcoxon 부호 순위 검정을 사용했다. 이 검정은 대응표본 t-검정의 비모수 대안으로 볼 수 있다.

```python
from scipy import stats

w_stat, p_value = stats.wilcoxon(
    df["after"],
    df["before"],
    alternative="greater"
)
```

| 구분 | 평균 | 표준편차 |
|---|---:|---:|
| 도입 전 | 0.73 | 0.08 |
| 도입 후 | 0.79 | 0.09 |

- W-통계량: 465.0
- p-value: 0.000
- 결론: 귀무가설 기각

![인센티브 프로그램 전후 생산성](/img/statistics-day/07-productivity-before-after.png)

인센티브 프로그램 도입 후 평균 생산성이 0.73에서 0.79로 증가했고, Wilcoxon 검정에서도 통계적으로 유의한 차이가 확인됐다.

Wilcoxon 검정은 단순히 평균 차이를 계산하는 방식이 아니라 전후 차이의 부호와 순위를 이용한다. 이상치나 비정규성의 영향을 줄일 수 있다는 점이 장점이다.

# 정규성 검정 결과만으로 자동 결정하면 될까?

정규성 검정의 p-value가 0.05보다 작다고 해서 무조건 비모수 검정을 써야 하는 것은 아니다.

표본 수가 많으면 아주 작은 분포 차이도 정규성 검정에서 유의하게 나타날 수 있다. 반대로 표본 수가 작으면 비정규성을 잘 발견하지 못할 수도 있다.

따라서 다음 내용을 함께 보는 것이 좋다.

```python
import matplotlib.pyplot as plt
import statsmodels.api as sm

fig, axes = plt.subplots(1, 2, figsize=(10, 4))

axes[0].hist(pm25, bins=15)
axes[0].set_title("Histogram")

sm.qqplot(pm25, line="45", ax=axes[1])
axes[1].set_title("Q-Q Plot")

plt.tight_layout()
plt.show()
```

- 히스토그램
- Q-Q Plot
- 이상치
- 표본 수
- 분석 목적
- 모수 검정의 강건성

# 이번 실습에서 알게 된 점

예전에는 p-value에 따라 기계적으로 검정을 선택하면 된다고 생각했다.

이번 실습을 통해서는 검정 전에 데이터가 어떻게 만들어졌는지부터 확인해야 한다는 것을 알게 됐다.

특히 시간별 PM2.5를 일평균으로 집계한 것처럼, **분석 단위를 어떻게 설정했는지에 따라 검정 대상 자체가 달라질 수 있다.**

앞으로는 다음 순서로 접근하려고 한다.

1. 분석 단위와 비교 구조를 확인한다.
2. 결측치와 이상치를 확인한다.
3. 분포를 시각적으로 확인한다.
4. 검정 가정을 확인한다.
5. 모수 검정 또는 비모수 검정을 선택한다.
6. p-value와 효과의 크기를 함께 해석한다.
