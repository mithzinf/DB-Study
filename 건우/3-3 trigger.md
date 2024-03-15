# 3. trigger (트리거)

**데이터베이스에서 어떤 이벤트가 발생했을시 자동적으로 실행되는 프로시저**

즉

데이터에 변경이 생겼을 때 (insert, update, delete),

이것이 계기가 되어 자동적으로 실행되는 프로시저를 의미한다.

### 예시 : 닉네임 변경 이력 저장

닉네임이 업데이트 될 때 마다, 로그 테이블에 기존닉네임을 저장을 해준다.

```SQL
CREATE TRIGGER log_user_nickname_trigger
BEFORE UPDATE
ON users FOR EACH ROW
BEGIN
    insert into users_log values(OLD.id, OLD.nickname, now());
END
```

- `BEFORE` : 이벤트 이전에 트리거 발생

- `FOR EACH ROW` : 업데이트 되는 각 ROW에 대해 트리거 실행 

- `OLD` : UPDATE 이전의 튜플을 가르킴 (`BEFORE UPDATE` 이기 때문)

### 예시2

```
사용자가 마트에서 상품을 구매할 때 마다

지금까지 누적된 구매 비용을 구하는 트리거를 작성
```

```SQL
CREATE TRIGGER sum_buy_prices_trigger
AFTER INSERT
ON buy FOR EACH ROW
BEGIN
    DECLARE total INT;
    DECLARE user_id INT DEFAULT NEW.user_id;
    
    -- 집계함수를 통해 누적 비용을 total에 저장
    select sum(price) into total from buy where user_id = user_id; 
    -- 누적 비용을 update
    update user_buy_stats set price_sum = total where user_id = user_id 
END
```

- `NEW` : 여기선 데이터 insert 이후의 튜플을 가르킴 (앞에 `AFTER INSERT`이기 떄문)

## 참고

### 트리거는 여러 이벤트를 한번에 감지 할 수 있다. (단 mysql은 안됨)

- 예시
    ```SQL
        CREATE TRIGGER avg_empl_salary_trigger
        AFTER INSERT OR UPDATE OR DELETE
        ON employee
        FOR EACH ROW
        EXECUTE FUNCTION update_avg_empl_salary();
    ```



### row 단위가 아니라 statement 단위로 trigger가 실행될 수 있도록 할 수 있다. (mysql 안됨)

- 앞전의 예시의 경우 벌크성 업데이트 발생시 트리거가 여러번 수행된다.

    ```SQL
        CREATE TRIGGER avg_empl_salary_trigger
        AFTER INSERT OR UPDATE OR DELETE
        ON employee
        FOR EACH STATEMENT
        EXECUTE FUNCTION update_avg_empl_salary();
    ```

- 하지만 위의 경우는 벌크성 업데이트가 발생해도 한번만 실행된다. 
    (`EACH STATEMENT`와 `EACH ROW`의 차이)

### trigger를 발생시킬 디테일한 조건을 지정할 수 있음 (mysql은 안됨)

```SQL
CREATE TRIGGER log_user_nickname_trigger
   BEFORE UPDATE
   ON userse
   FOR EACH ROW
   WHEN (NEW.nickname IS DISTINCT FROM OLD.nickname)-- 사용자의 닉네임이 기존 닉네임과 다른경우
   EXECUTE FUNCTION log_user_nickname();
```

## 트리거의 단점 및 주의사항

- 트리거는 소스 코드로는 발견할 수 없는 로직임

- 어떤 동작이 일어나는지 파악하기 어렵고 장애시 대응이 힘듬

- `stored procedure`의 경우 WAS의 소스코드에 `stored procedure`에 대한 코드가 존재하기는 함.

- 그러나 trigger는 아에 존재하지 않고, DB에서 발생되는 로직임.

- trigger를 많이 만들게되면 trigger끼리 연쇄적으로 호출되어 예상치 못한 동작을 할 위험이 있음.

- 과도한 trigger는 DB에 부하를 주고 응답을 느리게 만든다.

- 따라서 trigger를 사용할거면 문서정리를 꼭하자. (그런데 가능한 사용안하는것이..... 좋은듯)

