# partial matching

### prefix queries on strings
- 접두사 쿼리로, 제공한 접두사를 포함하는 문자열을 검색 결과로 나타내고 싶을 때 사용
- 이는 역색인이 저장되고 정렬되는 방법에 따라 달라지는데, es 는 이 작업을 효율적으로 굉장히 빠르게 처리한다.
- 문자열 타입만 가능

- 201로 시작하는 문자열
~~~
{
    "query": {
        "prefix": {"year":"201"}
    }
}
~~~

### 주의
- 해당 쿼리는 비용이 많이 드는 쿼리에 속한다. 그렇기에 아래 파라미터로 조정해야 한다.
  - search.allow_expensive_queries = true 로 활성화해야 한다. (default - false)
  - 이때, [index_prefixes](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-prefixes.html) 가 활성화된 경우, 최적화된 커리가 빌드되어, 이 파라미터가 false 여도 실행될 것이다.


### wildcard queries
- 문자열 에서 * 기호를 사용하여 와일드 카드를 쓴다.
- *는 문자열 어느 곳에서든 사용할 수 있고 하나 또는 그 이상의 문자를 나타낸다.
- 문자열 타입만 가능

- 1로 시작하는 모든 결과
~~~
{
    "query": {
        "wildcard": {"year":"1*"}
    }
}
~~~

- 1903, 1913, 1923 ...
~~~
{
    "query": {
        "wildcard": {"year":"19*3"}
    }
}
~~~



### regex match


