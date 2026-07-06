---
title: "SQL 기초 3편: 집합 연산자, 뷰, 윈도우 함수와 마케팅 분석 실습"
date: 2026-07-06
draft: false
categories:
  - SQL
tags:
  - SQL
  - Window Function
  - RANK
  - LAG
  - SUM OVER
  - CTE
  - Marketing Analytics
---

# SQL 기초 3편: 집합 연산자, 뷰, 윈도우 함수와 마케팅 분석 실습

> 이번 글에서는 SQL 기초에서 한 단계 더 나아가 집합 연산자, 뷰, 윈도우 함수, CTE를 정리한다.  
> 특히 실습 SQL 파일에 포함된 마케팅 분석 문제를 기준으로, SQL이 단순 조회 문법이 아니라 분석 도구로 어떻게 쓰이는지 정리했다.

---

## 1. 집합 연산자

집합 연산자는 두 개 이상의 SELECT 결과를 하나로 결합하거나 비교할 때 사용한다.

대표적인 집합 연산자는 다음과 같다.

| 연산자 | 의미 |
|---|---|
| `UNION` | 합집합. 중복 제거 |
| `UNION ALL` | 합집합. 중복 유지 |
| `INTERSECT` | 교집합 |
| `EXCEPT` | 차집합 |
| `CROSS JOIN` | 가능한 모든 조합 |

![집합 연산자 개념](/img/sql-basics/set-operators.svg)

---

## 2. UNION과 UNION ALL

`UNION`은 두 SELECT 결과를 합친 뒤 중복을 제거한다.

```sql
SELECT customerNumber
FROM orders
WHERE strftime('%Y', orderDate) = '2003'

UNION

SELECT customerNumber
FROM orders
WHERE strftime('%Y', orderDate) = '2004';
```

반면 `UNION ALL`은 중복을 제거하지 않는다.

```sql
SELECT customerNumber
FROM orders
WHERE strftime('%Y', orderDate) = '2003'

UNION ALL

SELECT customerNumber
FROM orders
WHERE strftime('%Y', orderDate) = '2004';
```

중복 제거가 필요 없다면 `UNION ALL`이 더 빠를 수 있다.

---

## 3. INTERSECT와 EXCEPT

`INTERSECT`는 두 결과에 공통으로 존재하는 데이터를 조회한다.  
즉, 교집합이다.

`EXCEPT`는 앞쪽 SELECT 결과에서 뒤쪽 SELECT 결과를 제외한다.  
즉, 차집합이다.

다만 DBMS마다 지원 여부가 다를 수 있다.  
예를 들어 MySQL은 `INTERSECT`를 지원하지 않는 버전이 많고, Oracle에서는 `EXCEPT` 대신 `MINUS`를 사용하기도 한다.

---

## 4. 뷰(View)

뷰는 다른 테이블에서 파생된 논리적인 테이블이다.  
실제 데이터를 새로 저장한다기보다는, 자주 사용하는 SELECT 쿼리에 이름을 붙여두는 것에 가깝다.

```sql
CREATE VIEW customer_order_summary AS
SELECT c.customerNumber,
       c.customerName,
       COUNT(o.orderNumber) AS order_count
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName;
```

뷰의 장점은 다음과 같다.

| 장점 | 설명 |
|---|---|
| 독립성 | 테이블 구조가 바뀌어도 응용 프로그램 영향을 줄일 수 있다 |
| 편리성 | 자주 쓰는 복잡한 쿼리를 간단히 사용할 수 있다 |
| 보안성 | 사용자별로 보여줄 컬럼이나 행을 제한할 수 있다 |

다만 뷰를 통한 갱신에는 제약이 있고, 원천 테이블이 삭제되면 해당 테이블을 기반으로 하는 뷰도 영향을 받는다.

---

## 5. 윈도우 함수란?

윈도우 함수는 행과 행 사이의 관계를 계산하는 함수이다.  
`GROUP BY`처럼 데이터를 하나의 행으로 줄이지 않고, 원래 행을 유지한 상태에서 순위나 누적합, 이전 값 등을 계산할 수 있다.

윈도우 함수는 `OVER` 구문을 필수로 사용한다.

```sql
윈도우함수() OVER (
    PARTITION BY 그룹기준
    ORDER BY 정렬기준
)
```

