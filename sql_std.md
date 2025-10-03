# sql join의 종류
- inner join:
- 두 테이블에서 조건에 일치하는 행만 반환
```sql
SELECT u.user_id, u.name, o.order_id
FROM users u
INNER JOIN orders o
  ON u.user_id = o.user_id;
```
