---
title: "SQL 기초 1편: 데이터베이스, SELECT, 조건문, 집계 함수 정리"
date: 2026-07-06
draft: false
categories:
  - Study
tags:
  - SQL
  - Database
  - SELECT
  - WHERE
  - GROUP BY
  - HAVING
---

# SQL 기초 1편: 데이터베이스, SELECT, 조건문, 집계 함수 정리

> 이 글은 이어드림스쿨 SQL 수업 자료와 실습 SQL 파일을 바탕으로 정리한 내용이다.  
> 단순히 명령어를 외우기보다는, 데이터베이스에서 데이터를 조회하고 요약하는 흐름을 이해하는 것을 목표로 한다.

---

## 1. 데이터베이스와 SQL

데이터베이스는 여러 사람이 공유해서 사용할 목적으로 데이터를 통합하여 관리하는 데이터의 모음이다.

SQL은 `Structured Query Language`의 약자로, 데이터베이스에 접근하고 데이터를 조작하기 위한 표준 언어이다.

데이터베이스는 크게 관계형 데이터베이스와 비관계형 데이터베이스로 나눌 수 있다.  
이번 글에서는 테이블 형태로 데이터를 저장하고, 테이블 간 관계를 활용하는 **관계형 데이터베이스**를 기준으로 정리한다.

관계형 데이터베이스는 하나 이상의 테이블로 구성된다.  
테이블은 다시 컬럼과 레코드로 구성된다.

![관계형 데이터베이스의 기본 구조](/img/sql-basics/relational-db-structure.svg)

---

## 2. 테이블, 컬럼, 레코드

테이블은 데이터를 저장하는 표이다.

예를 들어 고객 정보를 저장하는 `customers` 테이블이 있다면 다음과 같은 컬럼을 가질 수 있다.

| 컬럼 | 의미 |
|---|---|
| `customerNumber` | 고객 번호 |
| `customerName` | 고객 이름 |
| `country` | 국가 |
| `creditLimit` | 신용 한도 |

테이블의 각 행은 하나의 데이터이다.  
이 행을 레코드라고 부른다.

---

## 3. SELECT: 데이터를 조회하는 기본 문법

SQL에서 가장 기본이 되는 문법은 `SELECT`이다.

```sql
SELECT 컬럼명
FROM 테이블명;
```

모든 컬럼을 조회하고 싶으면 `*`를 사용할 수 있다.

```sql
SELECT *
FROM customers;
```

하지만 실무에서는 보통 `SELECT *`를 남발하지 않는 것이 좋다.  
데이터가 많을수록 불필요한 컬럼까지 모두 조회하면 속도가 느려지고, 어떤 컬럼을 사용했는지도 명확하지 않기 때문이다.

필요한 컬럼만 명시하는 습관이 좋다.

```sql
SELECT customerNumber,
       customerName,
       country
FROM customers;
```

---

## 4. DISTINCT: 중복 제거

`DISTINCT`는 중복된 값을 제거할 때 사용한다.

```sql
SELECT DISTINCT country
FROM customers;
```

주의할 점은 `DISTINCT` 뒤에 컬럼을 2개 이상 적으면, 컬럼 조합 전체를 기준으로 중복을 판단한다는 것이다.

```sql
SELECT DISTINCT country, city
FROM customers;
```

이 경우 `country`가 같더라도 `city`가 다르면 서로 다른 행으로 취급된다.

---

## 5. WHERE: 조건으로 필터링하기

`WHERE`는 원하는 조건에 맞는 행만 조회할 때 사용한다.

```sql
SELECT customerNumber,
       customerName,
       country
FROM customers
WHERE country = 'USA';
```

조건문에서 자주 사용하는 연산자는 다음과 같다.

| 연산자 | 의미 |
|---|---|
| `=` | 같다 |
| `!=`, `<>` | 같지 않다 |
| `>` | 크다 |
| `<` | 작다 |
| `>=` | 크거나 같다 |
| `<=` | 작거나 같다 |

---

## 6. AND, OR, NOT

조건을 여러 개 연결할 때는 `AND`, `OR`, `NOT`을 사용한다.

```sql
SELECT customerName,
       country,
       creditLimit
FROM customers
WHERE country = 'USA'
  AND creditLimit >= 100000;
```

`AND`는 모든 조건을 만족해야 한다.  
`OR`는 조건 중 하나라도 만족하면 된다.  
`NOT`은 조건을 부정한다.

