## Database & DBMS & DB system

### Database

Database란 전자적으로 저장되고 사용되는 관련 있는 데이터들의 **조직화된 집합**을 말한다.

> ### 조직화된 집합 vs 조직화되지 않은 집합
> 여러 책들이 꽂혀 있는 책장을 생각해보자. 이때 책장에 있는 책들이 정리가 안 된 채로 있다면 어떤 책이 어디 있는지 파악하기 어려울 것이다.   
> 따라서 특정 책을 찾는 시간이 오래 걸리게 된다. 이처럼 정리되지 않은 집합을 **조직화되지 않은 집합**이라고 한다.   
> 반면에 책들이 잘 정리된 책장에서는 이름 또는 번호를 통해 원하는 책을 쉽게 찾을 수 있다. 이러한 책들을 **조직화된 집합**이라고 한다.

### DBMS(Database Management System)

사용자에게 DB를 정의 또는 관리할 수 있는 기능을 제공하는 SW 시스템을 DBMS(Database Management System)라고 부른다.   
ex) PostgreSQL, MySQL, Oracle, ...

### Meta data

- DB를 정의하는 과정에서 부가적인 데이터가 발생하는데, 이 부가적인 데이터를 **meta data**라고 한다.
- DB를 정의하거나 기술하는 데이터, 즉 데이터를 설명하기 위한 데이터(**data about data**)라고도 부른다.
- 대표적으로 데이터 유형, 구조, 제약 조건, 보안, 저장, 인덱스, 사용자 그룹 등 데이터를 뒷받침하는 다양한 정보가 존재한다.    
  ex) 사진의 경우 해상도, 크기, 촬영일 등등

### DB system 동작 방식

<img width=40% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/e5dcce97-f66a-4d93-894d-ba313e818be3">

1. 사용자는 애플리케이션을 통해 원하는 데이터를 요청한다.
2. 요청이 들어오면, DBMS SW에서는 해당 쿼리가 무엇을 의미하는지 분석한다.
3. 요청으로 들어온 쿼리가 원하는 것을 파악하고나면 그 요청을 처리한다.
4. 요청을 처리하기 위해서 요청된 데이터의 메타 데이터를 확인한다.
5. 확인한 메타 데이터를 바탕으로 실제 요청 받은 데이터를 찾아서 반환한다.

---

## Data model

DB의 구조(데이터 유형, 관계, 제약 사항 등)를 기술하기 위해 사용되는 개념들이 모인 집합을 data model이라고 한다.   
DB 구조를 **추상화**해서 표현할 수 있는 수단을 제공하며, 추상화 수준과 DB 구조화 방식에 따라 여러 종류가 존재한다.

### Conceptual data model

일반 사용자들도 쉽게 이해할 수 있는 개념들로 이루어진 모델이다.   
그만큼 추상화 수준이 가장 높으며, 비즈니스 요구 사항을 추상화하여 기술할 때 사용한다.   
대표적으로 아래와 같은 **ERD**가 개념적 데이터 모델에 포함된다.

<img width=40% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/fa92eba7-3852-4540-8972-3bc9953a726b">

### Logical data model

이해하기 어렵지 않으면서도 디테일하게 DB를 구조화할 수 있는 개념들로 이루어진 모델이다.   
여기서 디테일하다는 것은 데이터가 컴퓨터에 저장될 때의 구조와 크게 다르지 않음을 의미한다.   
어느 정도 추상화가 되어 있기 때문에 특정 DBMS나 Storage에 종속되지 않는 수준에서 구조화가 가능하다.

MySQL과 Oracle 같은 **Relation Data Model**과 사용자 정의, 비정형 정보 타입을 지원하는 **Object Data Model**, 그리고 이 둘을 혼합한 **Object-Relation Data Model**(PostgreSQL)이 존재한다.

- Relation data model   
  <img width=30% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/4a9a4814-3759-4f82-a176-782f6ee6a539">

- Object-Relation data model   
  <img width=30% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/ddf35899-a624-491f-976c-d502f2d24040">

### Physical data model

컴퓨터에 데이터가 어떻게 파일 형태로 저장되는지를 기술할 수 있는 수단을 제공하는 모델이다.   
데이터 포맷, 순서, 그리고 access path 등이 여기에 포함된다.

<img width=40% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/39b91a76-bae2-41cb-b6cf-66e259f20760">

> #### access path란?
> 데이터 검색을 빠르게 하기 위한 구조체로, 대표적으로 index가 존재한다.

---

## Database schema & state

데이터 모델을 바탕으로 DB의 구조를 기술한 것을 **DB Schema**라고 하고, 특정 시점의 DB에 있는 데이터를 **DB State**(또는 **snapshot**)라고 한다.   
DB 스키마는 DB를 설계할 때 정해지며 한번 정해진 후에는 자주 변경되지 않는다(자주 변경되어서도 안 된다). 반면에 DB의 상태는 비교적 자주 변경된다.

