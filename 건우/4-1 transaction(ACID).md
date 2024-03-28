# 4-1 transaction

- 단일한 논리적인 작업 단위

- 논리적인 이유로 여러 SQL문들을 단일 작업으로 묶어서 나눠질 수 없게 만든 것

- transaction의 SQL문들 중에 일부만 성공해서 DB에 반영되는 일은 일어나지 않는다.

- 예시로는 은행의 이체 (A -> B로 송금을 하면 A의 계좌도 수정 B의 계좌도 수정 해야함)

## 예제(MySQL)

```SQL
START TRANSACTION;

UPDATE account SET balance = balnace - 2000 WHERE id = 'J';

UPDATE account SET balance = balnace + 2000 WHERE id = 'H';

COMMIT
```

- `AUTOCOMMIT`

    - 각각의 SQL문을 자동으로 Transaction 처리.

    - SQL문이 성공하면 자동 `commit`, 문제있으면 `rollback`

        - `commit`은 DB에 영구적으로 저장하는 것을 의미

        - `rollback`은 D이전 상태로 되돌리는 것을 의미

    - MySQL에서는 default로 `autocommit`이 enabled 되어 있음.

    - 그러나 `START TRANSACTION` 실행과 동시에 `autocommit`이 비활성화 됨.

    - `COMMIT`이 되거나 `ROLLBACK`이 되면 다시 `autocommit`이 활성화 된다. 

## 일반적인 transaction 사용 패턴

1. `transaction` 시작(begin) 한다.

2. 데이터를 읽거나 쓰는 등의 SQL문을 포함해 로직 수행

3. 일련의 과정들이 문제없이 동작한다면 `transaction`을 `commit`

   중간에 문제가 발생했다면 `transaction`을 `rollback`

## 소스코드에서 transaction

```Java
public void transfer(String fromId, String told, int amount) {
    try {
        Connection conn = ..;
        conn.setAutoCommit(false);
        ...

        PreparedStatement stmt 
          = conn.prepareStatement("INSERT INTO my_table (column1, column2) VALUES (?, ?)");
        stmt.setString(1, "value1");
        stmt.setString(2, "value2");
        stmt.executeUpdate(); 

        ...
        conn.commit();
    } catch (Exception e) {
        ...
        conn.rollback();
        ...
    } finally {
        conn.setAutoCommit(true); //다시 원래설정으로 변경(connection pool을 이용하기 때문)
    }
}
```

- 스프링같은 경우에는 AOP를 이용해서 중복코드 분리 (`@Transactional`)

## ACID

 트랜잭션의 4가지 성질

### Atomicity

- ALL or NOTHING

- 전부다 성공하거나, 전부다 실패하거나

- 중간에 SQL문이 실패하면 지금까지의 작업을 모두 취소하고 `rollback`

**[ 개발자가 할일 ]**

- `commit` 시점과 `rollback` 시점을 챙겨야한다.. (`commit`, `rollback` 자체는 DBMS가 함)

    - 이 부분은 프레임워크에서 많은 부분을 default로 통제하고 있어서 모르기 쉬움. 하지만 알아야 함.

    - ex) 스프링에서 unchecked exception이 발생하면 자동 `rollback`

    - `@Transactional` 이 끝나는 범위내에서 `commit`, 그리고 `@Transactional`은 밖으로 전이됨. 

### Consistency

- transaction은 DB 상태를 consistent 상태에서 또 다른 consistent 상태로 바꿔줘야 한다

- consistent 하다는 것은 DB에 정의된 rule을 위반하지 않는 것

    - 이 rule은 DB에 걸려있는 제약조건이나,
    
      개발자가 trigger로 커스텀하게 만든 규칙 일수도 있음. 

- transaction이 DB에 정의된 rule을 위반했는지는 commit 전에 확인하고 DBMS가 알려줌.

- 개발자는 application 관점에서도 transactioin이 consistent하게 동작하도록 신경을 써주어야한다.


### Isolation

- 여러 transaction이 동시에 실행될 때도 혼자 실행되는 것 처럼 동작

    - 서로 영향을 안미치고도록 (격리)

- DBMS는 여러 종류의 isolation level을 제공. 

    - isolation level이 높을수록 격리성이 보장되지만 퍼포먼스는 떨어진다.

- 상황에 따라서 개발자는 isolation level중 어떤 level로 transaction을 동작시킬지 설정을 해야한다.

- 동시성 제어의 주된 목표는 isolation 이다.

### Durability

- commit된 transaction은 DB에 영구적으로 반영된다.

- 즉 DB프로세스가 죽더라도 commit된 상태는 그대로 반영되어 있음.

- 영구적으로 저장한다는 것은 휘발성 메모리(RAM)가 아닌 비휘발성 메모리(HDD, SDD)에 저장함을 의미

## TMI

- DB에 제약조건이 걸려있으면 어떤시점에서 제약조건을 검증할까? `COMMIT` `execute` 

    ```
    execute() 되는 시점에서 제약조건을 검증한다.
    ```

- `COMMIT`이 되기전에 쿼리를 `execute` 한 상태는 DB에 영향을 주는가? 안주는가? (네트워크를 타는가?)

    ```
    위에서 추측할 수 있듯이 DB에 영향을 준다. (네트워크를 탄다) 
    
    다만 임시 반영 상태이고, `COMMIT`이 완료되어야 영속적으로 반영된다.
    ```

- 스프링에서 `@Transaction` 안에서 checked 예외가 발생하면 그 트랜잭션을 재사용할 수 없음.

    - try-catch문을 이용해서 다시 SQL로직을 작성해서 뭐 헤보려고 해도 절대 안됨.

    - https://techblog.woowahan.com/2606/