일부 DB에서는 `AND` 대신 `&&`, `OR` 대신 `||`, `NOT A` 대신 `!A` 형태도 사용할 수 있다.  
하지만 SQL을 처음 공부할 때는 `AND`, `OR`, `NOT`을 명확하게 쓰는 것이 좋다.

---

## 7. BETWEEN, IN, LIKE

### 7-1. BETWEEN

`BETWEEN A AND B`는 A부터 B 사이의 값을 조회할 때 사용한다.

```sql
SELECT customerName,
       creditLimit
FROM customers
WHERE creditLimit BETWEEN 50000 AND 100000;
```

### 7-2. IN

`IN`은 여러 값 중 하나에 포함되는지 확인할 때 사용한다.

```sql
SELECT customerName,
       country
FROM customers
WHERE country IN ('USA', 'France', 'Germany');
```

`A NOT IN B`는 B에 포함되지 않는 A를 찾는 조건이다.

```sql
SELECT customerName,
       country
FROM customers
WHERE country NOT IN ('USA', 'France');
```

### 7-3. LIKE

`LIKE`는 특정 문자열 패턴을 찾을 때 사용한다.

```sql
SELECT productName
FROM products
WHERE productName LIKE '%Ford%';
```

`%`는 앞뒤에 어떤 문자가 와도 된다는 의미이다.

| 패턴 | 의미 |
|---|---|
| `'왕자%'` | 왕자로 시작하는 문자열 |
| `'%왕자'` | 왕자로 끝나는 문자열 |
| `'%왕자%'` | 왕자를 포함하는 문자열 |

---

## 8. INSERT, UPDATE, DELETE

SQL은 조회뿐만 아니라 데이터를 추가, 수정, 삭제할 수도 있다.

### 8-1. INSERT

```sql
INSERT INTO book (title, author)
VALUES ('돈키호테', '세르반테스');
```

### 8-2. UPDATE

```sql
UPDATE book
SET title = 'Don Quixote'
WHERE title = '돈키호테';
```

`UPDATE`를 사용할 때는 `WHERE`를 빼먹으면 위험하다.  
`WHERE`가 없으면 테이블 전체 데이터가 수정될 수 있다.

### 8-3. DELETE

```sql
DELETE FROM book
WHERE title = 'Don Quixote';
```

마찬가지로 `DELETE`도 `WHERE`가 없으면 테이블 전체 데이터가 삭제될 수 있다.

```sql
DELETE FROM book;
```

위 쿼리는 `book` 테이블의 모든 행을 삭제한다.  
실습이 아니라면 매우 조심해야 한다.

---

## 9. ORDER BY와 LIMIT

`ORDER BY`는 정렬할 때 사용한다.

```sql
SELECT productLine,
       MSRP
FROM products
ORDER BY MSRP DESC;
```

`ASC`는 오름차순, `DESC`는 내림차순이다.  
`ORDER BY 컬럼명`만 쓰면 기본값은 `ASC`이다.

```sql
ORDER BY MSRP
```

위 쿼리는 다음과 같다.

```sql
ORDER BY MSRP ASC
```

`LIMIT`은 출력할 행의 개수를 제한한다.

```sql
SELECT productName,
       MSRP
FROM products
ORDER BY MSRP DESC
LIMIT 5;
```

SQLite에서는 다음과 같이 쓸 수도 있다.

```sql
LIMIT 1, 5
```

이 의미는 첫 번째 행부터가 아니라 **두 번째 데이터부터 5개**를 가져오라는 뜻이다.  
인덱스처럼 시작 위치가 0부터라고 생각하면 이해하기 쉽다.

---

## 10. 집계 함수

집계 함수는 여러 행을 하나의 값으로 요약할 때 사용한다.

| 함수 | 의미 |
|---|---|
| `COUNT()` | 개수 |
| `SUM()` | 합계 |
| `AVG()` | 평균 |
| `MAX()` | 최댓값 |
| `MIN()` | 최솟값 |

### 10-1. COUNT

```sql
SELECT COUNT(*) AS customer_count
FROM customers;
```

`COUNT(*)`는 행의 개수를 센다.  
반면 `COUNT(컬럼명)`은 해당 컬럼에서 NULL이 아닌 값의 개수를 센다.

### 10-2. SUM

```sql
SELECT SUM(amount) AS total_payment
FROM payments;
```

### 10-3. AVG

```sql
SELECT AVG(creditLimit) AS avg_credit_limit
FROM customers;
```

### 10-4. MAX, MIN

```sql
SELECT MAX(creditLimit) AS max_credit_limit,
       MIN(creditLimit) AS min_credit_limit
FROM customers;
```

---

## 11. GROUP BY: 기준별로 묶어서 집계하기

