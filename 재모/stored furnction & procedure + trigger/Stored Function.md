## Stored Function이란?
Stored function은 DBMS에 저장되고 사용되는 사용자 정의 함수다.   
주로 공통적으로 적용하고 싶은 로직이 있을 때 사용되며, SQL과 함께 사용할 수 있다.

### 문법

-  stored function 생성
    ```mysql
    delimiter $$

    CREATE FUNCTION function_name(param1, param2, ...) # 함수 시그니처
    RETURNS return_data_type # 리턴 타입
    NO SQL # SQL_DATA_ACCESS (루틴에 대한 데이터 액세스 특성)
    BEGIN # body 시작
        RETURN (1000 + FLOOR(RAND() * 1000)); # 실행 로직
    END # body 끝
    $$

    delimiter ;
    ```

- stored function 삭제
    ```mysql
    DROP FUNCTION function_name;
    ```

- 등록된 stored function 확인
    ```mysql
    SHOW FUNCTION STATUS WHERE DB = 'db_name';
    ```

- stored function 선언문 확인
    ```mysql 
    SHOW CREATE FUNCTION function_name;
    ```

### delimiter를 $$로 변경해야 함에 주의하자.
기존의 delimiter는 세미콜론(;)이기 때문에, 함수 body에서 나오는 세미콜론이 나오면 함수 생성문이 종료된 것으로 판단한다.   
따라서 함수나 프로시저를 생성할 때 delimiter를 임의의 다른 문자로 변경해줘야 하는데, 일반적인 쿼리문에서 거의 사용되지 않는 문자 조합으로 변경한다.   
특히 다른 문자보다 가독성이 좋으면서 관습에 따라 널리 사용되는 '$$'로 변경하는 편이다.

### SQL_DATA_ACCESS
MySQL에 기본적으로 내장되어 있는 INFORMATION_SCHEMA에는 ROUTINES 테이블이 존재한다.   
이 테이블은 Stored 루틴에 대한 여러 정보를 제공하는데, 그 중 하나가 SQL_DATA_ACCESS다.

<img width=50% src="img/routines 테이블.png">

위 사진에서는 많은 컬럼 중에서 극히 일부만 조회했다. 다른 정보에 대해서는 [공식 문서](https://dev.mysql.com/doc/refman/8.3/en/information-schema-routines-table.html)를 참고하자.  

공식 문서에 따르면, SQL_DATA_ACCESS는 루틴에 대한 데이터 액세스 특성이며, 값은 `CONTAINS SQL`, `NO SQL`, `READS SQL DATA`, `MODIFIES SQL DATA` 중의 하나라고 한다.

| 옵션                | 설명                                           |
|-------------------|----------------------------------------------|
| CONTAINS SQL      | 읽기, 수정을 포함한 모든 SQL문을 포함                   |
| NO SQL            | SQL문 사용 안함                                   |
| READS SQL DATA    | 데이터 읽기(SELECT) 전용                            
| MODIFIES SQL DATA | 데이터 읽기 뿐만 아니라 수정(INSERT, UPDATE, DELETE)도 가능 |

### 예제
- 임직원의 ID를 열자리 정수로 랜덤하게 발급하는 함수 (단, 맨 앞자리는 1로 고정)
    ```mysql
    delimiter $$

    CREATE FUNCTION id_generator()
    RETURNS int
    NO SQL
    BEGIN
        RETURN (1000000000 + FLOOR(RAND() * 1000000000));
    END
    $$

    delimiter ;
    ```
    ```mysql
    INSERT INTO employee VALUES (id_generator(), 'JEHN', '1991-08-04', 'F', 'PO', 1000000000, 1005);
    ```

    <img width="40%" src="img/employee 테이블 조회.png">
    
- 부서의 ID를 파라미터로 받으면 해당 부서의 연봉을 알려주는 함수
    ```mysql
    delimiter $$

    CREATE FUNCTION dept_avg_salary(d_id int)
    RETURNS int
    READS SQL DATA 
    BEGIN
        DECLARE avg_sal int;
        select avg(salary) into avg_sal
        from employee where dept_id = d_id;
  
        RETURN avg_sal;
    END
    $$

    delimiter ;
    ```

- 졸업 요건 중 하나인 토익 800 이상을 충족했는지를 알려주는 함수
    ```mysql
    delimiter $$

    CREATE FUNCTION toeic_pass_fail(toeic_score int)
    RETURNS char(4)
    NO SQL
    BEGIN
        DECLARE pass_fail char(4);
        IF toeic_score is null THEN SET pass_fail = 'fail';
        ELSEIF toeic_score < 800 THEN SET pass_fail = 'fail';
        ELSE SET pass_fail = 'pass';
        END IF;
  
        RETURN pass_fail;
    END
    $$

    delimiter ;
    ```

---

## 언제 사용하면 좋을까?
Stored function에 비즈니스 로직을 둘 수도 있겠지만, 되도록 비즈니스 로직은 Logic tier에 두는 것이 좋다.   
만약 stored function, 즉 Data tier에 비즈니스 로직을 둔다면 관리 비용이 증가하기 때문이다.

따라서, 비즈니스 로직보다는 공통적으로 쓰이는 util 함수로 사용하면 좋을 것 같다.
