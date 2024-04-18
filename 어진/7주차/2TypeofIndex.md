# 2가지 경우의 인덱스  
1. Clustered Index  
2. Non-Clustered Index  
 

---


1. Clustered Index

- 정의 : 테이블의 데이터를 지정한 컬럼에 대해 물리적으로 데이터를 재배열함
    - 비유 : 책의 특정 페이지를 이미 알고 있어서, 바로 해당 페이지를 펼치는 것과 같다

- 특징
  - 데이터가 테이블에 삽입되는 순서에 상관없이 Index로 생성되어 있는 컬럼을 기준으로 정렬되어 삽입됨(물리적으로 재배열)
    - 데이터 삽입, 수정, 삭제시 테이블의 데이터를 정렬
  - 테이블당 1개만 허용
    - So, 테이블에서 Index를 걸면 가장 효율적일거 같은 컬럼을 Clustered Index로 지정
  - PK 설정 시, 해당 컬럼에 대해 자동으로 Clustered Index 생성
  - 인덱스의 리프 페이지가 곧 데이터! 즉, 테이블 자체가 인덱스이기에 따로 인덱스 페이지를 생성하지 x
  - 속도 성능은 우수하나, 데이터 추가/수정/삭제시 성능이 떨어짐 (비교 : Non-Clustered Index)

- 주의점
    - 사용자가 많은 시간에는 함부로 Clustered Index를 추가하면 안됨
      - 이유 : 테이블에 데이터가 많이 저장된 상태에서 ALTER를 통해 Clustered Index를 추가한다면, 많은 데이터를 정렬해야 해서 많은 리소스를 차지하게 되기 때문



        
- 방식

  ![image](https://github.com/mithzinf/DB-Study/assets/124668883/fb89cc8b-a0e7-48e3-a652-8a8d1e9cf4a7)  

  1. Clustered Index를 구성하기 위해, 레코드를 해당 컬럼으로 정렬한 후 > 루트 페이지 생성하게 됨  
  2. Clustered Index는 루트 페이지 + 리프 페이지로 구성 & 리프페이지는 **데이터 그 자체** & **Index 자체에 데이터가 포함됨**  
  3. Index Page를 키값과 데이터 페이지 번호로 구성하고, 검색하고자하는 데이터의 키 값으로 페이지 번호를 검색하여 데이터를 찾음  






--




2. Non - Clustered Index


- 정의 : 물리적으로 데이터를 배열하지 않은 상태로, 데이터 페이지가 구성됨  
    - 비유 : 책으로 비유하자면, 목차에서 찾고자 하는 내용의 페이지를 찾고난 뒤 해당 페이지로 이동하는 것  
    - 테이블의 데이터는 그대로 두고, 지정된 컬럼에 대해 정렬시킨 인덱스를 만드는 것뿐  


- 특징
  - 레코드는 정렬 X & 인덱스 페이지만 정렬  
  - 한 테이블에 여러개의 Non-Clustered Index 생성 가능  
  - Unique 제약조건 설정 시, 해당 컬럼에 대해 자동으로 Non- Clustered Index 생성  
  - 인덱스를 생성할 때, 데이터 페이지는 그대로 둔 상태에서 별도의 인덱스 페이지가 따로 생성되는 것이라 추가 용량 필요함  
  - 속도 성능은 떨어지나, 데이터 추가/수정/삭제시 성능은 뛰어남 (비교 : Clustered Index)

- 주의점
    - 한 테이블의 여러 개의 Non-Clustered Index 존재 가능하지만, 함부로 남용할 시 오히려 시스템 성능을 떨어뜨릴 수 있다
  



        
- 방식


  ![image](https://github.com/mithzinf/DB-Study/assets/124668883/125569c5-6b52-418c-a96d-c4c4938e90a1)  


  1. Non-Clustered Index는 데이터 페이지 건들지 않고, 별도의 장소에 인덱스 페이지를 생성함  
    1-1. 인덱스 페이지 : 키값(정렬하여 인덱스 페이지 구성) + RID로 구성     
  2. 인덱스 페이지 中 리프 페이지에 **INDEX로 구성한 열을 정렬**한 후 **위치 포인터(RID)**를 생성  
  3. Non-Clustered Index에서 인덱스 자체의 *리프 페이지*는 데이터가 아니라 **데이터가 위치하는 포인터(RID)**  
    3-1. RID는 '파일그룹번호-데이터페이지번호-데이터페이지오프셋'으로 구성되는 포인팅 정보로 알고 있음 됨  
  4. 리프 페이지(중간 레벨 인덱스 페이지)들을 생성 > 이 리프 페이지를 찾기 위한 루트 페이지(루트 레벨 인덱스 페이지)를 생성  
  5. 검색하고자하는 데이터의 키 값을 루트 페이지에서 비교 > 비교 후, 리프 페이지 번호를 찾고, 리프 페이지에서 RID 정보로 실제 데이터의 위치로 이동  





---

##### 부록 

- Clustered Index 와 Non-Clustered Index 한눈에 보기  
  ![image](https://github.com/mithzinf/DB-Study/assets/124668883/5ec6c7e7-dda9-4726-9434-243447c4ef22)

- 참고 사이트
  https://velog.io/@gillog/SQL-Clustered-Index-Non-Clustered-Index#clustered-index      
  https://choiblack.tistory.com/53
  
