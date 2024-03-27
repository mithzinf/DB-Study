# Lock - 동시성 제어 기법  
1. lock이란?
2. write-lock  
3. read-lock
4. 2PL 프로토콜
***

1. lock이란?
- 같은 데이터에 또 다른 read/write가 있을때, 발생할 수 있는 예상치 못한 문제에 대한 해결책  

> lock을 통해 예시의 2개의 트랜잭션 처리 실행  
예) Transaction 1 & Transaction 2



1-1. **Transaction 1**에 대한 write lock : x를 20으로 바꾸는 <span style="color:green"> write_lock(x) </span>을 실행  
![image](https://github.com/mithzinf/DB-Study/assets/124668883/22862f69-1414-4d3f-b800-2a7ef22f9fa2)  



1-2. **Transaciont 2**가 연달아 실행되고, write lock을 실행하려 시도 : x를 90으로 바꾸는 <span style="color:blue"> write_lock(x) </span>   
![image](https://github.com/mithzinf/DB-Study/assets/124668883/c30744f9-3864-4b4b-a723-6298216e20d9)  



하지만, **Transaction 1**의 write_lock(x)이 걸려있기 때문에, **Transaction 2**의 write_lock(x)은 실행되지 못하고, **Transaction 1**의 lock이 풀릴 때 까지 기다리게 됨  
![image](https://github.com/mithzinf/DB-Study/assets/124668883/b61de64b-9e1a-41a2-aade-afda2459b7c9)  



1-3. **Transaction 1**의 <span style="color:green"> write_lock(x) </span>이 실행 종료된 후에, **Transaction 2**의 <span style="color:blue"> write_lock(x) </span>이 실행됨  
![image](https://github.com/mithzinf/DB-Study/assets/124668883/0588be9f-52d6-419b-ab75-7070d94bd864)  



1-4. 이후, 다른 트랜잭션에서 write나 read를 할 수 있도록, lock을 풀어주는 **unlock(x)** 를 꼭 해줘야만 함
![image](https://github.com/mithzinf/DB-Study/assets/124668883/259ba61c-bd21-4cc6-a6bc-615b5050345c)



---



2. 🔒write lock

- read / write(INSERT, MODIFY, DELETE)할 때 사용
- 다른 트랜잭션이 같은 데이터를 read/write 하는 것을 허용치 않음



---


3. 🔒read lock(shared lock)

- read할 때 사용
- 다른 트랜잭션이 같은 데이터를 read하는 것 허용


> 하나의 x 값에 대해, 2 개의 Transaction이 read하려할 때 
예) read_lock(x)이 2개가 발생하는 경우


![image](https://github.com/mithzinf/DB-Study/assets/124668883/52f3b369-ff89-4b8d-93dc-ba72ac23b467)  

1.**Transaction 2**이 먼저 데이터 x값을 읽기 위해 접근 시작 : read_lock(x) 획득 & 실행  
2.뒤이어 **Transaction 1**도 데이터 x값을 읽기 위해 접근 시작 : read_lock(x) 획득 & 실행  
*read_lock은 다른 트랜잭션이 같은 데이터를 대해서 읽으려고 할때, 그 읽는 행위를 허용하기 때문에 **Transaction 1**도 read_lock(x)을 획득할 수 있는 것*    
3.두 트랜잭션 다 read가 끝난 후에는, unlock(x)을 해줘야 함  

##### Lock 호환성  


![image](https://github.com/mithzinf/DB-Study/assets/124668883/d1a618fd-7cb8-4adf-9417-9ff6b87f0d71)


---

##### lock을 써도 생기는 이상한 현상...👻
> 정해진 x,y값에 대해, Serial Schedule이 실행되는 경우 결과값은 어떻게 될까?  
예시 A) 트랜잭션 1 -> 트랜잭션 2 순의 Serical Schedule  


![image](https://github.com/mithzinf/DB-Study/assets/124668883/e6c58176-bce0-4153-8fb1-5341bbec9824)



> 정해진 x,y값에 대해, Serial Schedule이 실행되는 경우 결과값은 어떻게 될까?  
예시 B) 트랜잭션 2 -> 트랜잭션 1 순의 Serical Schedule

![image](https://github.com/mithzinf/DB-Study/assets/124668883/96084159-426d-4b88-9ab8-dd77eee55e7c)


**위의 두 개의 serial schedule을 보니, 시작하는 순서에 따라 x,y값이 달라지는 걸 볼 수 있음 : 즉, UnSerializable Schedule이란 뜻**  
**이를 통해 lock을 사용하는 것만으로, serializable 하지 않을 수 있다는 걸 명심**



##### 이상한 현상에 대한 원인 :   unlock(x) -> read_lock(y) -> write_lock(y)가 진행되는 구간

![image](https://github.com/mithzinf/DB-Study/assets/124668883/a8df7d5f-b328-4d70-93a1-4f78db5774a3)


##### 이상한 현상 해결 방안 : (예시 B의 경우) unlock(x)와 write_lock(y)의 순서만 맞바꿔주면 됨


![image](https://github.com/mithzinf/DB-Study/assets/124668883/111c4b6d-fa3c-4c75-ae85-7264b4238b70)



그러면 이와 같이 트랜잭션이 바뀌게 됨



![image](https://github.com/mithzinf/DB-Study/assets/124668883/2c989250-6f39-4537-8508-3cbd8723bfdc)



##### 이상한 현상 해결 방안 : 예시 A의 경우도 똑같이 진행해주면 된다 : write_lock(y) -> unlock(y) 순으로 진행하면 됨



![image](https://github.com/mithzinf/DB-Study/assets/124668883/e94e4ddb-50dd-4887-ab4a-2fd4b4120d4d)




4. 2PL 프로토콜  
- 트랜잭션에서 모든 lock operation이 최초의 unlock operation보다 먼저 수행되도록 하는 것
- Serializability를 보장함
- 단점 : 상대의 unlock 상태를 기다리다가 목 빠지는 교착상태 Deadlock에 빠질 수도 있다


- 2PL 프로토콜의 종류

  - conservative 2PL : 모든 lock을 취득한 뒤, 트랜잭션 시작, 데드락 발생 X, 실용성 꽝  
    ![image](https://github.com/mithzinf/DB-Study/assets/124668883/1a336ad7-9154-4e90-b9ce-689f0d5f930f)    
  - strict 2PL (S2PL) : strict schedule을 보장하는 2PL, recoverability를 보장함, write-lock을 commit/rollback될 때 그 해당 lock의 unlock이 실행
    ![image](https://github.com/mithzinf/DB-Study/assets/124668883/d6b72a81-0c3d-4c40-bd50-f7071a4f5bea)  
  - strong strict 2PL (SS2PL / rigorous 2PL) : strict schedule 보장하는 2PL, recoverability 보장함, read-lock/write-lock 모두 commit/rollback될 때 그 해당 lock에 대한 unlock이 실행, S2PL보다 구현이 쉬움
   ![image](https://github.com/mithzinf/DB-Study/assets/124668883/dc57883b-dce3-4e7f-975c-989b664c1d5e)  




- 2PL 프로토콜의 단점
  - read-read를 제외하고, 한쪽이 block이 되므로 전체 처리량이 좋지 않다
 
    



  
