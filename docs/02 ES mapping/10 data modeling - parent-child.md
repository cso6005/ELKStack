# parent-child

다른 종류의 개체들 간의 관계를 모델링 하려고 할 때,

예를 들어, 해리포터 라는 영화 프렌차이즈가 있고, 그에 자식이라고 칭할 수 있는 그 프렌차이즈를 구성하는 시리즈 영화들 (불사조 기사단, 불의 잔...)이 있다고 하자.

es 는 이러한 데이터 구조를 parent-child 관계로 모델링 하는 능력을 가지고 있다.

### nested objects 와 비교
- 해당 구조는 nested-objects model 과 비슷하다. 둘 다 하나의 엔티티를 다른 엔티티와 연결할 수 있다.
- 차이점은 **nested-objects 의 경우 모든 엔티티가 동일한 문서 내**에 있는 반면,
- **parent-child 의 경우 완전히 별개의 문서**라는 것이다.
  
### parent-child 
- 이 구조를 가져가면 한 문서 유형을 다른 문서 유형과 일대다 관계로 연결할 수 있다. 즉, 한 부모가 여러 자식에게 연결된다. 
- es 는 parent-child에 대한 map을 유지한다.
  - 이 map 덕분에 쿼리 시간 조인이 빠르지만, parent-child 관계에서 **두 문서는 같은 샤드에 있어야** 한다는 제한이 있다.
  - map 은 doc values 에 저장되므로 메모리에서 최대한 많이 실행되는 경우 빠르게 실행되지만, map 이 매우 클 때는 확장가능하게 충분해야 한다.

### mappings

~~~
{
  "mappings": {
    "properties": {
      "film_to_franchise": {
        "type":"join",
        "relations": {"franchise":"file"}
      }
    }
  }
}
~~~

### query
- has_parent : 해당 franchise 를 pranet 로 둔 child 검색
~~~
{
  "query": {
    "has_parent": {
      "parent_type": "franchise",
      "query": {
        "match": {
          "title": "Star Wars"
        }
      }
    }
  }
}
~~~

- has_chlid : 해당 영화를 chlid 로 둔 franchise 검색
~~~
{
  "query": {
    "has_child": {
      "type": "film",
      "query": {
        "match": {
          "title": "The Force Awakens"
        }
      }
    }
  }
}
~~~

### 장점 및 쓰기 유용한 상황
- 자식 문서를 다시 인덱싱하지 않고도 부모 문서를 업데이트 할 수 있다.
- 자식 문서는 부모나 다른 자식 문서에 영향을 미치지 않고, 추가 변경 삭제할 수 있다.
  - 자식 문서의 수가 많고 자주 추가 또는 변경해야 할 때 유용
- 자식 문서는 검색 요청의 결과로서 반환 될 수 있다.

