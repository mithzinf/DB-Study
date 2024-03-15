# 데이터베이스란?

전자적으로 (electornically) 저장되고 사용되는

관련있는(related) 데이터들의 조직화된 집합(organized collection)


## DBMS 

- database management systems

- 사용자에게 DB를 정의하고 만들고 관리하는 기능을 제공하는 소프트웨어 시스템

### metadata (catalog)

- 데이터를 설명하기 위한 데이터

- 즉, 데이터베이스를 정의하거나 기술하는 data 

- 데이터 유형, 구조, 제약조건, 인덱스 ... 등

- metadata도 DBMS를 통해 관리된다.

## 데이터 모델

### conceptual data model

- 가장 추상화된 모델

- 일반 사용자도 이해 가능

- 비즈니스 요구사항을 설명하는데 사용

### logical data model

- 중간정도의 추상화 모델

- 데이터가 컴퓨터에 저장될때의 구조와 크게 다르지않은 모델

- 특정 DBMS나 Storage에 종속되지 않은정도로 구조화 가능

- ex) **relational data model** (관계형 모델), object data model, object-relational model...

### physical data model

- 실제 데이터가 저장되는 구조와 가장 근접

## DB 스키마

- 데이터베이스의 구조를 기술

- 테이블이 뭐가있고, Column은 뭐가있는지 등..

## DB State (snapshot)

- 특정 시점에 데이터베이스에 있는 데이터

## three-schema architecture (사실 잘 와닿지는 않음.....)

데이터베이스 시스템을 구축하는 아키텍처중 하나.

왜 3계층으로 나누는가? -> 유저 어플리케이션과 database를 분리시키기 위해

대부분의 DBMS가 이 3계층을 완벽하게, 명시적으로 나누지는 않는다.

### External Schema

- external view, user view 라고도 불림

- 특정 유저들이 필요로하는 데이터만 표현

- Logical Data Model을 통해 표현

### Conceptual Schema

- 전체 데이터베이스에 대한 구조 기술

- 물리적 저장구조는 숨김.

- entities, data types, relationships, user operations, constraints에 집중

- Logical Data Model을 통해 표현

### Internal Schema

- 물리적으로 데이터가 어떻게 저장되는지

- Physical Data Model을 통해 표현

- data storage, data structure, access path (ex : index)

## Database Language

### DDL (Data Definition Language)

- conceptual schema를 정의하기 위해 사용됨.

- internal schema까지 정의할 수도 있음. (그러나 internal schema는 보통 파라미터로 정의)

### SDL (Storage Definition Language)

- internal schema를 정의하기 위해 사용됨.

- 그러나 요즘은 SDL이 거의 없고, 파라미터로 설정

### VDL (View Definition Language)

- external schema 를 정의하기 위해 사용됨.

- 그러나 대부분 DDL이 역할도 수행

### DML (Data Manipluation Language)

- Databse의 Data를 활용하기 위한 언어

- 데이터 추가, 수정, 삭제 , 검색

### 오늘날의 Database Language

DDL , SDL, VDL, DML 이런것들이 따로 존재하기 보다는 통합된 언어로 존재

=> RDB의 SQL


<br>
<br>
<br>
<br>

# 관계형 데이터 베이스란? (Relational Database)

- relation data model 에 기반하여 구조화된 database

- 여러개의 relations로 구성됨.


### 개념 정리

relation -> 테이블

도메인 -> 속성(attribute)

tuple -> 각 attribute 값으로 이루어진 리스트. (테이블 row)

차수(degree) -> attribute의 수

### relation schema

- relation의 구조

- relation 이름과 attirubte의 리스트로 표기

- ex) STUDENT(id, name, grade ,major ...)

- attribute와 관련된 constraints도 포함된다.


### NULL의 의미

- 값이 존재하지 않음

- 값이 실제론 존재하나 아직 모름 (할당이 안됨)

- 해당 사항과 관련이 없다

- 모호하고 중의적인 의미이기 때문에 가능하면 되도록 쓰지 않는게 좋다.

## Keys

- 아래에 attirubtes set 이라는 말이 자주나오는데, set이라고 함은 하나가 아닌 두개이상일 수도 있다는 점을 유념

### superkey

- relation 에서 tuples를 unique하게 식별할 수 있는 attributes set

### candidate key

- 어느 한 attribute라도 제거하면 unique하게 tuples를 식별할 수 없는 super key

- minimal superkey

### primary key

- relation 에서 tuples를 unique하게 식별하기 위해 선택된 candidate key

- 보통 attribute 숫자가 적은 candidate key를 primary key로 고름

### unique key

- primary key가 아닌 candidate key

- 대체키 (alternate key) 라고도 함

### foreigin key

- 다른 relation의 PK를 참조하는 attributes set

## Constraints

relations들이 언제나 지켜줘야하는 제약사항

### domain constraints

- attirbute의 속성과 관련된 제약조건

- ex) age가 음수값일 순 없다. 초등학교의 학년을 나타내는 grade 속성이 6을 넘길수는 없다. 

- 비즈니스와 관련됨

### key constraints

- 서로 다른 tuples는 같은 value의 key를 가질 수 없다.

### NULL value constraints

- attribute가 NOT NULL로 명시되면 NULL을 값으로 가질 수 없다.

### entity integrity constraint

- primary key는 value에 null을 가질 수 없다.

### referential integrity constraint

- FK와 PK의 도메인(attribute)이 같아야함.

- 참조하고 있는 테이블의 PK에 없는 value를 FK값으로 가질 수 없다.



