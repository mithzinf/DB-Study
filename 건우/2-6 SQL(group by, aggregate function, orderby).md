# 2. SQL(group by, order by, 집계함수)

## ORDER BY

- 조회 결과를 특정 attribute(s) 기준으로 정렬하여 가져오고 싶을 때 사용

- default 정렬은 오름차순이다.

- 오름차순 -> ASC, 내림차순 -> DESC

### 예제 : 임직원의 정보를 연봉기준 정렬

```SQL
SELECT * FROM employee ORDER BY salary; 
```

- 위의 쿼리는 기본 정렬이므로 오름차순이다.

```SQL
SELECT * FROM employee ORDER BY DESC;
```

### 예제2: 부서별로 정렬을 한 이후에, 같은 부서에서는 연봉으로 정렬

```SQL
SELECT * FROM employee ORDER BY dept_id ASC, salary DESC;
```

- 즉 order by는 attribute 여러개를 이용해서 정렬이 가능하다.

- 이때 우선순위는 앞에 명시된 attribute부터 차례대로 정렬한다.

## aggregate function

- 여러 tuple들의 정보를 **요약**해서 **하나의 값으로 추출**하는 함수

- ex) COUNT, SUM, MAX, MIN, AVG

- 이때 NULL값들은 제외하고 요약값을 추출한다.

### 예제 : 임직원의 수를 조회

```SQL
SELECT COUNT(*) FROM employee;
```

- 조회되는 튜플의 수를 조회한다.

```SQL
SELECT COUNT(dept_id) FROM employee
```

- 이 경우에는 dept_id가 NULL인 케이스는 카운트 하지않는다.

- 즉 튜플수를 카운트하려면 *(asterisk)를 사용하는 것이 좋다.


### 예제2: 프로젝트 2002에 참여한 임직원의 수와 최대연봉 최소연봉 평균연봉 조회

```SQL
SELECT COUNT(*), MAX(salary), MIN(salary), AVG(salary)
FROM works_on W JOIN employee E ON W.empl_id = E.id
WHERE W.proj_id = 2002;
```
## GROUP BY

- 관심있는 attribute(s)를 기준으로 그룹을 나눠서 **그룹별로 aggregate function을 적용하고 싶을 때 사용**

- grouping attribute(s): 그룹을 나누는 기준이 되는 attribute(s)

- grouping attribute(s)에 NULL이 있으면 NULL값을 가지는 튜플끼리 묶인다.


### 예제: 각 프로젝트별로 임직원 수와 최대연봉 최소연봉 평균연봉 조회

```SQL
SELECT W.proj_id, COUNT(*), MAX(salary), MIN(salary), AVG(salary)
FROM works_on W JOIN employee E ON W.empl_id = E.id
GROUP BY W.proj_id;
```

- 조회 결과를 기준으로 W.proj_id 로 그룹핑

- 그룹핑한 그룹별로 각각의 통계를 조회

## HAVING

- GROUP BY와 함께 사용

- aggregate function의 결과값을 바탕으로 **그룹을 필터링 하고 싶을 때 사용**

- HAVING절에 명시된 조건을 만족하는 그룹만 결과에 포함된다.

### 예제
```
프로젝트 참여 인원이 7명 이상인 프로젝트들에 대해서,

각 프로젝트별로 각 프로젝트별로 임직원 수와 최대연봉 최소연봉 평균연봉 조회
```

```SQL
SELECT W.proj_id, COUNT(*), MAX(salary), MIN(salary), AVG(salary)
FROM works_on W JOIN employee E ON W.empl_id = E.id
GROUP BY W.proj_id;
HAVING COUNT(*) >= 7;
```

## 연습

### 예제 : 각 부서별 인원수를 인원수가 많은 순서대로 정렬후 조회

```SQL
SELECT dept_id count(*) AS empl_count
FROM employee, 
GROUP BY dept_id
ORDER BY empl_count DESC;
```

### 예제1 : 각 부서별 - 성별 인원수를 인원수가 많은 순서대로 정렬해서 조회

```SQL
SELECT dept_id sex, COUNT(*) AS empl_count
FROM employee
GROUP BY dept_id, sex
ORDER BY empl_count DESC;
```

### 예제2 : 회사 전체 평균 연봉보다 평균 연봉이 적은 부서들의 평균 연봉을 조회

1. 부서 별로 평균 연봉을 조회

2. having을 이용해서 회사 전체 평균 연봉보다 적은 tuple은 필터링 (이때 subquery 사용)

```SQL
SELECT dept_id, AVG(salary)
FROM employee
GROUP BY dept_id
HAVING AVG(salary) < (
    SELECT AVG(salary) FROM employee
);
```

### 예제3 
```
프로젝트 참여 인원이 7명이상인 프로젝트에 한정해서

각 프로젝트별로 프로젝트에 참여한 90년대생들의 수와 이들의 평균연봉 조회

정렬은 프로젝트id 오름차순 정렬
```

```SQL
SELECT proj_id, COUNT(*), ROUND(AVG(salary), 0)
FROM works_on W JOIN employee E ON w.empl_id = E.id
WHERE E.birth_date BETEWEEN '1990-01-01' AND '1999-12-13'
AND W.proj_id IN (SELECT proj_id FROM works_on
                 GROUP BY proj_id HAVING COUNT(*) >= 7)
GROUP BY W.proj_id;
ORDER BY W.proj_id
```

- 조건에 따라서 HAVING에 걸지 WHERE에 걸지 판단을 해야한다.

- HAVING은 결과값을 바탕으로 그룹을 필터링한다.

## SELECT의 개념적인 실행순서

```SQL
SELECT attribute(s) or aggregate function(s)    -- 6.
FROM table(s)                                   -- 1.
[WHERE condition(s)]                            -- 2.
[GROUP BY group attributes(s)]                  -- 3.
[HAVING group condition(s)]                     -- 4.
[ORDER BY attributes(s)]                        -- 5.
```