예를 들어 국가별로 고객을 나누고, 각 국가 안에서 신용한도 순위를 매길 수 있다.

![윈도우 함수에서 PARTITION BY와 ORDER BY](/img/sql-basics/window-function-partition.svg)

---

## 6. RANK, DENSE_RANK, ROW_NUMBER

순위를 매기는 대표적인 윈도우 함수는 다음과 같다.

| 함수 | 특징 |
|---|---|
| `ROW_NUMBER()` | 같은 값이어도 고유한 순번 부여 |
| `RANK()` | 공동 순위가 있으면 다음 순위를 건너뜀 |
| `DENSE_RANK()` | 공동 순위가 있어도 다음 순위를 건너뛰지 않음 |

예를 들어 점수가 다음과 같다고 가정한다.

| 이름 | 점수 |
|---|---:|
| A | 100 |
| B | 90 |
| C | 90 |
| D | 80 |

`RANK()` 결과는 `1, 2, 2, 4`가 된다.  
`DENSE_RANK()` 결과는 `1, 2, 2, 3`이 된다.

---

## 7. 실습 예제: 고객별 총 결제액 순위 TOP 10

```sql
SELECT customerNumber,
       customerName,
       total_payment,
       payment_rank
FROM (
    SELECT c.customerNumber,
           c.customerName,
           ROUND(SUM(p.amount), 2) AS total_payment,
           RANK() OVER (ORDER BY SUM(p.amount) DESC) AS payment_rank
    FROM customers c
    JOIN payments p
        ON c.customerNumber = p.customerNumber
    GROUP BY c.customerNumber, c.customerName
)
WHERE payment_rank <= 10
ORDER BY payment_rank;
```

이 쿼리는 고객별 총 결제액을 계산한 뒤, 결제액 기준으로 순위를 매긴다.  
마케팅 관점에서는 VIP 고객 후보를 찾는 데 사용할 수 있다.

여기서 중요한 점은 `RANK() OVER (...)`이다.

```sql
RANK() OVER (ORDER BY SUM(p.amount) DESC)
```

전체 고객을 결제액이 큰 순서로 정렬하고, 그 순서에 따라 순위를 부여한다.

---

## 8. PARTITION BY: 그룹별 순위 매기기

`PARTITION BY`는 윈도우 함수에서 그룹을 나누는 역할을 한다.

```sql
SELECT country,
       customerName,
       creditLimit,
       country_rank
FROM (
    SELECT country,
           customerName,
           creditLimit,
           RANK() OVER (
               PARTITION BY country
               ORDER BY creditLimit DESC
           ) AS country_rank
    FROM customers
)
WHERE country_rank <= 3
ORDER BY country, country_rank;
```

이 쿼리는 국가별로 고객을 나누고, 각 국가 안에서 `creditLimit`이 높은 순서대로 순위를 매긴다.

즉, `PARTITION BY country`는 “국가별로 따로 계산하겠다”는 뜻이다.

---

## 9. LAG와 LEAD

`LAG`는 이전 행의 값을 가져온다.  
`LEAD`는 다음 행의 값을 가져온다.

분석에서는 특히 시간 흐름이 있는 데이터에서 자주 사용한다.

| 함수 | 활용 예시 |
|---|---|
| `LAG` | 전월 대비 매출 비교, 직전 주문과 비교 |
| `LEAD` | 다음 주문일 확인, 다음 상태 확인 |

---

## 10. 실습 예제: 월별 매출과 전월 대비 증감

```sql
SELECT year_month,
       monthly_revenue,
       prev_month_revenue,
       ROUND(monthly_revenue - prev_month_revenue, 2) AS mom_diff
FROM (
    SELECT strftime('%Y-%m', o.orderDate) AS year_month,
           ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS monthly_revenue,
           LAG(ROUND(SUM(od.quantityOrdered * od.priceEach), 2)) OVER (
               ORDER BY strftime('%Y-%m', o.orderDate)
           ) AS prev_month_revenue
    FROM orders o
    JOIN orderdetails od
        ON o.orderNumber = od.orderNumber
    GROUP BY strftime('%Y-%m', o.orderDate)
)
ORDER BY year_month;
```

이 쿼리는 월별 매출을 계산하고, `LAG`로 전월 매출을 가져온다.  
그 다음 현재 월 매출에서 전월 매출을 빼서 전월 대비 증감액을 계산한다.