`GROUP BY`는 특정 컬럼을 기준으로 데이터를 묶어서 집계할 때 사용한다.

예를 들어 제품 라인별 제품 수를 구하려면 다음과 같이 작성한다.

```sql
SELECT productLine,
       COUNT(*) AS product_count
FROM products
GROUP BY productLine
ORDER BY product_count DESC;
```

이 쿼리는 `products` 테이블을 `productLine` 기준으로 묶은 뒤, 각 제품 라인에 제품이 몇 개 있는지 계산한다.

실습에서는 다음과 같은 문제로 사용했다.

> productLine별 등록 제품 수를 구하고, 제품 수가 많은 순서대로 정렬하기

```sql
SELECT productLine,
       COUNT(*) AS product_count
FROM products
GROUP BY productLine
ORDER BY product_count DESC;
```

---

## 12. HAVING: 그룹화된 결과에 조건 걸기

`WHERE`와 `HAVING`은 둘 다 조건을 거는 역할을 한다.  
하지만 사용하는 위치가 다르다.

| 구분 | 역할 |
|---|---|
| `WHERE` | 그룹화하기 전 개별 행에 조건 |
| `HAVING` | 그룹화한 뒤 집계 결과에 조건 |

예를 들어 `Classic Cars` 라인 제품을 3회 이상 구매한 고객을 찾는 쿼리는 다음과 같다.

```sql
SELECT c.customerNumber,
       c.customerName,
       COUNT(*) AS purchase_count
FROM customers c
JOIN orders o        ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber   = od.orderNumber
JOIN products p      ON od.productCode  = p.productCode
WHERE p.productLine = 'Classic Cars'
GROUP BY c.customerNumber, c.customerName
HAVING purchase_count >= 3
ORDER BY purchase_count DESC;
```

여기서 `WHERE`는 `Classic Cars` 제품만 남기는 역할을 한다.  
그 다음 고객별로 묶고, `HAVING`으로 구매 횟수가 3회 이상인 고객만 남긴다.

즉, 흐름은 다음과 같다.

```text
전체 데이터
→ WHERE로 Classic Cars만 필터링
→ 고객별 GROUP BY
→ COUNT 계산
→ HAVING으로 3회 이상만 필터링
```

---

## 13. 실습 예제: 국가별 고객 수

수업 실습에서는 국가별 고객 수를 구했다.

```sql
SELECT country,
       COUNT(*) AS customer_count
FROM customers
GROUP BY country
ORDER BY customer_count DESC;
```

이 쿼리는 글로벌 마케팅 관점에서 국가별 고객 규모를 파악할 때 사용할 수 있다.

---

## 14. 실습 예제: 제품 라인별 평균 MSRP

제품 라인별 평균 권장소비자가를 구하는 쿼리이다.

```sql
SELECT productLine,
       ROUND(AVG(MSRP), 2) AS avg_msrp
FROM products
GROUP BY productLine
ORDER BY avg_msrp DESC;
```

여기서 `ROUND(AVG(MSRP), 2)`는 평균값을 소수 둘째 자리까지 반올림한다.

---

## 15. 실습 예제: 2004년 주문 건수

SQLite에서는 날짜에서 연도만 추출할 때 `strftime()`을 사용할 수 있다.

```sql
SELECT COUNT(*) AS orders_in_2004
FROM orders
WHERE strftime('%Y', orderDate) = '2004';
```

`strftime('%Y', orderDate)`는 `orderDate`에서 연도만 추출한다.  
그래서 2004년에 해당하는 주문만 필터링할 수 있다.

---

## 16. 이번 글 정리

이번 글에서는 SQL의 가장 기본이 되는 조회, 조건, 정렬, 집계 문법을 정리했다.

핵심은 다음과 같다.

| 개념 | 핵심 |
|---|---|
| `SELECT` | 데이터를 조회한다 |
| `WHERE` | 개별 행을 조건으로 필터링한다 |
| `ORDER BY` | 결과를 정렬한다 |
| `LIMIT` | 출력 개수를 제한한다 |
| `COUNT`, `SUM`, `AVG` | 데이터를 요약한다 |
| `GROUP BY` | 기준별로 묶어서 집계한다 |
| `HAVING` | 그룹화된 결과에 조건을 건다 |

SQL을 공부할 때 처음부터 복잡한 서브쿼리나 윈도우 함수로 넘어가면 헷갈리기 쉽다.  
먼저 `SELECT → WHERE → GROUP BY → HAVING → ORDER BY` 흐름을 익히는 것이 중요하다.
