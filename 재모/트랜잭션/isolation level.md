## 여러 트랜잭션들이 동시에 실행될 때 발생 가능한 이상 현상들

### Dirty read
특정 트랜잭션에 의해 데이터가 변경되었지만 아직 commit되지 않은 상태에서 다른 트랜잭션이 해당 변경 사항을 조회할 수 있는 문제

<img width="500" src="img/dirty_read.png">

트랜잭션 B가 데이터를 변경하고 commit하지 않은 상태에서, 트랜잭션 A가 이어서 작업했는데 트랜잭션 B가 rollback되는 상황에서 이 문제는 치명적으로 다가온다.

### Non-repeatable read
같은 트랜잭션 내에서 같은 데이터를 n번 조회했을 때 읽어온 데이터가 다른 현상

<img width="500" src="img/nonrepeatable_read.png">

### Phantom read
데이터를 조회할 때마다 그 결과가 다르게 나오는 현상 

<img width="500" src="img/phantom_read.png">

---

## Isolation level
여러 이상 현상들이 모두 발생하지 않게 만들면 제약사항이 많아지는데, 그러면 동시 처리가 가능한 트랜잭션의 수가 감소해 DB의 성능이 떨어진다.
따라서 사용자가 임의로 이상 현상들 중에서 일부만 허용할 수 있도록 몇 가지 level을 제공하는데, 이것이 isolation level이다.

### Read uncommitted
- commit되지 않은 트랜잭션의 내용을 다른 트랜잭션이 조회하는 것을 허용한다.
  - dirty read, non-repeatable read, phantom read 모두 허용
- 데이터 무결성을 해칠 수는 있지만, 성능은 가장 빠르다는 장점이 존재한다.

### Read committed
- commit이 완료된 트랜잭션의 변경 사항만 다른 트랜잭션에서 조회할 수 있도록 허용한다.
  - non-repeatable read, phantom read 허용
- 트랜잭션 A에서 데이터가 변경되었지만 아직 commit되지 않은 상태에서, 트랜잭션 B가 해당 데이터를 조회할 경우 트랜잭션 A 시작 전의 데이터를 가져온다.   

### Repeatable read
- 특정 row를 조회할 때 항상 같은 데이터를 가져오는 것을 보장한다.
  - phantom read 허용
- row가 추가되는 것은 막지 못하기 때문에 phantom read 현상은 발생할 수 있다.
- MySQL의 InnoDB 엔진의 기본 격리 수준이 repeatable read다.

### Serializable
- 모든 이상 현상이 발생하지 않는 것을 보장한다.
- 가장 높은 데이터 정합성을 가지지만, 성능이 가장 떨어진다는 단점이 존재한다.

---

## 또 다른 이상 현상들

### Dirty write

### Lost update

### Dirty read 확장

### Read skew

### Write skew

### Phantom read 확장

---

## Snapshot isolation

## 실무에서 isolation level

