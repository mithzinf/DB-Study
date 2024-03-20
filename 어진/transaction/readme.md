# Transaction이란?


0. 예제를 통해 트랜잭션 개념 파악하기
'J가 H에게 20만원을 이체한다면, 각자의 계좌는 어떻게 변경되어야 할까?'
![image](https://github.com/mithzinf/DB-Study/assets/124668883/bc30a82c-b547-4559-a45e-f202565cc9b4)


먼저, J의 계좌에서 - 20만원이 되어야 하고, H의 계좌에서는 + 20만원이 되어야 함

  이를, SQL문으로 변환한다면
![image](https://github.com/mithzinf/DB-Study/assets/124668883/ca20ec4a-7010-4fa2-ad6a-36c0cf56544f)


UPDATE account SET balance = balance - 200000 WHERE id = "J";  
UPDATE account SET balance = balance + 200000 WHERE id = "H";  
로 변환 가능  

두 개의 **update** 문이 성립을 해야지만 이체라는 작업이 정상 처리됨 = = Transaction  

1. 트랜잭션 개념 정리

+ **단일한 논리적인 작업 단위**  
+ **논리적인 이유로 여러 SQL문들을 단일 작업으로 묶어서 나눠질 수 없게 만든 것**  
+ **트랜잭션의 SQL문들 중 일부만 성공할 경우 : DB에 반영되지 않는다**


##### J가 H에게 20만원 이체한 것을 transaction으로 구현 & COMMIT의 개념  
```sql
mysql> select * from account;  
```  

| id | balance |
| -- | ------- | 
| J  | 1000000 | 
| H  | 2000000 | 


```sql
mysql> start transaction; //트랜잭션을 시작한다는 명령어  
mysql> UPDATE account SET balance = balance - 200000 WHERE id = "J";  
mysql> UPDATE account SET balance = balance + 200000 WHERE id = "H";  
mysql> COMMIT;  //COMMIT : 지금까지 작업한 내용을 DB에 영구 저장하는 명령어 및 Tranaction 종료  
```


```sql
mysql> select * from account;  
```

| id | balance |
| -- | ------- | 
| J  |  800000 | 
| H  | 2200000 |   


##### 추가로 J가 H에게 30만원 이체한 것을 transaction으로 구현 & ROLLBACK 개념  

```sql
mysql> start transaction; //트랜잭션 시작  
mysql> UPDATE account SET balance = balance - 300000 WHERE id = "J";  
mysql> select * from account;
``` 

| id | balance |
| -- | ------- | 
| J  |  500000 | 
| H  | 2200000 |   

> J의 업데이트문만 실행이 된 상태이기 때문에, H의 잔액에는 변화가 없음

만약 ROLLBACK을 수행하게 되면 어떻게 되는가?

```sql 
mysql> rollback;  //rollback : 지금까지 작업들을 모두 취소하고, transaction 이전 상태로 되돌림 및 transaction 종료
```

##### AUTOCOMMIT 개념
- 각각의 SQL문을 자동으로 transaction 처리 해주는 개념
- SQL문이 성공적 실행 > 자동 COMMIT / SQL문 실행 중 문제 발생 > 자동 ROLLBACK
- Mysql에서는 autocommit 기능이 default로 설정되어있음
- 타 DBMS에서도 대부분 같은 기능을 제공함

1. AUTOCOMMIT을 활성화했을 때 
```sql 
mysql> select @@AUTOCOMMIT;
```

| @@AUTOCOMMIT |
|           1  | 

1의 의미 : TRUE, AUTOCOMMIT이 활성화 ING이라는 뜻  
```sql
mysql> insert into account values ('W',1000000);
```  

*AUTOCOMMIT이 활성화된 상태이기 때문에, insert문을 실행하면 자동 COMMIT이 되면서 account 테이블에 ('W', 1000000) 데이터가 영구 저장됨*  

```sql  
mysql> select * from account;  
```

| id | balance |
| -- | ------- | 
| J  |  800000 | 
| H  | 2200000 | 
| W  | 1000000 | 

2. AUTOCOMMIT을 비활성화 했을 때

```sql
mysql> set AUTOCOMMIT = 0;  
```  

Query OK, 0 rows affected(0.01 sec)  

```sql
mysql> delete from account where balance <= 1000000;
```

Query OK, 2 rows affected(0.01 sec)  

```sql
mysql> select * from account;  
```

| id | balance |
| -- | ------- | 
| H  | 2200000 | 
*autocommit을 off한 후의 delete문 실행 후 출력결과*
*만일 rollback을 한다면, 다시 이전 상태로 돌아갈 수 있음*


  
