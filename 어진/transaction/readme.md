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


## J가 H에게 20만원 이체한 것을 transaction으로 구현  
\`\`\`  
mysql> select * from account;  
\`\`\`  

| id | balance |
| -- | ------- | 
| J  | 1000000 | 
| H  | 2000000 | 


\`\`\`  
mysql> start transaction; //트랜잭션을 시작한다는 명령어  
mysql> UPDATE account SET balance = balance - 200000 WHERE id = "J";  
mysql> UPDATE account SET balance = balance + 200000 WHERE id = "H";  
mysql> COMMIT;  //COMMIT : 지금까지 작업한 내용을 DB에 영구 저장하는 명령어 및 Tranaction 종료  
\`\`\`
