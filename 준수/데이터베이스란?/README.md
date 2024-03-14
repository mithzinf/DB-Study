

### Database
전자적으로 저장되고 사용되는, 관련있는(related) 데이터들의 조직화 된 집합

### DBMS 
사용자에게 DB를 정의하고, 만들고 관리하는 기능을 제공하는 소프트웨어 시스템 
* ex) MySQL

DB를 정의하다 보면 부가적인 데이터가 발생하는데, 이를 `metadata` 라고 부른다.

#### Metadata

database를 정의하거나 기술하는 data. 
* ex) 데이터 유형, 구조, 제약 조건, 보안, 저장 등등...

metadata 또한 DBMS를 통해 관리된다.

### database system

쉽게 이야기 해서, `database` + `DMBS` + 관련 applications을 통틀어서 얘기한다. 줄여서 database 라고
이야기 하기도 한다. 


#### data models
데이터베이스의 구조를 기술하는데 사용될 수 있는 개념들이 모인 집합이다. DB 구조를 추상화하여 표현할
수 있는 수단을 제공하며, `data model` 은 여러 종류가 있으며 추상화 수준과 구조화가 조금씩 다르다.

### conceptual data models (개념적 데이터 모델링)

![](/Users/admin/Desktop/스크린샷 2024-03-14 오후 7.16.37.png)

추상화 수준이 가장 높으며, 직관적으로 이해하기 가장 쉽다. 주로 비즈니스 요구사항을 보여줄 때 사용하는
모델링 기법이다. 


### logical data models (논리적 데이터 모델링)

![](/Users/admin/Desktop/스크린샷 2024-03-14 오후 7.21.47.png)

구체화 된 업무 중심 (컴퓨터에 저장 되는 구조와 유사하게)의 데이터 모델링 기법이다. 개념적 데이터 모델링
보다 더 구체적이다. 이 때, `relational data model` 을 대표적으로 많이 사용하는데,

![](/Users/admin/Desktop/스크린샷 2024-03-14 오후 7.24.23.png)

이런식으로 데이터를 테이블 형태로 저장을 하고, 이 테이블을 `relation` 이라고 부른다.

이외에도 object data model, object-relational data model이 존재한다.


### physical data models (물리적 데이터 모델링)

컴퓨터에 데이터가 어떻게 파일 형태로 저장되는지를 기술할 수 있는가에 대한 수단을 제공한다. 
이 단계는 추상적이라기 보다는, 직접 코딩을 통해서 데이터베이스를 만들고, 테이블을 만들수 있다.

```mysql
create table member_tbl ( 
  member_uid bigint primary key auto_increment,
  member_name varchar(45) unique not null,
  member_pwd varchar(45) not null,
  member_status boolean not null
);
출처: https://inpa.tistory.com/entry/DB-📚-데이터-모델링-1N-관계-📈-ERD-다이어그램 [Inpa Dev 👨‍💻:티스토리]
```

#### database schema
data model을 바탕으로 구조를 설명한 것. 데이터베이스를 설계할 때 정해지며 그 후에는 잘 바뀌지
않는다.

schema를 보면서 데이터 베이스의 구조와 큰 그림을 파악할 수 있다.


#### database state

저장된 실제 데이터는 자주 변경 된다.


### three-schema architecture

데이터 베이스를 구축하는 아키텍쳐 중의 하나. 

물리적인 데이터 베이스 구조가 바뀔 때, user application에 영향이 가지 않도록 
분리하는 목적이 있음. (OCP 개념과 유사?)

세 가지 레벨이 존재하며 각각의 레벨마다 `schema`가 정의되어 있다. 

![](/Users/admin/Desktop/스크린샷 2024-03-14 오후 7.35.58.png)

#### internal schema

물리적으로 데이터가 어떻게 저장되는지 **physical data model**을 통해서 표현한다. 

* data storage, data structure 등

#### external schema

가장 유저랑 가까운 `schmea`로, 유저들이 필요로 하는 데이터만 표현하고 그 외의 데이터는 숨긴다.

**logical data model** 을 통해서 표현한다.

#### conceptual schema

전체 데이터 베이스의 구조를 기술하며, internal schema를 한번 추상화 시켜 표현했다. 

* entities, data types, relationships

**logical data models** 를 통해서 기술한다.


결론적으로, 이 아키텍쳐를 사용하는 이유는 각 레벨에서의 변화가 다른 레벨에서의 영향을 주지 않기
위해서 사용한다. 


### DDL (data definition language)

conceptual schema를 정의하기 위해서 사용되는 언어


### SDL (storage definition language)

internal schema를 정의하기 위해 사용 (요즘은 사용 잘 X)

### VDL (view definition language)
external schemas를 정의하기 위해 사용되는 언어

### DML (data manipulation language)

데이터 추가, 삭제, 검색 등등 기능을 제공한다.

