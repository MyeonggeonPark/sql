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
