## Transaction
논리적인 이유로 여러 SQL문들을 단일 작업으로 묶어서 나눠질 수 없도록 만든 것, 즉 단일한 논리 작업 단위를 트랜잭션이라고 한다.   
단일 작업이라는 말에서 알 수 있듯이 트랜잭션의 SQL문들 중에 일부만 성공하면 DB에는 반영되지 않는다.

### 트랜잭션 사용 패턴
1. 트랜잭션 시작
2. 데이터를 읽고 쓰는 SQL문들을 포함해서 로직 수행
3. 일련의 과정들이 문제 없이 동작했다면 commit
4. 중간에 문제가 발생했다면 rollback

### Commit & Rollback
트랜잭션 작업이 모두 성공하면, 해당 작업 내용은 DB에 정상적으로 반영되어야 한다.   
반면에 트랜잭션 작업 중 하나라도 문제가 발생한다면 모든 작업을 취소해야 한다.   
이러한 트랜잭션의 특성 때문에 논리적으로 묶어야 하는 SQL문들을 하나의 논리적인 작업 단위로 구성해야 하는데, 이때 사용하는 명령어가 `Commit`과 `Rollback`이다.   

- **Commit**: 지금까지 작업한 내용을 DB에 영구적으로 저장하는 명령어
    ```mysql
    START TRANSACTION 
    UPDATE account SET balance = balance - 200000 WHERE id = 'A';
    UPDATE account SET balance = balance + 200000 WHERE id = 'B';
    COMMIT;
    ``` 
    만약 A가 B에게 200,000원을 이체하는 경우 A와 B의 잔액이 성공적으로 수정되어 commit하면, 해당 내용은 DB에 영구적으로 저장된다.   
    트랜잭션이 commit되면 DB 사용자는 변경된 데이터의 결과를 확인할 수 있다.  

- **Rollback**: 지금까지 작업한 내용들을 모두 취소하고 트랜잭션 이전의 상태로 되돌리는 명령어
    ```mysql
    START TRANSACTION 
    UPDATE account SET balance = balance - 200000 WHERE id = 'A';
    ROLLBACK;
    ```
    만약 A가 B에게 200,000원을 이체하는 과정에서 B의 잔액은 증가시키지 않고 A의 잔액만 감소시킨 경우 rollback하면 트랜잭션이 종료되고, 변경 내용은 DB에 저장되지 않는다.   
    즉, A와 B의 잔액 모두 원래 상태 그대로 있게 된다.

### Auto commit
생각해보면 나는 SQL문을 작성할 때 commit을 따로 해주지 않았지만, 데이터는 정상적으로 변경되는 것을 확인할 수 있었다.   
이는 DBMS에서 제공하는 auto commit 명령어 때문인데, auto commit이 무엇일까?

auto commit이란 이름 그대로 각각의 SQL문을 자동으로 commit해주는 개념이다.   
SQL문이 성공적으로 실행한다면 commit을, 문제가 발생한다면 rollback을 자동으로 수행한다.   

내가 따로 commit과 rollback을 입력하지 않고도 자동으로 DB에 반영되었던 이유는 MySQL과 대부분의 DBMS에서 auto commit의 default를 enable로 가지고 있기 때문이다.

당연히 auto commit은 임의로 키고 끌 수 있으며, 그 명령어는 다음과 같다. 
```mysql
SET autocommit = 0; # 0: off / 1: on 
```

> #### START TRANSACTION
> 물론 트랜잭션을 위해 auto commit을 항상 수동으로 끌 필요는 없다. 
> 위에서 본 `START TRANSACTION` 명령어는 트랜잭션 실행과 동시에 auto commit을 꺼버리기 때문이다.   
> commit 또는 rollback으로 인해 트랜잭션이 종료되면 auto commit은 다시 원래의 상태로 돌아간다.

### Transaction in Java
우리는 비즈니스 로직을 처리할 때, SQL문을 DB에 직접 작성하지 않고 프로그래밍 언어를 사용한다.   
그 중 Java를 이용해 트랜잭션을 다음과 같이 처리할 수 있다.   
```java
public void transfer(String fromId, String toId, int amount) {
    try {
        connection = ... // DB 커넥션 획득
        connection.setAutoCommit(false); // 트랜잭션 시작
        
        ... // 비즈니스 로직 실행
        
        connection.commit(); // 로직이 정상적으로 실행된 경우 commit
    } catch (Exception e) {
        ...
        connection.rollback(); // 예외 발생 시 rollback
    } finally {
        connection.setAutoCommit(true); // 트랜잭션 종료
    }
}
```
이때 commit과 rollback에 상관 없이 auto commit을 되돌리는 이유는 connection을 재사용하기 때문이다.   
만약 auto commit이 false로 되어 있는지도 모르는 상태에서 SQL문을 실행한다면 DB에 변경 사항이 적용되지 않을 것이다.   
이러한 예기치 못한 문제를 방지하기 위해 auto commit은 다시 원상태로 되돌리는 것이 좋다.

> #### 관심사 분리
> 위에서 본 코드에는 트랜잭션을 수행하는 로직과 비즈니스 로직이 함께 섞여 있다. 이는 하나의 메서드에 서로 다른 여러 개의 관심사가 존재한다는 의미를 가지고 있는 것이다.   
> 이렇게 하나의 메서드가 여러 관심사를 처리하는 것은 SRP에 어긋나므로, Spring의 `@Transactional`과 같은 트랜잭션 관리 추상화 기법을 사용하는 것이 좋다.

