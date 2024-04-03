 # MVCC 이어서
1. mySQL에서 lost update 해결법
2. mySQL에서 write skew 해결법
3. postgreSQL에서 write skew 해결법
4. 두 RDBMS의 차이

---


1. mySQL에서 lost update 해결법


> mySQL의 MVCC는 lost update 문제를 repeatable read 만으로 해결할 수 없다, 또 다른 방안이 필요함
> 해결법 : SQL 쿼리에 **FOR UPDATE** 추가


✔️**트랜잭션1, 트랜잭션2를 MYSQL 상에서 작동할 때**
![image](https://github.com/mithzinf/DB-Study/assets/124668883/689699f7-bc25-432f-ab56-5d301f688fda)

1) 트랜잭션2부터 동작함 : MYSQL에서는 x값을 읽을 때 추가적으로 lock을 설정할 수 있도록 해야함
   - 이를 mySQL에서는 **Locking read** 라고 함
   - 개발자가 쿼리문을 짤 때 SELECT balance FROM account WHERE id = 'X'; 이런식으로 짜게 될텐데 이 SQL 쿼리에 FOR UPDATE문을 더해준다
   - 
