[197. Rising Temperature](https://leetcode.com/problems/rising-temperature/description/)
#### SQL 실행순서
1. FROM (테이블 불러오기)
2. JOIN / ON
3. WHERE (행 필터링)
4. GROUP BY (집계 그룹화)
5. HAVING (집계 결과 필터링)
6. SELECT (컬럼 계산, 여기서 윈도우 함수 실행)
7. ORDER BY
8. LIMIT / FETCH
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