```sql
monthly_revenue - prev_month_revenue
```

이런 방식은 마케팅 트렌드 분석에서 매우 자주 사용된다.

---

## 11. SUM() OVER()

일반적인 `SUM()`은 `GROUP BY`와 함께 사용하면 여러 행을 하나의 행으로 요약한다.

반면 `SUM() OVER()`는 원래 행을 유지하면서 전체 합계나 그룹별 합계를 계산할 수 있다.

제품 라인별 매출 비중을 구하는 예제이다.

```sql
SELECT productLine,
       line_revenue,
       ROUND(line_revenue / SUM(line_revenue) OVER (), 4) AS revenue_ratio
FROM (
    SELECT p.productLine,
           ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS line_revenue
    FROM products p
    JOIN orderdetails od
        ON p.productCode = od.productCode
    GROUP BY p.productLine
)
ORDER BY line_revenue DESC;
```

여기서 핵심은 다음 부분이다.

```sql
SUM(line_revenue) OVER ()
```

`OVER()` 안에 아무것도 없으므로 전체 행을 기준으로 합계를 계산한다.  
따라서 각 제품 라인의 매출을 전체 매출로 나누어 매출 비중을 구할 수 있다.

---

## 12. CTE란?

CTE는 `WITH` 구문으로 만드는 임시 결과 집합이다.  
복잡한 쿼리를 단계별로 나눠서 읽기 쉽게 만들 때 사용한다.

```sql
WITH line_sales AS (
    SELECT ...
),
country_stats AS (
    SELECT ...
)
SELECT ...
FROM country_stats;
```

CTE의 장점은 복잡한 분석 쿼리를 여러 단계로 쪼갤 수 있다는 것이다.

---

## 13. 실습 예제: 국가 × productLine 마케팅 대시보드

실습 파일의 마지막 문제는 여러 개념을 종합해서 사용하는 복합 쿼리였다.

요구사항은 다음과 같았다.

1. 국가·제품라인별 매출 집계
2. 국가 내 제품라인 매출 순위 계산
3. 국가 전체 매출 대비 비중 계산
4. 국가 평균 라인 매출 초과 여부 표시
5. 전체 국가 평균 매출보다 높은 국가만 필터링

쿼리는 다음과 같다.

```sql
WITH line_sales AS (
    SELECT c.country,
           p.productLine,
           ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS line_revenue
    FROM customers c
    JOIN orders o        ON c.customerNumber = o.customerNumber
    JOIN orderdetails od ON o.orderNumber = od.orderNumber
    JOIN products p      ON od.productCode = p.productCode
    GROUP BY c.country, p.productLine
),
country_stats AS (
    SELECT country,
           line_revenue,
           productLine,
           SUM(line_revenue) OVER (PARTITION BY country) AS country_total,
           AVG(line_revenue) OVER (PARTITION BY country) AS country_avg_line_rev,
           RANK() OVER (
               PARTITION BY country
               ORDER BY line_revenue DESC
           ) AS country_line_rank
    FROM line_sales
),
country_filter AS (
    SELECT country
    FROM (
        SELECT country,
               SUM(line_revenue) AS country_total
        FROM line_sales
        GROUP BY country
    )
    WHERE country_total > (
        SELECT AVG(cntry_total)
        FROM (
            SELECT SUM(line_revenue) AS cntry_total
            FROM line_sales
            GROUP BY country
        )
    )
)
SELECT cs.country,
       cs.productLine,
       cs.line_revenue,
       cs.country_line_rank,
       ROUND(cs.line_revenue / cs.country_total, 4) AS pct_of_country,
       CASE
           WHEN cs.line_revenue > cs.country_avg_line_rev THEN 'Y'
           ELSE 'N'
       END AS above_avg_flag
FROM country_stats cs
WHERE cs.country IN (SELECT country FROM country_filter)
  AND cs.country_line_rank <= 3
ORDER BY cs.country, cs.country_line_rank;
```

이 쿼리는 처음 보면 길고 복잡하다.  
하지만 CTE 단위로 나누어 보면 이해하기 쉽다.

---

## 14. 복합 쿼리 해석

### 14-1. `line_sales`

