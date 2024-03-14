## Stored Procedure란?
Stored procedure는 RDBMS에 저장되고 사용되는 사용자 정의 프로시저다.   
주로 구체적인 하나의 태스크(task)를 수행한다.

### 문법
- stored procedure 정의
    ```mysql
    delimiter $$
    CREATE PROCEDURE pdocedure_name(param1, param2, ...)
    BEGIN
        SET result = a * b; # 실행 로직
    END
    $$
    delimiter ;
    ```

- stored procedure 삭제
    ```mysql
    DROP PROCEDURE procedure_name;
    ```

- stored procedure 호출
    ```mysql
    CALL procedure_name(param1, param2, ...);
    ```

- 등록된 stored procedure 확인
```mysql
    SHOW PROCEDURE STATUS;
```

- stored procedure 내용 확인
    ```mysql
    SHOW CREATE PROCEDURE procedure_name;
    ```

### 예제
- 두 정수의 곱셈 결과를 가져오는 프로시저
    ```mysql
    delimiter $$
    CREATE PROCEDURE product(IN a int, IN b int, OUT result int)
    BEGIN
        SET result = a * b;
    END
    $$
    delimiter ;
    ```
    ```mysql
    CALL product(5, 7, @result); # @result: 사용자가 정의한 변수
    SELECT @result;
    ```
    
    <img width="10%" src="img/product 프로시저 결과.png">

- 두 정수를 맞바꾸는 프로시저
    ```mysql
    delimiter $$
    CREATE PROCEDURE swap(INOUT a int, INOUT b int)
    BEGIN
        SET @temp = a;
        SET a = b;
        SET b = @temp;
    END
    $$
    delimiter ;
    ```

- 각 부서별 평균 연봉을 가져오는 프로시저
    ```mysql
    delimiter $$
    CREATE PROCEDURE get_dept_avg_salary()
    BEGIN
        select dept_id, avg(salary) from employee
        group by dept_id;
    END
    $$
    delimiter ;
    ```

- 사용자가 프로필 닉네임을 바꾸면 이전 닉네임을 로그에 저장하고, 새 닉네임으로 업데이트하는 프로시저
    ```mysql
    delimiter $$
    CREATE PROCEDURE change_nickname(user_id int, new_nick varchar(30))
    BEGIN
        insert into nickname_logs (
            select id, nickname, now() from users where id = user_id;
        );
        update users set nickname = new_nick where id = user_id;
    END
    $$
    delimiter ;
    ```

> ### IN, OUT, INOUT
> IN이 붙은 파라미터는 프로시저에 전달하는 값을 의미한다. 이 파라미터는 프로지서 내부에서 값을 수정할 수는 있지만, 프로시저를 호출한 클라이언트가 수정할 수는 없다.
> 
> OUT이 붙은 파라미터는 프로시저가 반환하는 값을 의미한다. 이 파라미터의 초기값은 NULL이며, 프로시저 내부에서 이 파라미터에 값을 저장한 후 반환한다.
> 
> IN과 OUT 외에 INOUT이라는 키워드도 존재하는데, 이 키워드가 붙은 파라미터는 IN과 OUT의 특성을 모두 지닌 것이라고 보면 된다. 다시 말해서, 파라미터로 전달되어 반환까지 되는 값을 의미한다.

---

## Stored Procedure의 장단점 in 3-tier 아키텍처
3-tier 아키텍처에서 Stored Procedure의 목적은 비즈니스 로직을 구현하여 다양한 이점을 취하는 것에 있다.   
여기서 비즈니스 로직을 구현한다는 것은 결국 Logic tier 뿐만 아니라, data tier에도 비즈니스 로직을 두겠다는 의미로 해석된다.   
이에 따라 많은 장점들이 있지만, 결론적으로는 강력한 단점들로 인해 사용을 지양하는 편이다.

### 장점
- #### application에 대해서 transparent하게 동작한다.
    다음과 같이 프로지서를 통해 DB 서버에서 비즈니스 로직을 처리한다고 가정해보자.   
    <img width=30% src="img/transparent 장점.png">   
이때 프로시저의 구현 내용만 변경해준다면, logic tier에 있는 모든 WAS 인스턴스의 소스 코드를 변경하지 않아도 비즈니스 로직을 업데이트 할 수 있게 된다.   
이를 transparent하게 동작한다고 한다.
  
- #### 네트워크 트래픽을 줄여 응답 속도를 향상시킬 수 있다.
    application과 DB는 서로 다른 서버에서 동작하기 때문에, WAS에서 비즈니스 로직을 수행할 때마다 네트워크 트래픽이 발생한다.   
이때 하나의 메서드에 여러 SQL문이 있다면 계속해서 DB 서버에 네트워크 요청을 보내게 된다.   
    <img width=30% src="img/네트워크 트래픽 장점1.png">   
만약 프로시저에서 비즈니스 로직을 수행한다면, WAS는 비즈니스 로직을 처리하기 위해 프로시저만 호출하면 된다.      
따라서 네트워크 트래픽이 감소하게 되고, 이를 통해 응답 속도를 향상시킬 수 있다.   
    <img width="30%" src="img/네트워크 트래픽 장점2.png">



- #### 여러 서비스에서 재사용 가능하다.
    만약 서로 다른 기술로 구현된 여러 서비스들이 같은 DB를 사용할 때(예를 들어 카카오톡, 카카오페이, 카카오뱅크), 프로시저를 사용한다면 어떻게 될까?   
    <img width="30%" src="img/여러 서비스 재사용 가능.png">   
    서로 다른 언어, 프레임워크로 구현된 서비스더라도, 같은 프로시저를 호출해주기만 하면 비즈니스 로직을 처리할 수 있게 된다.

