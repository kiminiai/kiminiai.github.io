---
title: "SQL 기초 2편: JOIN과 서브쿼리로 여러 테이블 연결하기"
date: 2026-07-06
draft: false
categories:
  - Study
tags:
  - SQL
  - JOIN
  - INNER JOIN
  - LEFT JOIN
  - Subquery
  - EXISTS
---

# SQL 기초 2편: JOIN과 서브쿼리로 여러 테이블 연결하기

> 1편에서는 단일 테이블에서 데이터를 조회하고 집계하는 방법을 정리했다.  
> 이번 글에서는 여러 테이블을 연결하는 `JOIN`과 쿼리 안에 또 다른 쿼리를 넣는 `서브쿼리`를 정리한다.

---

## 1. JOIN이 필요한 이유

실무 데이터는 하나의 테이블에 모든 정보가 들어 있지 않다.

예를 들어 고객 정보는 `customers` 테이블에 있고, 주문 정보는 `orders` 테이블에 있고, 주문 상세 정보는 `orderdetails` 테이블에 있을 수 있다.

고객별 주문 금액을 구하려면 이 테이블들을 연결해야 한다.  
이때 사용하는 것이 `JOIN`이다.

![JOIN으로 테이블 연결하기](/img/sql-basics/relational-db-structure.svg)

---

## 2. INNER JOIN

`INNER JOIN`은 두 테이블에서 조건이 일치하는 데이터만 가져온다.

```sql
SELECT c.customerName,
       o.orderNumber,
       o.orderDate
FROM customers c
INNER JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

위 쿼리는 고객 테이블과 주문 테이블을 `customerNumber` 기준으로 연결한다.  
고객 번호가 양쪽 테이블에 모두 존재하는 데이터만 조회된다.

---

## 3. LEFT JOIN

`LEFT JOIN`은 왼쪽 테이블의 모든 데이터를 유지하면서 오른쪽 테이블과 연결한다.

```sql
SELECT c.customerName,
       o.orderNumber
FROM customers c
LEFT JOIN orders o
    ON c.customerNumber = o.customerNumber;
```

이 경우 주문이 없는 고객도 결과에 나타날 수 있다.  
다만 주문 정보가 없으면 `orders` 쪽 컬럼은 NULL로 표시된다.

![INNER JOIN과 LEFT JOIN의 차이](/img/sql-basics/join-types.svg)

---

## 4. JOIN 뒤에는 거의 ON이 붙는다

JOIN에서 중요한 것은 어떤 기준으로 두 테이블을 연결할지 정하는 것이다.  
이 기준을 `ON`에 작성한다.

```sql
JOIN orders o
  ON c.customerNumber = o.customerNumber
```

처음 공부할 때 `ON`과 `IN`을 헷갈릴 수 있다.

| 구분 | 사용 위치 | 의미 |
|---|---|---|
| `ON` | JOIN 뒤 | 테이블 연결 조건 |
| `IN` | WHERE 또는 서브쿼리 | 여러 값 중 포함 여부 확인 |

---

## 5. 실습 예제: 국가별 총 주문 금액

수업 실습에서는 국가별 총 주문 금액을 구했다.

```sql
SELECT c.country,
       ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS total_revenue
FROM customers c
JOIN orders o         ON c.customerNumber = o.customerNumber
JOIN orderdetails od  ON o.orderNumber = od.orderNumber
GROUP BY c.country
ORDER BY total_revenue DESC
LIMIT 10;
```

이 쿼리의 흐름은 다음과 같다.

```text
customers
→ orders와 customerNumber 기준으로 연결
→ orderdetails와 orderNumber 기준으로 연결
→ 국가별 GROUP BY
→ quantityOrdered * priceEach 합산
→ 매출 상위 10개 국가 조회
```

마케팅 관점에서는 국가별 매출 규모를 보고 지역별 예산 배분이나 캠페인 우선순위를 정할 수 있다.

---

## 6. 실습 예제: 제품 라인별 매출과 판매 수량

```sql
SELECT p.productLine,
       ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS total_revenue,
       SUM(od.quantityOrdered) AS total_qty
FROM products p
JOIN orderdetails od
    ON p.productCode = od.productCode
