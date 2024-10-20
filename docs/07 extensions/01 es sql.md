# es sql
1. 쿼리의 신택스가 검증되는 시점에 SQL 쿼리를 내부의 추상적인 신택스 트리나 AST로 보내는 것으로 시작한다.
2. 그러면 분석기는 내재된 색인으로 우리가 사용하려는 표와 열 그리고 기능들과 맞춰 쿼리를 실행할 논리적인 계획을 만든다.
3. 쿼리를 최적화하는 쿼리 플래너에 보내져 쓸모없는 작업 과정을 제거하게 한다.
4. 그리고 마지막으로 쿼리 실행기에 의해 작동되는 물리적인 계획이 만들어지고 
5. 실행기는 클라이언트에게 결과를 준다.


- es xpack 에서 sql query 지원
~~~
/_xpack/sql/?format=txt -d ' 
{
    "query" : "select * from movies"
}'
~~~ 

- sql 을 dsl 로 변환
~~~
/_xpack/sql/translate?
~~~ 

### es 명령행 인터페이스 sql 프롬프터
elasticsearch-sql-cli
