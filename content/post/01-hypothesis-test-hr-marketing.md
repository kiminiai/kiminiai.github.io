---
title: "t-test부터 ANOVA, 카이제곱 검정까지: 가설검정을 실제 HR·마케팅 데이터에 적용해봤다"
date: 2026-07-11
slug: hypothesis-test-hr-marketing
categories:
  - Study
tags:
  - Statistics
  - T-test
  - ANOVA
  - Chi-Square
  - Python
draft: false
---

# 통계 검정은 결국 '차이가 우연인지' 확인하는 과정이었다

이번 수업에서는 t-test, ANOVA, 카이제곱 검정을 한 번에 실습했다.

이름만 보면 서로 완전히 다른 분석처럼 보였는데, 직접 문제를 풀고 나니 공통점이 있었다.

> 내가 데이터에서 발견한 차이나 관계가 우연히 나타난 것인지, 통계적으로 의미가 있는지를 확인하는 과정이다.

예전에는 p-value가 0.05보다 작으면 귀무가설을 기각한다고 외웠다. 그런데 실제 데이터를 가지고 문제를 풀어보니, 검정을 시작하기 전에 **무엇을 비교하는지**, **데이터가 서로 독립인지**, **변수가 수치형인지 범주형인지**를 먼저 판단해야 했다.

![가설검정 분석 흐름](/img/statistics-day/01-test-workflow.png)

## 어떤 검정을 선택해야 할까?

| 확인하려는 내용 | 사용한 검정 |
|---|---|
| 서로 다른 두 집단의 평균 비교 | 독립표본 t-검정 |
| 동일 대상을 전후로 측정한 평균 비교 | 대응표본 t-검정 |
| 세 집단 이상의 평균 비교 | One-Way ANOVA |
| 두 범주형 변수의 연관성 확인 | 카이제곱 독립성 검정 |

검정 이름을 외우기보다는 **데이터의 구조에 따라 선택한다**고 생각하는 편이 이해하기 쉬웠다.

# 1. 온라인과 오프라인 영업 사원의 월매출은 다른가?

온라인 채널 영업 사원 80명과 오프라인 채널 영업 사원 75명의 월매출을 비교했다.

두 집단은 서로 다른 사람들로 구성되어 있으므로 독립표본 t-검정을 사용했다.

- 귀무가설(H0): 두 채널의 평균 월매출은 같다.
- 대립가설(H1): 두 채널의 평균 월매출은 다르다.

```python
import pandas as pd
from scipy import stats

df = pd.read_csv("sales_1_ttest_indep.csv", encoding="utf-8-sig")

online = df.loc[df["channel"] == "온라인", "monthly_sales"]
offline = df.loc[df["channel"] == "오프라인", "monthly_sales"]

# 먼저 등분산 여부 확인
levene_stat, levene_p = stats.levene(online, offline)

# Levene 검정 결과에 따라 equal_var 결정
equal_var = levene_p >= 0.05
t_stat, p_value = stats.ttest_ind(
    online,
    offline,
    equal_var=equal_var
)
```

독립표본 t-검정 전에 Levene 검정을 한 이유는 두 집단의 분산을 같다고 가정해도 되는지 판단하기 위해서였다.

실습 결과는 다음과 같았다.

| 구분 | 결과 |
|---|---:|
| 온라인 평균 | 3,213.374만 원 |
| 오프라인 평균 | 2,849.041만 원 |
| t-통계량 | 4.786 |
| p-value | 0.000 |

![채널별 월매출 평균](/img/statistics-day/02-channel-sales.png)

p-value가 0.05보다 작으므로 귀무가설을 기각했다. 이번 표본에서는 온라인과 오프라인 채널의 월매출 평균에 통계적으로 유의한 차이가 있었다.

여기서 조심할 점은 **온라인 채널이 무조건 더 우수하다고 단정할 수는 없다는 것**이다. 채널 외에도 담당 지역, 상품, 경력, 고객 수와 같은 변수가 다를 수 있기 때문이다. t-test는 평균 차이를 확인해주지만 그 차이의 원인까지 설명해주지는 않는다.

# 2. 캠페인 전후 전환율은 실제로 높아졌을까?

이번에는 같은 60개 광고 소재의 캠페인 전후 전환율을 비교했다.

같은 광고 소재를 두 번 측정했기 때문에 두 데이터는 독립적이지 않다. 따라서 독립표본 t-검정이 아니라 대응표본 t-검정을 사용했다.

```python
pre = df["pre_cvr"]
post = df["post_cvr"]
diff = post - pre

t_stat, p_two_sided = stats.ttest_rel(post, pre)

# 대립가설이 '캠페인 후가 더 크다'인 단측검정
p_value = p_two_sided / 2 if t_stat > 0 else 1 - p_two_sided / 2
```

