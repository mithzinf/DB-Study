 # MVCC
1. MVCC 등장 배경 / 설명   
2. postgreSQL - lost update 문제 및 해결방안  
3. mySQL - lost update 문제 및 해결방안

---

1. MVCC 등장 배경
- 동일한 데이터에 대한 read 작업과 write 작업이 동시에 실행될 경우, 한 작업이 실행되는 동안 다른 작업이 멈춰있게 되어 전체 처리량이 저하되고 성능 문제가 발생하였음 -> 이런 문제 해결하기 위해 MVCC 등장

- 특징
  - 데이터 읽기 시, 최근 커밋된 데이터만 읽음
  - read와 write 블록 방지 : MVCC는 read와 write 작업이 서로를 블록하지 않음 -> 그로 인해 lock 기반 동시성 제어에 비해 더 뛰어난 성능 효과 有
  - 오늘날 대부분의 RDBMS는 MVCC 기반으로 동작함

- 장점
  - 데이터를 읽을 때, 특정 시점 기준으로 가장 최근에 commit된 데이터를 읽음
  - 데이터 변화(write) 이력 관리
  - read와 write는 서로를 block하지 않음
---



2. postgreSQL에서lost update 문제 및 해결 방안 : [요약](#lost-update-개념-설명)
> lost update : DB에서 두 개의 트랜잭션이 동시에 같은 데이터를 업데이트할 때 발생하는 현상
> 데이터의 일관성 붕괴, 예상치 못한 결과 초래 有
> 예제를 통해 설명  

✔️**모든 트랜잭션이 read commited인 경우, lost update 발생**  
👉초기값 : x = 50, y = 10 / 트랜잭션 1 : x가 y에 40을 이체한다 / 트랜잭션 2 : x에 30을 입금한다
🎲**기대값 : x = 40, y = 50이 되어야 함**
![image](https://github.com/mithzinf/DB-Study/assets/124668883/b951c489-c55b-45c5-9df8-ae8c004b7465)    
1) 먼저 트랜잭션 1이 실행하여 x값을 읽고("umm.. x값이 현재 50이군"), y에 40을 이체하기 위해 x값을 10으로 변경해준다(write(x=10))  
   이때, MVCC로 동작하기 때문에 가상의 공간을 만들어 두고 그 안에 x = 10값을 넣어놓음  
2) 트랜잭션 2도 실행하여, x에 30을 입금해야하므로 현재의 x값을 읽어준다  
3) 현재 commit된 x의 값은 50이기 때문에 (x = 10이라는 값은 오롯이 트랜잭션 1의 가상 공간에 저장된 값이고 commit이 되어 있지 않기 때문에 영향 미치지 x) 트랜잭션 2는 x의 값을 50으로 읽어옴  
4) x+30이라는 트랜잭션 2의 연산에 의해 write(x = 80) 실행 > 실행하기 위해서는, 먼저 x에 대한 write_lock을 선취득해야함   
   ![image](https://github.com/mithzinf/DB-Study/assets/124668883/fa73317e-296e-4fd7-b0d7-26c2baa53abe)  

5) BUT x에 대한 write_lock은 트랜잭션 1이 보유하고 있는 상황이기 때문에, 트랜잭션 1의 write_lock이 해제가 될 때까지 트랜잭션 2는 기다려야 함(가마니 상태)
6) 그런 상태에서, 트랜잭션1은 못다한 작업(y에 40을 더해주는 작업)을 다시 진행하게 됨
7) y에 40을 더해주는 작업을 실행하기 위해, 먼저 y의 값을 읽어준다 : read(y) = 10
8) y값을 읽은 후, y에 40을 더해주기 위해 write(y=50) 작업을 실행하기 위해서 먼저 y에 대한 write_lock(y)을 취득해야함
9) write_lock(y) 취득한 후, 트랜잭션1 내 가상 공간에 y = 50값을 넣어놓음  
![image](https://github.com/mithzinf/DB-Study/assets/124668883/5974276f-6444-475a-a587-b5e6c41a5054)   
10) 트랜잭션1에서 해야할 작업은 다 마쳤기 때문에, 트랜잭션1은 commit을 실행함 & 트랜잭션1의 가상 공간에 저장되어 있던 x,y값은 실제로 DB상에 저장됨 : **x = 10, y = 50**
11) write_lock을 기다리고 있던 트랜잭션2는 write_lock을 획득함 *마참내...* & 트랜잭션 초기에 진행하려다가 말았던 write(x = 80) 실행하게 됨
12) 트랜잭션 1때와 동일하게, 트랜잭션 2 또한 MVCC로 동작하기 때문에 트랜잭션 2만의 가상 공간을 만든 후 그 안에 x = 80 값을 넣어놓음
13) 그 다음 트랜잭션 2 commit 실행하게 되면, 트랜잭션 2의 가상공간에 저장되어 있던 x = 80값이 DB에 실제로 저장되므로 x와 y값은 다음과 같이 변한다 **최종 결과 : x = 80, y = 50**
14) 최종적으로 write_lock은 commit 등판과 동시에 사라지게 된다

