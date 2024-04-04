# SQL (null)

## NULL의 의미

- **unknown**    

    - 알려지지 않음. 
    
    - ex. 생일이 NULL인경우 분명 생일은 있지만 알려지지 않음

- unavailable or withheld  

    - 공개하지 않음. 

    - 이용할 수 없음.

- not applicable     
    
    - 집전화를 저장해야하는데 아에 없다면? 적용할 수 없음


=> 즉 NULL은 여러가지 의미를 가진다.

## NULL을 찾는 비교연산자


- NULL을 찾는 비교연산자는 '='나 '!='를 사용하지 않는다.

- IS NULL과 IS NOT NULL을 사용한다.

- 예시

    ```SQL
    SELECT id FROM employee WHERE birth_date IS NULL;
    ```

    ```SQL
    SELECT id FROM employee WHERE birth_date IS NOT NULL;
    ```


## NULL과의 비교연산

위의 내용에서 알 수 있듯이, NULL은 여러가지 의미를 가진다.

따라서 NULL 비교도 일반적인 방법으로 처리할 수 없게된다.

예시를 보자

![alt text](img/1-3/image-8.png)

위의 테이블에서

id가 14인 직원의 생일과 id가 15인 직원의 생일을 같다고 할 수 있을까?

희박한 확률로 같을 수도있고, 다를 수도 있다. **즉 단정지을 수가 없다.**

이러한 문제 때문에 SQL에서는 Three-Valued Logic에 따라서 비교연산을 수행한다.

## Three-Valued logic

SQL에서 NULL과 비교연산을 하게되면 그 결과는 **UNKNOWN** 이다.

UNKNOWN 은 TRUE일수도 있고, FALSE일 수도 있다는 의미이다.

따라서 SQL에서는 비교/ 논리 연산의 결과로 TRUE, FALSE, UNKNOWN 

3가지 값을 사용하게 되고, 이것을 Three-Valued Logic이라고 한다.

### UNKNOWN의 논리연산

- TRUE AND UNKNOWN = UNKNOWN

- FALSE OR UNKNOWN = UNKNOWN

- NOT UNKNOWN = UNKNOWN

### WHERE절의 조건

- SQL에서는 WHERE절의 조건의 결과가 **TRUE인 경우에만 선택**된다.

- 즉 **결과가 FALSE거나 UNKNOWN이면 해당 튜플은 선택되지 않는다.**

### 예제

- 아래의 SQL은 2000년대생이 없는 부서의 ID와 이름을 조회한다.

    ```SQL
    SELECT D.id, D.name
    FROM department AS D
    WHERE D.id NOT IN (
        SELECT E.dept_id
        FROM employee E
        WHERE E.birth_date >= '2000-01-01'
    );
    ```

- 그런데 만약 2000년대 생인 직원이 있는데, 그 직원의 부서가 NULL이라면?

- NOT IN (?, ?, ? , ..NULL) 이런식으로 될 것이다.

- 문제는 여기서 D.id와 NOT IN (?, ?, ?, .. NULL)을 비교연산 할 때이다.

    - NULL값 때문에 해당 비교연산의 결과가 TRUE가 될 수 없다는 것이다. (UNKNOWN이 됨)

    - 그러면 WHERE절의 조건 결과는 TRUE가 될 수 없고, 결국 이 쿼리는 조회할 수 있는 튜플이 없다.

    - 실제로 2000년생이 없는 부서가 있더라도 조회가 안되는 것이다.

- 아래와 같이 수정을 해주어야 원하는대로 동작한다.

    ```SQL
    SELECT D.id, D.name
    FROM department AS D
    WHERE D.id NOT IN (
        SELECT E.dept_id
        FROM employee E
        WHERE E.birth_date >= '2000-01-01' AND E.dept_id IS NOT NULL
    );
    ```

    ```SQL
    SELECT D.id, D.name
    FROM department AS D
    WHERE D.id NOT EXISTS (
        SELECT E.dept_id
        FROM employee E
        WHERE E.dept_id = D.id AND E.birth_date >= '2000-01-01'
    );
    ```

