[197. Rising Temperature](https://leetcode.com/problems/rising-temperature/description/)
#### SQL 실행순서
1. FROM (테이블 불러오기)
2. JOIN / ON
3. WHERE (행 필터링)
   - 그룹을 만들고, 각 그룹에 대해 집계 함수를 계산   
   - 이 단계에서 모든 집계 함수의 결과가 미리 계산   
5. GROUP BY (집계 그룹화)
6. HAVING (집계 결과 필터링)
7. SELECT (컬럼 계산, 여기서 윈도우 함수 실행)
8. ORDER BY
9. LIMIT / FETCH
-> Window functions 은 SELECT 단계에서 계산
```sql
WITH t AS(
    SELECT
        id,
        temperature - LAG(temperature, 1, NULL) OVER(ORDER BY recordDate) as diff
    FROM Weather
)

SELECT id
FROM t
WHERE diff > 0;
```
CTE를 사용해서 해결   
**BUT**, 원래 문제의 의도는 SELF JOIN을 사용해서 푸는 방법   
이렇게 문제는 풀었을 경우, 테이블의 날짜가 하루 간격이 아닌 경우는 오답을 배출한다.   
```sql
SELECT
    w1.id
FROM Weather w1
INNER JOIN Weather w2
ON w1.recordDate = w2.recordDate + INTERVAL '1' DAY
WHERE w1.temperature > w2.temperature;
```
#### SQL JOIN ON   
SQL의 JOIN 조건은 “어떤 행과 어떤 행을 연결할지”를 명시하는 **논리식(boolean expression)** 일 뿐입니다.   
즉, 테이블을 “한 줄씩 순차적으로 하루 더해서 붙이는” 게 아니라, 조건을 만족하는 모든 행 쌍을 찾아서 조합하는 거예요.   


------


[626. Exchange Seats](https://leetcode.com/problems/exchange-seats/description/)
```sql
SELECT
    id,
    CASE
        WHEN MOD(id, 2) = 0 THEN LAG(student, 1, student) OVER(ORDEr BY id)
        ELSE LEAD(student, 1, student) OVER(ORDEr BY id)
    END AS student
FROM Seat
```
SELF JOIN으로 해결해야하는 문제라고 풀이되어 있는데, SELF JOIN하는 방법은 모르겠음.   
```sql
SELECT
    s1.id,
    COALESCE(s2.student, s1.student) AS student
FROM Seat s1
LEFT JOIN Seat s2
ON (
    (s1.id % 2 = 0 AND s2.id = s1.id - 1)
    OR (s1.id % 2 = 1 AND s2.id = s1.id + 1)
)
ORDER BY s1.id;
```


------


[178. Rank Scores](https://leetcode.com/problems/rank-scores/description/)   
```sql
SELECT
    score,
    DENSE_RANK() OVER(ORDER BY score DESC) AS 'rank'
FROM Scores
ORDER BY score DESC;
```
윈도우 함수 사용 풀이 쉽게 가능   
윈도우 함수를 사용하지 않고 풀이   
```sql
SELECT
  s.score,
  (
    SELECT COUNT(DISTINCT s2.score)
    FROM Scores AS s2
    WHERE s2.score >= s.score
  ) AS 'rank'
FROM Scores AS s
ORDER BY s.score DESC;
```   
score를 정렬하고 숫자를 rank 부여하지 않고,   
rank의 진짜 의미를 subquery로 구현   
실제로는 dense_rank를 구현한 것과 같다.   


------


[1251. Average Selling Price](https://leetcode.com/problems/average-selling-price/description/)   
#### SQL에서 날짜 비교   
```sql
WITH unit_price AS(
    SELECT
        us.product_id,
        us.units,
        p.price
    FROM Prices p
    LEFT JOIN UnitsSold us
    ON (us.product_id = p.product_id) AND (p.start_date <= us.purchase_date) AND (p.end_date >= us.purchase_date)
),

total_sum AS (
    SELECT
        product_id,
        SUM(units) AS total_units,
        SUM(units * price) AS total_revenue
    FROM unit_price
    GROUP BY product_id
)

SELECT
    product_id,
    ROUND(total_revenue / total_units, 2) AS average_price
FROM total_sum
```
통과하지 못한 쿼리: UnitsSold가 모두 빈 값으로 주어지는 경우, NULL값을 처리 할 수 없다.   
무제의 요구 조건에서 Prices 데이터가 주어지고 판매량이 0인 경우 평균 값을 0으로 리턴해야한다고 설명.   
```sql
SELECT
    p.product_id,
    IFNULL(ROUND(SUM(p.price * u.units) / SUM(u.units), 2), 0) AS average_price
FROM
    Prices AS p
LEFT JOIN
    UnitsSold AS u
ON
    p.product_id = u.product_id
    AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY
    p.product_id;
```
답안을 찾아보고 이해가지 않는 부분이 두 가지 있다.   
지금 이 테이블의 조인 구조는 Prices 테이블에 UnitsSold 테이블이 -> 1:1+ 이상으로 매핑되는 구조인데,   
LEFT JOIN의 경우 1:1+ 구조가 어떻게 출력되는지 모르겠다.   
다른 하나는 CTE를 써서 나눠서 테이블 연산을 하는것이 좋은지, 답안 처럼 한번에 연산하는게 좋은지 모르겠다.   
- 표준 SQL에서 문자 타입의 날짜를 날짜/시간 타입으로 변환하는 경우
```sql
CAST('2025-10-04' AS DATE)
CAST('2025-10-04 13:45:00' AS TIMESTAMP)
```


------


[1174. Immediate Food Delivery](https://leetcode.com/problems/immediate-food-delivery-ii/description/)   
```sql
SELECT
    ROUND(SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1 ELSE 0 END) / COUNT(order_date) * 100, 2) AS immediate_percentage
FROM Delivery d
WHERE d.order_date IN (
    SELECT MIN(order_date)
    FROM Delivery
    GROUP BY customer_id
);
```
처음 쿼리를 작성할 때, d.order_date만 IN 으로 설정하여, 첫주문 날짜가 겹치는 고객도 함께 집계되는 구조   
```sql
SELECT
    ROUND(SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1 ELSE 0 END) / COUNT(order_date) * 100, 2) AS immediate_percentage
FROM Delivery d
WHERE (d.customer_id, d.order_date) IN (
    SELECT customer_id, MIN(order_date)
    FROM Delivery
    GROUP BY customer_id
);
```
customer_id를 사용하여 data 중복을 해결   
- SUM, COUNT 함수는 행 전체를 다상으로 적용되기 때문에, GROUP BY 없이 적용


------


[184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/description/)   
```sql
WITH top AS (
    SELECT
        d.name,
        e.departmentId,
        MAX(e.salary) AS salary
    FROM Employee e
    LEFT JOIN Department d
    ON e.departmentId = d.id
    GROUP BY d.name, e.departmentId
)
SELECT
    t.name AS Department,
    e.name AS Employee,
    e.salary AS Salary
FROM Employee e
LEFT JOIN top t
ON e.departmentId = t.departmentId AND e.salary = t.salary
WHERE t.name IS NOT NULL;
```
내가 작성한 쿼리는 제공되는 테이블을 join하고, CTE로 생성된 테이블을 다시 조인하는 형태로 작성되었다.   
MAX 값을 구하여 그에 해당하는 값만 출력이 필요하기 때문에, WHERE 절에 서브쿼리로 넣어서 매번 계산을 하는 경우는 피하고 싶었다.   
그러나 JOIN부분에 subquery로 넣으면 더 적은 연산으로 해결할 수 있다...   
```sql
SELECT d.name AS Department,
       e.name AS Employee,
       e.salary AS Salary
FROM Employee e
JOIN Department d
  ON e.departmentId = d.id
JOIN (
  SELECT departmentId, MAX(salary) AS max_salary
  FROM Employee
  GROUP BY departmentId
) m
  ON e.departmentId = m.departmentId
 AND e.salary = m.max_salary;
```
필요한 max을 사용하여 join할 경우 한번만 연산하면 되기 때문에, 연산이 중복되는 것을 피하게 된다.   


------


[특정 기간동안 대여 가능한 자동차들의 대여비용 구하기](https://school.programmers.co.kr/learn/courses/30/lessons/157339)   
```sql
SELECT
  t.CAR_ID,
  t.CAR_TYPE,
  t.FEE
FROM (
  SELECT
    c.CAR_ID,
    c.CAR_TYPE,
    ROUND(c.DAILY_FEE * 30 * (1 - COALESCE(p.DISCOUNT_RATE, 0) / 100)) AS FEE
  FROM CAR_RENTAL_COMPANY_CAR c
  JOIN CAR_RENTAL_COMPANY_DISCOUNT_PLAN p
    ON p.CAR_TYPE = c.CAR_TYPE
   AND p.DURATION_TYPE = '30일 이상'
  WHERE c.CAR_TYPE IN ('세단', 'SUV')
    AND NOT EXISTS (
      SELECT 1
      FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY h
      WHERE h.CAR_ID = c.CAR_ID
        AND h.START_DATE <= DATE '2022-11-30'
        AND h.END_DATE   >= DATE '2022-11-01'
    )
) AS t
WHERE t.FEE < 2000000
ORDER BY t.FEE DESC, t.CAR_TYPE ASC, t.CAR_ID DESC;
```
다른 부분들은 비슷했는데, NOT EXISTS관련 부분이 내 답안과 차이가 있었다.   
NOT EXISTS의 경우 서브쿼리의 결과가 하나도 없을 때 TRUE를 반환하는 명령어이다.   
반대로 EXISTS의 경우는 서브쿼리의 결과가 하나 이상 있을 때 TRUE를 반환하는 명령어이다.   


------


[177. Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/description)   
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN

SET N=N-1;

  RETURN (
      SELECT
        DISTINCT salary
      FROM Employee
      ORDER BY salary DESC 
      LIMIT 1 OFFSET N      
  );
END
```
변수를 입력 받는 방법   
행의 개수 제한: LIMIT   
첫 행을 건너뛰고 몇번째 행부터 출력할지: OFFSET   


------