---


## lost update 개념 설명
기대값 : x = 40, y = 50  
결과값 : x = 80, y = 50으로   
이처럼 기대값과 다른 결과값이 나온 경우를 **lost update** 라고 함


---

### lost update 문제 해결 시도 1 
✔️**트랜잭션 2의 아이솔레이션 레벨을 repeatable read 수준으로 변경 : 결과 실패**    

    
![image](https://github.com/mithzinf/DB-Study/assets/124668883/0752bccb-4835-463e-b5b1-942898894d7b)   

1) 노란색 일직선 기준 윗부분까지는 아까와 똑같이 동작함 : 트랜잭션 1의 commit으로 인해 DB상의 x,y값이 각각 x=10, y=50으로 업데이트 & 두 개의 write_lock은 반환,해제  
2) 트랜잭션1의 write_lock이 해제되는 것을 기다렸던 트랜잭션 2는 바로, x에 대한 write_lock을 획득하게 되고 x값에 80을 대입하게 된다 (write(x = 80) 발동)  
   2-1) BUT 트랜잭션 2는 repeatable read 상태이기 때문에, write(x=80)작업이 실패하게 됨  
   **이유) postgreSQL에서 repeatable read 상태일 때는, 같은 데이터에 먼저 update한 트랜잭션이 commit 되면 후순위 트랜잭션은 rollback이 되기 때문에 *write(x=80)이 빠꾸가 되는 거임* aka first updater win 이라고 칭함**  



##### 결과 : 트랜잭션 1만 commit, 트랜잭션 2는 rollback으로 마무리됨으로 DB에는 x = 10, y = 50으로 update  
**기대값 : x = 40, y = 50과 다른 값이 출력되어 lost update 해결 실패**  
   ![image](https://github.com/mithzinf/DB-Study/assets/124668883/679bb00e-f39a-4549-873e-d551a5032c88)    
   **=> 한 트랜잭션(트랜잭션2)의 격리 수준을 repeatable read로 승격한다 : 실패**    
   하지만...그 이후, 트랜잭션 2는 rollback 이후 x에 30을 입금하는 작업을 재개하면 기대값 출력 될수도 있음   

---



### lost update 문제 해결 시도 2
✔️**트랜잭션1, 트랜잭션 시작 순서를 바꿔서 트랜잭션2가 먼저 실행되도록 한다면? (트랜잭션1 : read committed & 트랜잭션2 : repeatable read인 상황) : 결과 실패**



![image](https://github.com/mithzinf/DB-Study/assets/124668883/f17b8f7f-76d6-4784-b041-2945d8f599ad)  

1) 트랜잭션2가 먼저 동작하고, x에 30을 입금하기 위해 먼저 x값을 읽어온다 : read(x) => 50
2) 그 다음 트랜잭션 1이 연이어 동작하여 x값을 읽어옴 : read(x) => 50
3) 'x에 30을 입금'하기 위해 트랜잭션 2를 통해 write(x=80)를 실행하자 -> So, 트랜잭션 2에 가상 공간이 생기게 되고 그 안에 x = 80이 들어가게 된다  
   ![image](https://github.com/mithzinf/DB-Study/assets/124668883/8c36c201-e266-489d-80aa-1f13c9a2cc6e)  
4) 그 후, 트랜잭션 1이 연이어 동작하여, x가 y에 40을 이체하기 위해 x의 값을 10으로 다시 써준다 : 
 write(x=10)
 4-1) 트랜잭션에서 write를 실행하기 위해서는, write_lock을 먼저 보유하고 있어야 함
 4-2) write_lock은 현재, 트랜잭션 2이 x에 대한 write_lock을 가지고 있기 때문에 트랜잭션1은 트랜잭션2가 실행을 다 마칠 때까지 그저 기다리고 있어야 함  
   ![image](https://github.com/mithzinf/DB-Study/assets/124668883/0b30fb10-77f6-4da4-8f46-55a3895a6d65)
5) 트랜잭션 2가 commit 명령어 실행 : DB 상의 x값 : 80, y값 : 10으로 update 및 write_lock 해제
6) 트랜잭션 1이 write_lock 보유 - x가 y에 40을 이체하기 위해 x의 값을 10으로 다시 써준다 : 
 write(x=10)
 6-1) 트랜잭션 1 내 MVCC 가상공간에 x = 10을 저장