GROUP BY p.productLine
ORDER BY total_revenue DESC;
```

여기서는 `products`와 `orderdetails`를 `productCode` 기준으로 연결했다.

매출은 다음 식으로 계산했다.

```sql
quantityOrdered * priceEach
```

이후 제품 라인별로 묶어서 총 매출과 총 판매 수량을 구했다.

---

## 7. COUNT(DISTINCT ...)가 필요한 이유

JOIN을 하면 데이터가 늘어날 수 있다.  
예를 들어 한 고객이 여러 주문을 했고, 한 주문 안에 여러 제품이 있다면 고객 한 명이 여러 행으로 반복될 수 있다.

그래서 고객 수나 주문 수를 셀 때는 `COUNT(DISTINCT ...)`가 필요할 수 있다.

```sql
SELECT o.territory,
       COUNT(DISTINCT c.customerNumber) AS customer_count,
       COUNT(DISTINCT ord.orderNumber) AS order_count,
       ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS total_revenue
FROM offices o
JOIN employees e     ON o.officeCode = e.officeCode
JOIN customers c     ON e.employeeNumber = c.salesRepEmployeeNumber
JOIN orders ord      ON c.customerNumber = ord.customerNumber
JOIN orderdetails od ON ord.orderNumber = od.orderNumber
GROUP BY o.territory
ORDER BY total_revenue DESC;
```

`COUNT(c.customerNumber)`를 사용하면 JOIN 때문에 같은 고객이 여러 번 세어질 수 있다.  
반면 `COUNT(DISTINCT c.customerNumber)`는 중복 고객을 한 번만 센다.

---

## 8. 서브쿼리란?

서브쿼리는 하나의 쿼리 안에 포함된 또 다른 쿼리이다.

```sql
SELECT customerNumber,
       customerName,
       creditLimit
FROM customers
WHERE creditLimit > (
    SELECT AVG(creditLimit)
    FROM customers
);
```

이 쿼리는 전체 고객 평균 신용한도보다 높은 고객을 찾는다.

괄호 안의 쿼리가 먼저 평균 신용한도를 구하고, 바깥 쿼리가 그 값보다 큰 고객을 조회한다.

![서브쿼리의 실행 흐름](/img/sql-basics/subquery-flow.svg)

---

## 9. 단일 행 서브쿼리

단일 행 서브쿼리는 결과를 하나의 값만 반환하는 서브쿼리이다.

예를 들어 평균 신용한도는 하나의 값이다.

```sql
SELECT AVG(creditLimit)
FROM customers
```

그래서 바깥 쿼리에서 `>`, `<`, `=`, `<>` 같은 비교 연산자와 함께 사용할 수 있다.

```sql
SELECT customerNumber,
       customerName,
       creditLimit
FROM customers
WHERE creditLimit > (
    SELECT AVG(creditLimit)
    FROM customers
);
```

---

## 10. 다중 행 서브쿼리

다중 행 서브쿼리는 결과가 여러 행으로 나오는 서브쿼리이다.  
이 경우 `IN`, `ANY`, `ALL` 등을 사용한다.

### 10-1. IN

```sql
SELECT customerNumber,
       customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
    WHERE strftime('%Y', orderDate) = '2003'
);
```

`IN`은 서브쿼리 결과 중 같은 값이 있으면 TRUE가 된다.

### 10-2. 2003년과 2004년 모두 주문한 고객

```sql
SELECT customerNumber,
       customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber
    FROM orders
    WHERE strftime('%Y', orderDate) = '2003'
)
AND customerNumber IN (
    SELECT customerNumber
    FROM orders
    WHERE strftime('%Y', orderDate) = '2004'
)
ORDER BY customerNumber;
```

이 쿼리는 2003년에 주문한 고객이면서 동시에 2004년에도 주문한 고객을 찾는다.  
리텐션 마케팅 관점에서는 연속 구매 고객을 식별하는 데 사용할 수 있다.

---

## 11. NOT IN: 주문하지 않은 고객 찾기

한 번도 주문하지 않은 고객을 찾는 쿼리이다.

```sql
SELECT customerNumber,
       customerName,
       country
