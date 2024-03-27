# MVCC
1. MVCC 등장 배경 / 설명
2. postgreSQL - lost update 문제 및 해결방안
3. mySQL에서 lost update 문제 및 해결방안 
***


1. MVCC 등장 배경
- 동일한 데이터에 대한 read 작업과 write 작업이 동시에 실행될 경우, 한 작업이 실행되는 동안 다른 작업이 멈춰있게 되어 전체 처리량이 저하되고 성능 문제가 발생하였음 -> 이런 문제 해결하기 위해 MVCC 등장
- 특징
  - 데이터 읽기 시, 최근 커밋된 데이터만 읽음
  - read와 write 블록 방지 : MVCC는 read와 write 작업이 서로를 블록하지 않음 -> 그로 인해 lock 기반 동시성 제어에 비해 더 뛰어난 성능 효과 有
  - 오늘날 대부분의 RDBMS는 MVCC 기반으로 동작함

---