7) 그 후, y+40 작업을 해줘야 하므로 y값을 read한 후 write(y=50)을 해준다 : read(y)=>10과 write(y=50) 실행함
   7-1) 트랜잭션 1 내 MVCC 가상공간에 y = 50 저장  
   ![image](https://github.com/mithzinf/DB-Study/assets/124668883/2a133ae5-24c0-49e7-8da2-96b6694c57f7)    
8) 일련의 이체 작업 완료 후 트랜잭션 commit 명령어 실행
9) DB상의 x,y값 : **x=10 y=50** 으로 update 및 lock 해제   

##### 결과 : 트랜잭션 1,2 둘다 commit으로 마무리 & DB에는 x = 10, y = 50으로 update  
**기대값 : x = 40, y = 50과 다른 값이 출력되어 lost update 해결 실패**  
**=> 트랜잭션 1의 격리 수준이 read committed로는 충분하지 않음**  


---


### lost update 문제 해결 시도 3
✔️**만일, 트랜잭션1, 트랜잭션2 둘 다 repeatable read로 격리수준을 바꿔주게 된다면? : 기대값과 같은 결과값 출력 성공**
- repeatable read : 같은 데이터를 먼저 update한 선순위 트랜잭션이 commit 상태가 되면, 후순위 트랜잭션은 rollback이 된다 

![image](https://github.com/mithzinf/DB-Study/assets/124668883/7add0058-bb05-4622-ba99-1920d4a84d62)  
1) 트랜잭션2가 먼저 동작 : x에 30 입금을 실행하기 위해 먼저 x값을 읽어온다 : read(x) = 50
2) 이어서 트랜잭션1도 동작 : 동작 후 x값을 읽어 온다 : read(x)=50
3) 트랜잭션2의 연산부터 진행할 것이기 때문에, x에 30을 입금하는 연산 실행 : write(x = 80) & write_lock 발동
4) 그 다음 트랜잭션1의 'x가 y에 40 이체' 연산 실행 시도 but 실패: write(x = 10) 하려고 했으나 write_lock이 이미 트랜잭션2에서 발동중이기 때문에, 트랜잭션1은 발목 잡힌 상태로 트랜잭션2의 commit만을 기다려야 함
5) 트랜잭션2는 commit 완 : **즉 x=80, y=10으로 DB상에 저장됨**
6) repeatable read 논리로 인해, 같은 데이터에 먼저 update가 된 트랜잭션2가 commit이 되면 후순위 트랜잭션인 트랜잭션1은 rollback이 된다
7) 트랜잭션1 rollback 후, DB상의 x=80, y=10을 바탕으로 **x가 y에 40 이체** 연산을 다시 진행한다
8) 이체 -> x = x-40 & y = y+40 대입
9) 트랜잭션1 MVCC 구조를 통해, 트랜잭션 1 내 가상공간 하나가 만들어지고, 그 안에 80-40 = 40을 x값에 대입, 10+40 = 50을 y값에 대입하여 넣어놓는다
10) 이어서 트랜잭션1 commit이 이루어지고, 그 결과로 write&read lock 해제 및 DB상의 x,y값이 각각 x=40,y=50으로 update
11) **기대값과 같은 결과값 출력** 꺄아악~!~!





---


3. mySQL에서 lost update 문제 및 해결방안


![image](https://github.com/mithzinf/DB-Study/assets/124668883/38c12727-0953-4d1a-9782-310af7fc1d73)
**결론 : mySQL에서는 lost update를 방지하는 repeatable read같은 개념의 부재로, lost update 같은 문제가 발생하게 됨**
mySQL에서는 위와 같은, repeatable read 같은 개념이 없다 (이런 개념은 postgreSQL에 존재하는 개념)  그래서, write가 실행 실패한다거나 중간에 가로막히는 일이 발생하지 않는다.  mySQL에서는 막힘없이 쭉쭉 진행하게 된다.  



### mySQL에서 lost update 문제에 대한 해결방안은? 다음 포스트에 이어서