---

## ACID
ACID란 트랜잭션이 안전하게 수행된다는 것을 보장하기 위한 4가지 성질을 가리키는 용어다.

### Atomicity(원자성)
원자성은 ***트랜잭션은 분해가 불가능한 최소의 단위인 하나의 원자처럼 동작한다***는 의미를 갖는다.   
다시 말해서 트랜잭션은 논리적으로 단일한 작업 단위이므로, 내부 SQL문들은 모두 성공하거나 모두 실패해야 한다.
중간에 SQL문이 하나라도 실패한다면, 아무일도 없던 것처럼 Rollback을 통해 지금까지의 작업을 모두 취소해야 한다.

원자성 관점에서 DBMS와 개발자의 역할은 다음과 같다.   
Commit을 통해 DB에 영구적으로 저장하는 것과 Rollback을 통해 이전 상태로 되돌리는 것은 DBMS의 역할이다.   
개발자는 이러한 commit과 rollback을 각각 언제 실행할지 결정하면 된다.      
예를 들어 문제 발생 시 rollback을 통해 이전 상태로 되돌릴 수도 있겠지만, plan B로 이어서 또 다른 로직을 수행할 수도 있을 것이다.

### Consistency(일관성)
일관성은 ***트랜잭션이 시작되기 전의 DB 상태가 일관적이었다면, 트랜잭션이 종료된 후의 DB 상태도 일관적이어야 한다***는 의미를 갖는다.   
다시 말해서 트랜잭션 시작 전후 DB는 항상 규칙을 만족하는 상태여야 한다. 따라서 만약 제약조건이나 trigger를 통해 정의된 DB 규칙을 위반하면 rollback해야 한다.     

예를 들어, 계좌 정보를 가지는 ACCOUNT 테이블의 잔액 컬럼은 0보다 크거나 같아야 한다는 제약 조건을 가지고 있다고 가정해보자.   
이때 40만원을 가지고 있던 철수가 맹구에게 80만원을 이체하려고 한다면 트랜잭션은 다음과 같이 수행될 것이다.   
```mysql
UPDATE account SET balance = balance - 800000 WHERE name = '철수';
UPDATE account SET balance = balance + 800000 WHERE name = '맹구';
```
이 SQL문들을 수행하고 나면 철수의 잔액은 -40만원이 되는데, 이는 제약 조건을 위반하므로 일관성이 깨진 것이다.
따라서 위 작업은 rollback을 통해 처음 상태로 되돌려야 한다.

### Isolation(격리성)
격리성(고립성이라고도 부름)은 ***모든 트랜잭션은 다른 트랜잭션에 영향을 주어서도 안 되고, 영향을 받아서도 안 된다***는 의미를 갖는다.   
즉, 여러 트랜잭션들이 동시에 실행될 때도 각자 실행되는 것처럼 동작하게 만들어야 한다.   

예를 들어, 120만원을 가지고 있던 철수가 맹구에게 100만원을 이체하는 동시에 넷플릭스 구독료 50만원이 빠져나간다고 가정해보자.   
그러면 트랜잭션은 다음과 같이 수행된다.   
```mysql
read(balance) >= 100만원 # Transaction A
update(balance = balance - 50만원) # Transaction B 
update(balance = balance - 100만원) # Transaction A
```
가장 먼저 맹구에게 100만원을 이체하는 트랜잭션 A가 시작되어 철수의 잔액을 조회하고 이체가 가능한지 판단한다.   
이때 넷플릭스 구독료가 빠져나가는 트랜잭션 B가 시작되어 잔액에서 50만원을 차감한다.   
그렇게 철수의 잔액은 50만원이 되면서 트랜잭션 B는 종료되는데, 여기서 문제가 발생한다.   
트랜잭션 A는 트랜잭션 B를 전혀 모르고 있으며, 이미 120만원이라는 값을 조회했기 때문에 120만원에서 100만원을 차감한다.   
최종적으로 철수의 잔액에서 넷플릭스 구독료 50만원은 차감되지 않고, 이체로 인한 100만원만 차감된다. (철수 입장에서는 쌩뚱맞게 50만원을 이득본 셈,,) 

이러한 문제를 해결하기 위해 트랜잭션은 Isolation이라는 특성을 가지고 있어야 한다.

> #### Isolation level
> 만약 격리성을 엄격하게 관리한다면 DB의 성능이 감소되고, 격리성을 살살 적용하면 동시성이 증가해 DB의 성능이 향상된다.   
> 이처럼 개발자는 상황에 따라 격리성을 때로는 엄격하게, 때로는 유연하게 관리해야 한다. 이를 위해 DBMS는 여러 종류의 isolation level을 제공한다. 

### Durability(지속성)
지속성은 ***commit된 트랜잭션은 DB에 영구적으로 저장되어 그 결과를 잃지 않아야 한다***는 의미를 갖는다.      
즉, DB 시스템에 문제가 생겨도 commit된 트랜잭션 내용은 무조건 DB에 남아 있어야 한다.   
DBMS는 기본적으로 트랜잭션의 지속성을 보장하므로, 개발자가 특별히 설정해줄 필요는 없다. 