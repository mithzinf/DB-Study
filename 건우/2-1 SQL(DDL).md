# SQL (DDL)

## MySQL data type

![alt text](img/image.png)

![alt text](img/image-1.png)

![alt text](img/image-2.png)

![alt text](img/image-3.png)


## Key Constaints

- attribute 하나로 구성될 때는 해당 attribute 옆에 적어도 됨.

- attribute 가 여러개면 DDL문 아래쪽에 작성
    ```SQL
    create table PLAYER(
        team_id VARCHAR(12),
        back_number INT,
        ...
        UNIQUE(team_id, back_number)
    )
    ```

### CHECK constraints

- attribute 값을 제한할 때 사용

- 도메인 제약조건 느낌

- CHECK constarints를 걸때에는 이름을 명확하게 정의하는 좋은 것 같다

- 아마 다른 제약조건은 에러메시지를 잘 띄워줄 것 같은데 얘는 이름을 정의해야 편할듯?


## Foreign Key

attribute가 다른 테이블의 primary key나 unique key를 참조할 때 사용

```SQL
create table EMployee (
    ...
    dept_id INT,
    FOREIGN KEY (dept_id)
        references DEPARTMENT(id)
        on delete reference_option
        on update reference_option
)
```

- reference option이 존재

- delete와 update 각각 다르게 reference opion을 걸수가 있음

### reference option

- CASCADE

    - 참조값의 삭제 / 변경을 참조하는 곳에 그대로 반영

    - ex) 참조하고 있던 레코드가 사라지면, 그 레코드를 참조하고있던 레코드도 삭제

    - JPA의 CASCADE랑 비슷한듯 다름. JPA는 **참조하는 객체의 영속성 상태를 전이**시킴.

    - 그리고 JPA의 CASCADE는 CREATE의 경우에도 반영됨.

- SET NULL

    - 참조값의 삭제 / 변경시 NULL로 변경

- RESTRICT

    - 참조값이 삭제 / 변경 되는것을 금지

    - MySQL 에서 **refernce option을 걸지 않으면 기본적으로 RESTRICT이 적용됨**

- NO ACTION

    - RESTRICT와 유사

    - MySQL에서는 RESTRICT와 NO ACTION이 똑같음.

    - 다른 DBMS에서는 트랜잭션 단위로 참조무결성을 검사


- SET DEFAULT

    - 참조값이 삭제 / 변경시 default 값으로 변경


...그런데 보통 Primary Key를 참조하는데, 참조값이 변경될 일이 있나?

###

## TMI

### DATABASE vs SCHEMA

- MySQL에서는 DATABASE와 SCHEMA가 같은 뜻을 의미한다.

- CREATE DATABASE company = DREATE SCHEMA company

- 다른 RDBMS에서는 의미가 다르게 쓰임 

- ex) PostgreSQL 에서는 SCHEMA가 DATABASE의 namespace를 의미

- 보통 다른 RDBMS는 데이터베이스안에 스키마를 정의하고, 그 스키마 안에 테이블이 정의됨.

### Mysql의 엔진

MySQL은 다양한 엔진을 지원한다.

**[ InnoDB ]**

- MySQL의 기본 스토리지 엔진으로, 트랜잭션 처리와 롤백, 복구 기능을 지원. 

- ACID(원자성, 일관성, 고립성, 지속성) 특성을 제공하여 데이터의 안정성을 보장. 

- 대규모 트랜잭션 처리에 적합합니다.

- MySQL 5.5부터는 InnoDB가 기본 스토리지 엔진이 됨.

**[ MyISAM ]**

- 이전 버전에서 주로 사용되던 스토리지 엔진

- 트랜잭션 처리를 지원하지 않고, 속도가 빠르지만 데이터 무결성을 보장하지 않음.

- 대규모 읽기 작업이 많은 웹 사이트나 블로그에 적합.

- MySQL 5.5 이전의 기본 스토리지 엔진

