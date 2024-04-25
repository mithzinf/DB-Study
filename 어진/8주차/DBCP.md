# DB 커넥션 풀


1. DBCP를 안 쓸 때 문제점  
2. DBCP의 개념과 원리  
3. DBCP 설정 방법
4. HikariCP minimumIdle 권장 사항  
5. 적절한 커넥션 수 찾는 법  
---



1. DBCP를 안 쓸 때 문제점


![image](https://github.com/mithzinf/DB-Study/assets/124668883/70f3ba47-4ee2-43f8-9b39-e32df81f48d8)  

- 매번 connection을 열고 닫는 것 : 과정 복잡 & 시간적인 비용 발생
- 서비스 성능에 좋지 않음 (DB는 물론이고, TCP/IP 커넥션 새로 생성하기 위해 리소스 매번 사용해야함.. 이는 SQL 성능에도 도움 하나도 안됨)



2. DBCP (database connection pool) 개념과 원리

요약 : 커넥션 풀 대여 방식  

![image](https://github.com/mithzinf/DB-Study/assets/124668883/cadd1007-3d9c-47b3-8614-f2c9f836580f)  


- connection 재사용
- 열고 닫는 시간 절약
- 애플리케이션 로직은 커넥션 풀에서 받은 커넥션을 사용해서 SQL을 DB에 전달 -> 그 결과를 받아 처리 -> 커넥션을 모두 사용하고 나면 커넥션을 종료하는 것이 아닌, 재사용할 수 있도록 해당 커넥션을 그대로 커넥션 풀에 반환
-   


- 커넥션 풀 : 서버당 최대 커넥션 수를 제한 가능 -> DB에 무한정 연결이 생성되는 것을 막아주어 DB를 보호하는 효과 有
- 커넥션 풀로서 얻는 이점이 매우 크기 때문에 실무에서는 항상 기본으로 사용
- 대표적인 커넥션 풀 오픈소스 : commons-dbcp2, tomcat-jdbc-pool, HikariCP 
- Spring Boot 2.0 부터는 기본 커넥션 풀로 hikariCP를 제공



3. DBCP 설정 방법(MYsql & Hikari)



![image](https://github.com/mithzinf/DB-Study/assets/124668883/7594c9a9-aa4e-439b-a6b0-79073ad3f24f)   

No 1. DB 서버의 설정


##### 주요 파라미터


- max_connections: client와 맺을 수 있는 최대 connection 수
- wait_timeout: connection이 inactive 할 때 다시 요청이 오기까지 얼마의 시간을 기다린 뒤에 close 할 것인지를 결정. 시간 내에 요청이 도착하면 0으로 초기화.
- wait_timeout은 비정상적인 connection 종료, connection을 다 쓰고 반환이 안되는 경우, 네트워크 단절의 경우와 같은 문제 시 connection을 반환할 수 있도록 한다.



No 2. Backend Application에서의 설정  



##### 주요 파라미터  



- minimumIdle: pool에서 유지하는 최소한의 idle connection 수
- idle connection 수가 minimumIdle보다 작고, 전체 connection 수도 maximumPoolSize보다 작다면 신속하게 추가로 connection을 만든다.
- maximunPoolSize: pool이 가질 수 있는 최대 connection 수. idle과 active(in-use) connection 합쳐서 최대 수.
- maxLifetime: pool에서 connection의 최대 수명. maxLifetime을 넘기면 idle일 경우 pool에서 바로 제거, active인 경우 pool로 반환된 후 제거.
  - pool로 반환이 안되면 maxLifetime이 동작하지 않는다. pool로 반환을 잘 시켜주는 것이 중요함.
  - pool로 반환이 안된다면 어쨌든 active한 상태로 인식되는데, MySQL의 wait_timeout 설정으로 DB상의 커넥션을 끊게 되면 backend application 상에 남아있는 같은 connection을 가지고 요청이 또 들어왔을 경우 DB에 요청이 전달되지 않고 exception이 발생하게 된다.
  - DB의 connection time limit보다 몇 초 짧게 설정해야 한다.
- connectionTimeout: pool에서 connection을 받기 위한 대기 시간.




4. HikariCP minimumIdle 권장 사항



- HikariCP에서의 minimumIdle의 기본 값은 maximumPoolSize와 동일 (= pool size 고정)하게 사용하도록 권장한다.

- minimumIdle < maximumPoolSize일 경우, 트래픽이 몰려올 때마다 connection을 추가로 만들어야 하는데 이 connection을 생성하는 속도보다 트래픽이 몰려오는 속도가 더 빠르다면, 백엔드 서버의 응답이 느려질 수 있기 때문에 처음부터 적절한 갯수로 pool size를 고정하도록 가이드를 제시하는 것이다.


5. 적절한 커넥션 수 찾는 법


![image](https://github.com/mithzinf/DB-Study/assets/124668883/8060239d-a42b-436f-8c1b-442c5bf67e1c)    



- 모니터링 환경 구축
  - 서버 리소스, 서버 스레드 수, 데이터베이스 커넥션 풀(DBCP) 등을 모니터링하기 위한 환경을 구축합니다.

- 부하 테스트
  - 백엔드 시스템의 부하를 테스트합니다. 초당 요청 수(request per second)와 평균 응답 시간(avg response time)을 확인하여 시스템 성능을 측정합니다.

- 리소스 확인
  - 백엔드 서버 및 데이터베이스 서버의 CPU, 메모리 등 리소스 사용량을 확인합니다.
    
- 서버 추가 확장
  - 백엔드 서버의 CPU나 메모리 사용량이 일정 수준을 초과한다면, 새로운 서버를 추가로 띄워 문제를 해결할 수 있습니다.

- DB 리소스 최적화
  - 백엔드 서버의 리소스가 적절하더라도 데이터베이스 서버의 CPU나 메모리 사용량이 높다면, secondary 추가, 캐시 레이어 도입, 샤딩(sharding) 등의 방법으로 데이터베이스 리소스 사용량을 최적화합니다.

- 요청 지표 확인
  - 리소스 사용량이 적절하더라도 초당 요청 수와 평균 응답 시간 지표가 좋지 않다면, 서버의 스레드 관리 모델을 살펴봅니다.
  - Thread per request 모델을 사용하는 경우, 활성 스레드 수를 확인하여 병목 현상 여부를 판단합니다.
  - 예를 들어, thread pool의 전체 thread 수가 5인데, 지표가 안좋은 지점에서 active thread 수가 5라면, thread pool의 thread의 개수가 적어 병목현상이 발생해 성능이 안좋을 수 있습니다.

- DBCP Connection 확인
  - thread pool의 개수 문제가 아니라면 다른 각도에서 생각을 해봐야합니다.
  - DBCP의 활성 연결(active connection) 수를 검토합니다. 만약 maximumPoolSize와 active connection이 같다면, 커넥션 수가 부족한 것일 수 있습니다. maximumPoolSize 값을 늘려 부하 테스트를 진행해보면 도움이 됩니다.

- 백엔드 서버 수 고려
  - 사용할 백엔드 서버의 수를 고려하여 DBCP의 max pool size를 결정합니다.









