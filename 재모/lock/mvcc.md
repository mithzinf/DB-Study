## MVCC (MultiVersion Concurrency Control)
MVCC는 lock 기반 동시성 제어 기법과 달리, 여러 버전의 데이터를 유지함으로써 read와 write가 서로 block되지 않도록 하는 동시성 제어 기법이다.   
즉, write-write 외에는 모두 허용해 동시 처리량을 증가시킨다.

### 특징
- 데이터를 읽을 때 특정 시점(isolation level에 따라 다름) 기준으로 가장 최근에 commit된 데이터를 읽는다.   
  이를 MySQL에서는 **Consistent read**라고 부른다.
- 데이터 변경 이력을 내부에서 관리한다. 따라서 추가적인 저장 공간을 더 많이 사용한다.
- read와 write는 서로를 block하지 않는다.
- Lock 기반 동시성 제어 기법보다 좋은 성능을 가지고 있어 오늘날 많이 사용된다.

### 예제
x에는 10이 저장되어 있으며, 트랜잭션 1과 2는 각각 x를 읽고 x를 50으로 바꾸는 작업을 수행한다.

<img width="500" src="img/mvcc_ex.png">

먼저 트랜잭션 2가 시작되어 write_lock(x)을 획득하고, 트랜잭션 2만 접근 가능한 공간에 x=50을 write한다.   
트랜잭션 2가 변경 내용을 commit하려고 할 때 트랜잭션 1이 x를 read하는데, MVCC는 commit된 데이터만 읽으므로 50이 아닌 10을 가져온다.   
그리고나서 트랜잭션 2는 변경 내용을 commit하여 wriet_lock(x)을 해제한다.
 
> RDBMS는 내부적으로 recoverability를 위해 트랜잭션이 commit될 때 write_lock을 해제하도록 구현되어 있다.

마지막으로 트랜잭션 1이 x를 또 다시 읽게 되는데, 이때 x의 값은 isolation level에 따라 달라진다.   
- #### read committed   
  read하는 시점을 기준으로 그 전에 commit된 데이터를 읽어온다. 따라서 read(x) = 50이 된다.
- #### repeatable read   
  트랜잭션 시작 시점을 기준으로 그 전에 commit된 데이터를 읽어온다. 따라서 read(x) = 10이 된다.
- #### serializable
  - MySQL => MVCC가 아닌 lock 기반으로 동작한다.
  - PostgreSQL => SSI(Serializable Snapshot Isolation) 기법이 적용된 MVCC로 동작한다.
- #### read uncommitted
  MVCC는 commit된 데이터를 읽어오기 때문에 'read uncommitted' level에서는 보통 MVCC가 적용되지 않는다.

