### Stored Function?

사용자가 정의한 함수이며 DMBS에 저장되고 사용되는 함수이다. 


```mysql
delimeter $$
    
CREATE FUNCTION 'name' (
파라미터
) RETURNS 반환할 데이터타입
BEGIN
	수행할 쿼리
	RETURN 반환할 값
END
    
delimeter ;
```