FROM customers
WHERE customerNumber NOT IN (
    SELECT DISTINCT customerNumber
    FROM orders
)
ORDER BY customerNumber;
```

이런 고객은 잠재 이탈 고객이나 윈백 캠페인 대상이 될 수 있다.

다만 실무에서는 `NOT IN` 내부 결과에 NULL이 포함될 경우 의도하지 않은 결과가 나올 수 있다.  
그래서 상황에 따라 `NOT EXISTS`가 더 안전할 수 있다.

---

## 12. FROM 서브쿼리

서브쿼리는 `FROM` 절에서도 사용할 수 있다.  
이 경우 서브쿼리 결과를 임시 테이블처럼 사용한다.

제품 라인별 매출을 구한 뒤, 그중 상위 3개만 조회하는 쿼리이다.

```sql
SELECT productLine,
       line_revenue
FROM (
    SELECT p.productLine,
           ROUND(SUM(od.quantityOrdered * od.priceEach), 2) AS line_revenue
    FROM products p
    JOIN orderdetails od
        ON p.productCode = od.productCode
    GROUP BY p.productLine
)
ORDER BY line_revenue DESC
LIMIT 3;
```

안쪽 쿼리는 제품 라인별 매출을 계산한다.  
바깥 쿼리는 그 결과에서 매출 상위 3개만 조회한다.

---

## 13. EXISTS

`EXISTS`는 서브쿼리 결과가 존재하는지 여부를 확인한다.

```sql
SELECT DISTINCT c.country
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customerNumber = c.customerNumber
      AND o.status = 'Shipped'
)
ORDER BY c.country;
```

이 쿼리는 배송 완료 주문 이력이 있는 국가 목록을 조회한다.

`EXISTS`에서 중요한 것은 서브쿼리가 어떤 값을 반환하느냐가 아니라, 조건을 만족하는 행이 존재하느냐이다.  
그래서 관습적으로 `SELECT 1`을 자주 사용한다.

---

## 14. 서브쿼리 주의사항

서브쿼리를 사용할 때는 다음을 주의해야 한다.

| 주의사항 | 설명 |
|---|---|
| 괄호 필수 | 서브쿼리는 보통 괄호 안에 작성한다 |
| 위치 주의 | 비교 연산자의 오른쪽에 두는 경우가 많다 |
| SELECT 문 사용 | 서브쿼리는 기본적으로 SELECT 문으로 작성한다 |
| 결과 개수 확인 | 단일 행인지 다중 행인지에 따라 연산자가 달라진다 |

단일 행 서브쿼리에는 보통 `=`, `>`, `<`, `<>` 같은 비교 연산자를 사용한다.  
다중 행 서브쿼리에는 `IN`, `ANY`, `ALL`, `EXISTS` 등을 사용한다.

---

## 15. JOIN과 서브쿼리 중 무엇을 써야 할까?

JOIN과 서브쿼리는 둘 다 여러 테이블의 관계를 활용한다.  
하지만 목적이 조금 다르다.

| 상황 | 추천 방식 |
|---|---|
| 여러 테이블의 컬럼을 함께 출력해야 함 | JOIN |
| 특정 조건을 만족하는 대상만 필터링 | 서브쿼리 |
| 존재 여부만 확인 | EXISTS |
| 집계 결과를 기준으로 필터링 | 서브쿼리 또는 FROM 서브쿼리 |

정답이 하나로 고정된 것은 아니다.  
같은 문제도 JOIN으로 풀 수 있고 서브쿼리로 풀 수 있다.

중요한 것은 “어떤 데이터가 기준 테이블이고, 어떤 조건으로 연결하거나 필터링하는가”를 이해하는 것이다.

---

## 16. 이번 글 정리

이번 글에서는 JOIN과 서브쿼리를 정리했다.

핵심은 다음과 같다.

| 개념 | 핵심 |
|---|---|
| `INNER JOIN` | 양쪽 테이블에서 조건이 일치하는 데이터만 조회 |
| `LEFT JOIN` | 왼쪽 테이블은 모두 유지하고 오른쪽 테이블을 연결 |
| `ON` | JOIN의 연결 기준 |
| 서브쿼리 | 쿼리 안에 포함된 또 다른 쿼리 |
| 단일 행 서브쿼리 | 결과가 하나인 서브쿼리 |
| 다중 행 서브쿼리 | 결과가 여러 개인 서브쿼리 |
| `EXISTS` | 조건을 만족하는 행의 존재 여부 확인 |

JOIN과 서브쿼리는 SQL에서 본격적으로 데이터 분석을 하기 위해 반드시 넘어야 하는 구간이다.  
처음에는 복잡해 보이지만, 결국 핵심은 “테이블을 어떻게 연결하고, 어떤 조건으로 필터링할 것인가”이다.
