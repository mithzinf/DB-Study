# 3. stored function

- 사용자가 정의한 함수

- DBMS에 저장되고 사용되는 함수


**[ 참고 ]**

- stored function을 작성할 땐 delimiter를 다른값으로 변경하는 것이 좋다.

  - function 안에서 ';'가 쓰였을 때 statement가 종료되었다고 인식하기 때문이다.

  - 따라서 처음에 stored function 작성 전에 delimiter를 변경하고 

    이후에 다시 원래 delimiter인 ';' 로 되돌려 준다.  

### 예제1

```
임직원의 ID를 열자리 정수로 랜덤하게 발급한다 

ID의 맨 앞자리는 1로 고정이다.
```

```SQL
delimeter $$
CREATE FUNCTION id_generator()
RETURNS int
NO SQL
BEGIN
    RETURN (1000000000 + floor(rand() * 1000000000))
END
$$

delimeter ;
```

**[ 실제 사용 쿼리 ]**

```SQL
INSERT INTO employee
VALUES (id_generator(), 'JEHN', 1991-01-01', 'F', 'PO', 50000000, 1005)
```

### 예제2

```
부서의 ID를 파라미터로 받으면

해당 부서의 평균 연봉을 알려주는 함수를 작성하자
```

```SQL
CREATE FUNCTION dept_avg_salary(d_id int)
RETURNS int
READS SQL DATA
BEGIN
    DECLARE avg_sal int;
    SELECT avg(salary) INTO avg_sal
    FROM employee
    WHERE dept_id = d_id;
    RETURN avg_sal;
END
```

또는 변수를 별도로 선언하지 않고 아래와 같이 작성 가능하다.

```SQL
...
    SELECT avg(salary) INTO @avg_sal
    FROM employee
    WHERE dept_id = d_id;
    RETURN @avg_sal;
...
```

**[ 실제 사용 쿼리 ]**

```SQL
SELECT *, dept_avg_salary(id)
FROM department;
```

### 예제 3

```
토익 800이상을 충족했는지 알려주는 함수를 작성하자
```

```SQL
CREATE FUNCTION toeic_pass_fail(toeic_score int)
RETURNS char(4)
NO SQL
BEGIN
    DECLARE pass_fail char(4);
    IF  toeic_score is null THEN SET pass_fail = 'fail';
    ELSE IF toeic_score < 800 THEN SET pass_fail = 'fail';
    ELSE                          SET pass_fail = 'pass';
    END IF;
    RETURN pass_fail;
END
```

**[ 실제 사용 쿼리 ]**

```SQL
SELECT *, toeic_pass_fail(toeic)
FROM student;
```

### 등록된 stored function 조회하기

```SQL
SHOW FUNCTION STATUS where DB = 'company';
```

### 등록된 stored function의 코드 조회하기

```SQL 
SHOW CREATE FUNCTION id_generator;
```

## stored function을 언제써야할까?

- 유틸함수로 쓰기에는 괜찮다.

- 하지만 비즈니스 로직을 stored function에 작성하는 것은 안좋다.


**[ 느낀점 ]**

유틸함수도 WAS에서 작성하고, 재사용하면 되는거 아닌가?

DB부하 너무 심할것 같은데? WAS vs DB 둘 중 하나 괴롭혀야한다면 WAS를 괴롭히는게 맞지 않나? 

그리고 이렇게하면 오히려 성능적으로도 더 안좋을 경우도 많을 것 같은데?

예제 2번의 실제 사용쿼리 보면 N+1처럼 작동할 것 같은데?

이거 왜쓰지?