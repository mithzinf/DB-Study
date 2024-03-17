## SQL 이란?

- DDL + DML + VDL 기능을 합친 것 

### SQL에서 relation 이란?
-> Relational Data Model은 중복을 허용하지 않았지만, SQL은 중복을 허용한다.


### CHAR 대신 VARCHAR를 사용하는 이유

CHAR
* 최대 몇 개의 '문자를' 가지는 문자열을 저장할 지 저장.
* 만약에 저장되는 문자열의 길이가 최대 길이보다 작다면 SPACE로 채워져서 저장된다.
-> ex) CHAR(4) -> `'a    '`

VARCHAR
* 가변길이 문자열 타입으로, 최대 길이보다 적게 문자열이 저장되더라도 SPACE로 채워지지 않는다.
-> ex) VARCHAR(4) -> `'a'`

만약에 데이터를 `equals`를 사용하여 비교하였는데 공백이 존재한다면, 문제가 생길 수 있다. 이럴 때 `trim`을
사용하여 공백을 제거해줘야 하는데, `VARCHAR`를 사용하여 데이터를 저장하면 이런 번거로움을 거치지 않아도 된다.

참고로, `MySQL` 에선 `VARCHAR`를 쓸 때 시간적인 성능이 `CHAR`보다 안좋다는 이야기가 있어, 휴대폰 번호 같은
고정된 문자열을 `CHAR`를 쓸것을 권장한다.


### Key Constraints
#### PRIMARY KEY
* 중복된 값을 가질 수 없으며, `NULL` 값을 허용하지 않는다.

#### UNIQUE
* UNIQUE로 지정된 attribute는 중복된 값을 가질 수 없다. RDBMS에 따라서 `NULL` 허용이 가능.

#### NOT NULL
* 해당 attribute는 `NULL`값을 가질 수 없고, UNIQUE와 주로 같이 사용.

### Constraints
#### DEFAULT
* DEFAULT 값을 저장하고 싶을 때 사용

#### CHECK
* 값을 제한하고 싶을 때 사용

**ALTER TABLE**
-> 이미 서비스 중인 테이블의 schema를 변경하는 것이라면 백엔드에 영향이 없을지 충분히 검토를 해야함.