- #### 민감한 정보에 대한 접근을 제한할 수 있다.
    개발자가 DB에 직접 접근하는 것은 막고, 프로시저에만 접근할 수 있도록 허용할 수 있다.   
    이렇게 하면 프로시저를 통해 민감한 정보에 직접 접근하는 것을 막는 것과 동시에 로직을 작성할 수 있다.   

### 단점
위에서 본 장점들은 사실 단점이 될 수도 있으며, 오히려 더 치명적일 수도 있기 때문에 실무에서는 사용을 지양한다.

- #### 유지 보수 비용이 증가한다.
    일부 비즈니스 로직은 소스 코드(logic tier)에서, 또 다른 일부 로직은 DB 서버(data tier)에서 관리하게 된다면 어떻게 될까?   
비즈니스 로직을 파악하거나 처리하기 위해 logic tier와 data tier를 계속해서 왔다갔다해야 하는데, 이로 인해 다음과 같은 문제가 발생한다.
  - 로직을 파악하는 시간이 오래 걸리고 어렵다.
  - 소스 코드와 프로시저 모두 버전 관리가 필요하다.
  - 개발하는 데 사용되는 언어(Java, Python 등)는 물론, 프로시저 관련 문법도 이해해야 한다.

- #### DB 서버를 추가하는 것은 많은 비용이 든다.
    만약 DB 서버에서 비즈니스 로직을 처리할 때, 트래픽이 몰려버리면 DB 서버의 CPU 사용량이 급증하여 장애가 발생할 것이다. (WAS는 분산되어 있으므로 괜찮다고 가정)
DB 서버를 추가하면 되겠지만, 그러면 기존 DB에서 모든 데이터를 모두 복제해야 한다. 이는 굉장한 시간이 걸리므로 긴급 대응을 하지 못할 가능성이 크다.    
    <img width="40%" src="img/db 서버 과부하.png">   
    > 비즈니스 로직을 logic tier에만 둔다면?

  요즘은 오토 스케일링과 같이 클라우드 서비스가 잘 되어 있기 때문에 WAS를 추가 투입하는 것은 상대적으로 쉽다.   
따라서 비즈니스 로직을 logic tier에서만 처리해준다면 CPU 혹은 메모리 부하를 쉽게 분산할 수 있게 된다.

- #### Stored procedure가 항상 transparent인 것은 아니다.
    프로시저의 구현 내용만 변경한다면 위에서 본 것처럼 transparent하겠지만, 시그니처를 변경하게 된다면 어떻게 될까?
당연히 logic tier의 호출 구문도 변경해줘야 한다.   
결국 프로시저를 사용한다고 해서 항상 transparent하게 동작하지는 않는다.

- #### transparent하다고 무조건 좋지는 않다.
    만약 새로 업데이트한 프로시저에 버그가 발견되어 급하게 롤백했더라도, 그 전에 이미 접속했던 모든 사용자에게 피해가 발생했을 것이다.   
    > 그렇다면 비즈니스 로직을 logic tier에만 둔다면 이 문제가 해결될까?
    
    물론 사용자에게 전혀 영향이 가지 않게 하지는 못한다. 하지만 문제의 영향을 최소화할 수는 있다.
아래 그림처럼 4개의 WAS 인스턴스가 있고, 단 하나의 인스턴스 로직만 업데이트한다고 가정해보자.   
<img width=40% src="img/transparent 단점.png">   
우선 하나의 서버만 변경 부분을 적용하고, 해당 서버를 모니터링하다가 문제가 발생했다면 해당 서버를 다시 롤백하면 된다.   
이렇게 되면 전체 트래픽에 문제가 발생하는 것이 아니라, 1/4의 트래픽만 문제가 발생하는 것이다. 즉, 번거롭긴 해도 예상치 못한 문제의 영향을 최소화할 수는 있다.

- #### 재사용 가능함은 양날의 검이다.
    서로 다른 여러 서비스에서 하나의 DB 서버를 사용할 때, 어떤 한 서비스의 트래픽이 급증하게 되면 DB가 과부하될 것이다.   
이는 하나의 DB를 재사용하던 모든 서비스까지 영향을 미치게 된다.   

    > 아키텍처를 개선하면 되겠네?

    다음과 같이 WAS와 DB 서버 사이에 데이터를 관리하는 data service 계층을 도입하면 문제를 해결할 수 있긴 하다.   
    <img width=40% src="img/재사용 가능함 단점.png">   
    모든 서비스가 data service에서 제공하는 api를 사용하다가, 한 서비스의 트래픽이 급증하면 해당 서비스와 연결을 끊어버리면 되기 때문이다.   
    하지만 결국 처음 의도와 달리 서비스들은 프로시저를 재사용하지 않고, data service만 재사용하는 꼴이 된다.

- #### 민감한 정보에 대한 접근을 완전히 제한할 수 없다.
    아무리 프로시저를 통해 DB에 접근할 수 있도록 해도, 개발자가 흑심을 품고 프로시저를 조작한다면 무용지물이 된다.   
게다가 DB에 대한 접근을 막아버리면 개발 및 CS 업무의 신속함도 당연히 떨어지게 된다.

    > 그럼 어떡하라고?

    위 문제를 해결하기 위해서는 다음 3가지 방법을 고려할 수 있다.
    - 담당자나 개발자에게만 DB 권한을 부여한다.
    - 민감한 정보는 암호화해서 저장한다.
    - 보안서약서 등을 통해 정책적으로 보안을 강화한다.


- #### 그 외
    - 오늘날의 프로그래밍 언어는 다양하고 강력한 기능을 제공하는 반면, 프로시저는 복잡하고 유연한 코드를 작성하기 어렵게 만든다.
    - 가독성이 떨어지고, 디버깅이 어려워진다.