```sql
WITH line_sales AS (
    SELECT c.country,
           p.productLine,
           ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS line_revenue
    FROM customers c
    JOIN orders o        ON c.customerNumber = o.customerNumber
    JOIN orderdetails od ON o.orderNumber = od.orderNumber
    JOIN products p      ON od.productCode = p.productCode
    GROUP BY c.country, p.productLine
)
```

이 단계에서는 국가와 제품 라인별 매출을 계산한다.

즉, 다음과 같은 결과를 만든다.

| country | productLine | line_revenue |
|---|---|---:|
| USA | Classic Cars | ... |
| USA | Motorcycles | ... |
| France | Classic Cars | ... |

---

### 14-2. `country_stats`

```sql
country_stats AS (
    SELECT country,
           line_revenue,
           productLine,
           SUM(line_revenue) OVER (PARTITION BY country) AS country_total,
           AVG(line_revenue) OVER (PARTITION BY country) AS country_avg_line_rev,
           RANK() OVER (
               PARTITION BY country
               ORDER BY line_revenue DESC
           ) AS country_line_rank
    FROM line_sales
)
```

이 단계에서는 국가별 통계를 계산한다.

- `country_total`: 국가 전체 매출
- `country_avg_line_rev`: 해당 국가의 제품라인 평균 매출
- `country_line_rank`: 국가 안에서 제품라인 매출 순위

여기서 윈도우 함수가 핵심이다.  
`PARTITION BY country`를 사용했기 때문에 국가별로 따로 합계, 평균, 순위를 계산한다.

---

### 14-3. `country_filter`

```sql
country_filter AS (
    SELECT country
    FROM (
        SELECT country,
               SUM(line_revenue) AS country_total
        FROM line_sales
        GROUP BY country
    )
    WHERE country_total > (
        SELECT AVG(cntry_total)
        FROM (
            SELECT SUM(line_revenue) AS cntry_total
            FROM line_sales
            GROUP BY country
        )
    )
)
```

이 단계에서는 전체 국가 평균 매출보다 높은 국가만 남긴다.  
즉, 마케팅 집중 국가를 필터링하는 역할을 한다.

---

### 14-4. 최종 SELECT

```sql
SELECT cs.country,
       cs.productLine,
       cs.line_revenue,
       cs.country_line_rank,
       ROUND(cs.line_revenue / cs.country_total, 4) AS pct_of_country,
       CASE
           WHEN cs.line_revenue > cs.country_avg_line_rev THEN 'Y'
           ELSE 'N'
       END AS above_avg_flag
FROM country_stats cs
WHERE cs.country IN (SELECT country FROM country_filter)
  AND cs.country_line_rank <= 3
ORDER BY cs.country, cs.country_line_rank;
```

최종적으로는 마케팅 집중 국가 중에서 제품 라인 매출 순위가 3위 이내인 데이터만 조회한다.

이 쿼리는 SQL을 단순 조회가 아니라 분석 도구로 사용하는 좋은 예시이다.

---

## 15. 이번 글 정리

이번 글에서는 SQL 심화 개념을 정리했다.

| 개념 | 핵심 |
|---|---|
| `UNION` | 두 SELECT 결과를 합치고 중복 제거 |
| `UNION ALL` | 두 SELECT 결과를 합치고 중복 유지 |
| `INTERSECT` | 두 결과의 교집합 |
| `EXCEPT` | 앞 결과에서 뒤 결과를 제외 |
| View | 자주 사용하는 쿼리에 이름을 붙인 논리적 테이블 |
| Window Function | 행을 유지한 상태에서 순위, 이전 값, 전체 합계 계산 |
| `RANK()` | 순위 계산 |
| `LAG()` | 이전 행 값 가져오기 |
| `SUM() OVER()` | 행을 유지하면서 전체 또는 그룹 합계 계산 |
| CTE | 복잡한 쿼리를 단계별로 나누는 WITH 구문 |

SQL을 처음 공부할 때는 문법이 많아 보여서 어렵게 느껴진다.  
하지만 분석 관점에서 보면 흐름은 단순하다.

```text
필요한 테이블을 연결하고
→ 조건에 맞게 필터링하고
→ 기준별로 집계하고
→ 순위나 비중을 계산하고
→ 분석 목적에 맞게 해석한다.
```

이 흐름을 잡으면 JOIN, 서브쿼리, 윈도우 함수도 단순한 문법 암기가 아니라 분석을 위한 도구로 이해할 수 있다.
