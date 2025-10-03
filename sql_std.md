# SQL JOIN의 종류
- INNER JOIN:   
- 두 테이블에서 조건에 일치하는 행만 반환   
```sql
SELECT u.user_id, u.name, o.order_id
FROM users u
INNER JOIN orders o
  ON u.user_id = o.user_id;
```




- LEFT JOIN(LEFT OUTER JOIN):   
- 왼쪽 테이블의 모든행 + 오른쪽 테이블 행 반환   
- 오른쪽에 매칭되는 값이 없으면 NULL 채움   
  - LEFT JOIN은 왼쪽(Prices)의 각 행을 최소 1행으로 무조건 유지   
  - 오른쪽에 조건을 만족하는 행이 여러 개(N개) 있으면, 왼쪽 1행이 오른쪽의 N행과 각각 결합되어 N행으로 “증식”   
  - 오른쪽에 조건을 만족하는 행이 0개면, 해당 왼쪽 행은 오른쪽 컬럼이 모두 NULL인 1행   
```sql
SELECT u.user_id, u.name, o.order_id
FROM users u
LEFT JOIN orders o
  ON u.user_id = o.user_id;
```
> 질문: NULL 값을 어떻게 처리할까?, LEFT JOIN과 LEFT OUTER JOIN은 차이가 없을까?




- RIGHT JOIN:
- RIGHT JOIN은 LEFT JOIN의 반대 -> 오른쪽 테이블의 모든 행 + 일치하는 왼쪽 테이블 행 반환   
- 왼쪽에 매칭되는 값이 없으면 NULL 채움
```sql
SELECT u.user_id, u.name, o.order_id
FROM users u
RIGHT JOIN orders o
  ON u.user_id = o.user_id;
```




- FULL OUTER JOIN:
- 왼쪽 + 오른쪽 테이블의 모든 행 반환   
- 매칭 안되면 NULL로 채움
```sql
SELECT u.user_id, u.name, o.order_id
FROM users u
FULL OUTER JOIN orders o
  ON u.user_id = o.user_id;
```
> 질문: 실무 혹은 분석을 하는 경우 왜 FULL OUTER JOIN을 사용할까? 사용하는 경우를 알고 싶다.   




- CROSS JOIN:
- 두 테이블의 카르테시안 곱(모든 조합)
- JOIN 조건을 주지 않으면 두 테이블의 행 개수를 곱한 만큼 결과 생성
```sql
SELECT u.user_id, p.product_id
FROM users u
CROSS JOIN products p;
```
> 예시: 고객 × 상품 모든 가능한 조합, 추천 시스템 후보군 생성(모든 고객에게 모든 상품 매칭)   




- 질문 정리:
1. LEFT JOIN과 LEFT OUTER JOIN은 같다.
2. NULL 값의 처리 방법
   - IS NULL / IS NOT NULL
   - NULL 여부 확인
     ```sql
     SELECT *
     FROM users
     WHERE phone_number IS NULL;
     ```


   - COALESCE(expr1, expr2, ..., exprN)   
   - NULL을 대체하는 표준 SQL 함수
   - 왼쪽부터 차례로 평가해서 첫 번째로 NULL 값이 아닌 값을 반환 -> 이 성질을 이용하여 NULL 값을 치환
     ```sql
     SELECT user_id, COALESCE(phone_number, '미등록') AS phone
     FROM users;
     ```
   - phone_number의 NULL 값을 미등록으로 치환   
   - ? 이 함수는 행단위로 확인을 하기때문에 테이블을 FULL INDEXING 한다고 생각하면 될까?
  

   - CASE WHEN   
   - 조건문으로 NULL 제어   
     ```sql
     SELECT order_id,
       CASE WHEN discount IS NULL THEN 0 ELSE discount END AS discount
     FROM orders;
     ```
   - 할인율이 NULL이면 0으로 변환, NULL이 아니면 원래 할인율 값을 반환   
   - ? 표준 SQL문에서 CASE WHEN 조건 부분에 컬럼으로 구성된 수식을 넣을 수 있을까? 만약 가능하다면 연산에 효율적인 부분은 무엇이 있을까?   
  

   - NULLIF(expr1, expr2)   
   - 두 값이 같으면 NULL 반환, 다르면 첫 번째 값 반환   
     ```sql
     SELECT NULLIF(credit_used, 0) AS valid_credit
     FROM payments;
     ```
   - credit_used가 0이면 NULL 반환, 지금까지와 다르게 특정 조건의 값을 NULL로 치환하는 방법
  

   - 집계 함수와 NULL
   - 표준 SQL 집계 함수는 자동으로 NULL을 제외하고 계산
     ```sql
     SELECT AVG(rating)
     FROM reviews;
     ```
   - 리뷰 평점에서 NULL인 행은 제외하고 평균 계산
  

   - 정렬 시 NULL 처리
   - 표준 SQL: NULLS FIRST, NULLS LAST
     ```sql
     SELECT *
     FROM products
     ORDER BY price NULLS LAST;
     ```
   - 가격이 NULL 값인 상품은 마지막에 표시

  
3. Common Table Expression(CTE)를 사용하는 경우
   - JOIN만 사용하는 경우
   - Subquery 사용하는 경우: 조건 필터링을 하는 경우, 직관적으로 빠를 수 있음
   - CTE를 사용하는 경우: 중간 집계 테이블이 필요한 경우 유지보수, 테이블의 재사용에 용이