결과는 다음과 같았다.

| 구분 | 결과 |
|---|---:|
| 캠페인 전 평균 | 3.252% |
| 캠페인 후 평균 | 3.777% |
| 평균 증가폭 | 0.526%p |
| t-통계량 | 13.235 |
| p-value | 0.000 |

![캠페인 전후 전환율](/img/statistics-day/03-campaign-cvr.png)

귀무가설을 기각했기 때문에 캠페인 이후 전환율 향상은 통계적으로 유의했다.

여기서는 단순히 p-value만 보는 것보다 **평균 증가폭이 0.526%p였다는 사실**도 같이 봐야 한다. 통계적으로 유의하더라도 실제 사업에서 의미 있는 크기인지 별도로 판단해야 하기 때문이다.

# 3. 직군이 4개라면 t-test를 반복하면 될까?

IT직, 기획직, 마케팅직, 영업직 지원자의 토익 점수를 비교했다.

집단이 4개이므로 t-test를 여러 번 반복하는 대신 One-Way ANOVA를 사용했다.

- 귀무가설(H0): 모든 직군의 토익 평균은 같다.
- 대립가설(H1): 적어도 한 직군의 평균은 다르다.

```python
from scipy import stats

groups = df.groupby("job_type")["toeic_score"]
group_data = [group.values for _, group in groups]

f_stat, p_value = stats.f_oneway(*group_data)
```

| 직군 | 평균 토익 점수 |
|---|---:|
| IT직 | 854.800 |
| 기획직 | 794.875 |
| 마케팅직 | 820.975 |
| 영업직 | 782.925 |

F-통계량은 10.878, p-value는 0.000으로 귀무가설을 기각했다.

![직군별 토익 평균](/img/statistics-day/04-toeic-anova.png)

하지만 ANOVA는 **어느 집단과 어느 집단이 다른지까지 알려주지는 않는다.** 그래서 Tukey HSD 사후검정을 추가로 수행했다.

```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd

tukey = pairwise_tukeyhsd(
    endog=df["toeic_score"],
    groups=df["job_type"],
    alpha=0.05
)

print(tukey.summary())
```

유의한 차이가 확인된 조합은 다음과 같았다.

- IT직 vs 기획직
- IT직 vs 영업직
- 마케팅직 vs 영업직

반면 IT직과 마케팅직의 차이는 5% 유의수준에서 유의하지 않았다.

이 실습을 통해 ANOVA의 p-value가 작다고 해서 모든 집단이 서로 다르다는 뜻은 아니라는 것을 알게 됐다.

# 4. 채용 전형과 합격 여부는 연관성이 있을까?

채용 전형은 공개채용·추천채용·인턴전환이고, 결과는 합격·불합격이다.

두 변수 모두 범주형이므로 평균을 비교하는 t-test나 ANOVA를 사용할 수 없다. 이 경우 카이제곱 독립성 검정을 사용한다.

```python
import pandas as pd
from scipy.stats import chi2_contingency

cross_table = pd.crosstab(df["hire_type"], df["final_pass"])

chi2, p_value, dof, expected = chi2_contingency(cross_table)
```

실제 교차표는 다음과 같았다.

| 전형 | 불합격 | 합격 |
|---|---:|---:|
| 공개채용 | 121 | 43 |
| 인턴전환 | 13 | 44 |
| 추천채용 | 44 | 35 |

![채용 전형과 합격 여부](/img/statistics-day/05-hiring-chi-square.png)

검정 결과는 다음과 같았다.

- 카이제곱 통계량: 46.137
- 자유도: 2
- p-value: 0.000

귀무가설을 기각했기 때문에 채용 전형 유형과 최종 합격 여부 사이에는 통계적으로 유의한 연관성이 있었다.

다만 카이제곱 검정 역시 인과관계를 증명하지는 않는다. 인턴전환 지원자는 이미 회사가 일정 기간 관찰한 지원자일 수 있기 때문에 전형 자체 외에 지원자의 특성이 다를 가능성도 있다.

# 이번 실습에서 가장 중요했던 점

이번에는 검정 공식을 외우는 것보다 검정 선택 기준을 이해한 것이 가장 컸다.

1. 수치형 평균을 비교하는가, 범주형 관계를 확인하는가?
2. 비교 대상이 서로 다른 집단인가, 같은 대상을 전후로 측정했는가?
3. 집단이 두 개인가, 세 개 이상인가?
4. p-value가 작다는 것과 실무적으로 효과가 크다는 것은 같은 의미인가?

앞으로 분석할 때는 p-value만 출력하고 끝내지 않고, 평균 차이와 표본 구조, 효과의 크기, 인과관계 해석의 한계를 함께 확인해야겠다.