### Three-Schema Architecture

Three-Schema Architecture는 DB 시스템을 구축하는 아키텍처 중 하나로, 사용자로부터 물리적인 DB를 분리시키는 것이 목적이다.

> ### 왜 사용자로부터 물리적인 DB를 분리시켜야 할까?
> 물리적인 DB 구조는 조금씩 변경될 수 있다. 따라서 물리적인 DB가 user application과 붙어 있다면 application에 영향을 미칠 수도 있다.   
> 예를 들어, DB의 구조를 변경할 때마다 애플리케이션을 지속적으로 수정하고 테스트해야 한다. 이 외에도 애플리케이션의 안정성과 성능 등 다양한 문제가 발생할 수 있다.   
> 이를 방지하고자 user application으로부터 물리적인 DB를 분리시키는 것이다.

Three-Schema Architecture에는 세 가지 레벨이 존재하는데, 각 레벨마다 별도의 스키마가 정의되어 있다. 이때 각각의 스키마는 단지 DB 구조를 표현만 하고, 실제 데이터가 존재하는 곳은 Internal level뿐이다.

<img width=50% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/bc50722e-3fe7-4e3b-8938-0bd2ea34483d">

#### External Schema
사용자가 직접 바라보는 스키마로, external views, user views라고도 불린다.   
Logical data model을 통해 특정 사용자들이 필요로 하는 데이터만 표현하고, 그외 알려줄 필요 없는 데이터는 숨긴다. → **추상화**   
이를 통해 사용자는 데이터가 어떻게 저장되어 있는지 신경쓰지 않고 데이터를 이해하고 사용할 수 있게 된다.

#### Conceptual Schema
Internal Schema를 한 번 추상화한 schema로, 물리적인 저장 구조에 관한 내용을 숨기는 대신 논리적으로 DB의 전체 구조를 기술한다.   
DB의 전체 구조를 기술하기 위해서 Logical data model을 사용하며, entity, data type, relationship, user operation, constraints에 집중한다.

#### Internal Schema
물리적인 저장 장치와 가까우며, Physical data model을 통해 물리적으로 데이터가 어떻게 저장되어 있는지 표현한다.   
예를 들어 data storage, data structure, access path 등 실체가 있는 내용을 기술한다.

<details>
  <summary> 각각의 스키마에 대해 더 쉽게 이해하고 싶다면? </summary>
    <div markdown="1">

DB는 개발자, DBA, 고객 등 다양한 분야의 사람들이 이용할 수 있다. 그런데 이 다양한 분야에 있는 사람들은 DB를 보는 관점이 모두 다르다.   
아래는 고객 데이터를 가지는 DB의 스키마를 나타낸 그림이다.

<img width=40% src="https://github.com/Hanjaemo/DB-Study/assets/110653660/a0a3a21b-e367-4ade-80eb-299115a57644">

데이터베이스에는 고객에 대한 다양한 데이터들이 담겨 있고, 개발자가 데이터를 명확하게 처리할 수 있도록 각각의 데이터에 메타 데이터를 나타낸다. 이것이 바로 **내부 스키마**다.

그런데 이 많은 양의 정보를 일반 사용자에게까지 보여줄 필요가 있을까? 그렇지 않다.   
따라서 **외부 스키마**를 통해 각각의 사용자마다 필요한 정보만 제공한다.

초기 데이터베이스의 구조는 이렇게 내부 스키마와 외부 스키마로만 구성되어 있었다.   
하지만 데이터베이스를 사용하는 부서가 증가하게 되면서, 데이터 관리의 복잡성이 증가하게 되었다.   
이를 해소하고자 데이터베이스의 전체적인 구조와 정책을 정의하는 **개념 스키마**를 사이에 둔 것이다.

  </div>
</details>


결국 Three-Schema Architecture의 목적은 안정적으로 DB 시스템을 운영하는 것이다.   
각 레벨을 독립시켜서 어느 레벨에서의 변화가 상위 레벨에 영향을 주지 않게 하기 위해 고안한 개념이다.   
예를 들어, Internal Schema가 변경되어도 Conceptual Schema를 변경할 필요 없고, 단지 두 Schema의 mapping만 변경하면 된다.   
여기서 **mapping**이란 각 레벨 사이에서 요청과 응답을 주고 받는 것을 의미한다.

> 대부분의 DBMS가 Three-Schema Architecture를 완벽하게 따르지는 않는다.   
> Internal level에서 Conceptual level로 미치는 영향을 막는 것까지는 괜찮지만, Conceptual level에서 External level로 미치는 영향을 막기란 쉽지 않기 때문이다.
