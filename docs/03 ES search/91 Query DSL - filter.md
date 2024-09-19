
## 1. bool

- bool (true, false) 로직을 사용하는 쿼리

### 종류

- must : bool must 절에 지정된 모든 쿼리가 일치하는 document 를 조회
- should : bool should 절에 지정된 모든 쿼리 중 하나라도 일치하는 document를 조회
- must_not : bool must_not 절에 지정된 모든 쿼리가 모두 일치하지 않는 document를 조회
- filter : must와 같이 filter 절에 지정된 모든 쿼리가 일치하는 document를 조회하지만, filter context에서 실행되기 때문에 score를 무시합니다.

bool 쿼리 내에 위의 각 절들을 조합해서 사용할 수도 있고,

또한 bool 절 내에 bool 쿼리를 작성할 수도 있다.