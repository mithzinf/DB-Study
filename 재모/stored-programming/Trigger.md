## Trigger란?
DB에서 어떤 이벤트(INSERT, UPDATE, DELETE 등)가 발생했을 때 자동으로 실행되는 프로시저를 Trigger라고 한다.

### 문법
- trigger 정의
    ```mysql
    delimiter $$
    CREATE TRIGGER trigger_name
    BEFORE UPDATE # {BEFORE | AFTER} {INSERT | UPDATE | DELETE} 실행 시점 
    ON table_name FOR EACH ROW # table_name의 각 row에 대해 실행
    BEGIN # trigger body 시작
        insert into users_log_values(OLD.id, OLD.nickname, now()); # 실행 내용
    END # trigger body 끝
    $$
    delimiter ;
    ```

- 등록된 trigger 확인
    ```mysql
    SHOW TRIGGERS;
    ```

- trigger 삭제
    ```mysql
    DELETE TRIGGERS;
    ```

> #### OLD, NEW 키워드
> trigger를 등록할 테이블의 이벤트 발생 전/후를 기준으로 발생 전이면 OLD, 발생 후면 NEW 키워드를 필드 앞에 붙인다. 

### 예제

- 사용자의 닉네임 변경 이력을 저장하는 트리거   
    ```mysql
    delimiter $$
    CREATE TRIGGER log_user_nickname_trigger
    BEFORE UPDATE
    ON users FOR EACH ROW 
    BEGIN 
        insert into users_log_values(OLD.id, OLD.nickname, now());
    END
    $$
    delimiter ;
    ```
    <img width="20%" src="img/닉네임 변경 이력.png">

- 사용자가 마트에서 상품을 구매할 때마다 지금까지 누적된 구매 비용을 구하는 트리거
    ```mysql
    delimiter $$
    CREATE TRIGGER sum_buy_prices_trigger
    AFTER INSERT 
    ON buy FOR EACH ROW 
    BEGIN 
        DECLARE total INT;
        DECLARE user_id INT DEFAULT NEW.user_id; # user_id라는 변수에 새로 추가되는 user_id 저장
        
        select sum(price) into total from buy where user_id = user_id;
        update user_buy_stats set price_sum = total where user_id = user_id;
    END
    $$
    delimiter ;
    ```
    <img width="30%" src="img/구매 비용 누적.png">
  
---

## Trigger 사용 시 tip
위에서 살펴본 예제 외에도 trigger를 사용할 때 다양한 tip이 존재한다.   
하지만 주의할 점은 아래에 나오는 tip들의 경우 MySQL에서는 지원을 하지 않는다는 것이다.

### 여러 이벤트 한번에 감지하기
trigger를 등록할 때 다음과 같이 INSERT, UPDATE, DELETE를 한 번에 감지하도록 설정할 수 있다.

<img width="25%" src="img/여러 이벤트 감지.png">

### FOR EACH ROW vs FOR EACH STATEMENT
1003 부서에 임지원이 다섯 명이 있을 때, 다음과 같이 FOR EACH ROW를 사용하면 trigger가 5번 실행된다.   
각각의 row에 대해 실행되기 때문이다.  

<img width="25%" src="img/for each row.png">

하지만 다음과 같이 FOR EACH STATEMENT를 사용하면 row 단위가 아니라 statement 단위로 실행된다.  

<img width="25%" src="img/for each statement.png">

### Trigger를 발생시킬 디테일한 조건 지정하기
trigger를 등록할 때 다음과 같이 WHEN을 통해 구체적인 조건을 명시할 수 있다.

<img width="25%" src="img/디테일한 조건 지정.png">

변경한 닉네임이 기존 닉네임과 달라야, 즉 실제로 변경될 경우에만 trigger가 실행된다.

---

## Trigger 사용 시 주의 사항
trigger는 소스코드에 명시되지 않기 때문에 어떤 동작이 일어나는지 파악하기 쉽지 않다.   
예를 들어, 프로시저의 경우 WAS의 코드에 `CALL procedure_name`과 같이 프로시저를 호출하는 로직을 명시한다.   
따라서 소스 코드를 보다가 프로시저의 존재를 파악할 수 있는데, trigger는 그렇지 않다. 이로 인해 개발할 때, 관리할 때 모두 문제를 파악하기 어려워 힘들다는 단점이 존재한다.

또한, trigger를 과도하게 사용하면 예상치 못한 상황에 수많은 trigger들이 연속적으로 실행될 수 있다.   
이는 개발자가 디버깅하기 어렵게 만들 뿐만 아니라, DB에 부담을 주면서 응답 속도를 느리게 만든다.

이렇듯 trigger는 data tier에 숨어 있기 때문에 문서 정리가 특히 중요하다.

> ### 그러면 trigger는 언제 사용하면 좋지?
> 가능하면 코드 레벨에서 처리하는 것이 가장 좋고, 정말 필요하다 싶을 때 trigger를 사용하는 것을 추천한다.   
> - 회원 탈퇴 시 일정 기간동안 탈퇴한 회원 데이터 보관
> - 비밀번호 변경 시 기존의 비밀번호 기록
> - 아이템 이동 경로를 기록하고, 유저가 해킹 당해서 아이템을 잃었을 때 그 기록을 추적하여 아이템 복구 가능
