# nested-objects

es 에서 단일 문서를 만드록, 삭제하고, 업데이트하는 것이 원자적이라는 사실을 감안할 때, **밀접하게 관련된 엔티티를 동일한 문서 내에 저장하는 것**이 합리적이다.

예를 들어, 주문과 주문과 관련된 모든 라인은 하나의 문서에 저장되어야 하고,

블로그 게시물과 그에 따른 댓글은 하나의 문서에 함께 저장되어야 할 것이다.

이렇게 관련된 모든 컨텐츠가 동일한 문서에 있으므로 쿼리 시점에 몇 번의 쿼리를 요청할 필요가 없고, 

블로그 게시물과 댓글을 결합 머징할 필요가 없으므로 검색성과가 더 좋을 것이다.

### 문제 상황

그렇다면, 어떠한 경우에 nested-objects 를 활용하면 좋은지 아래 문제 상황을 통해 확인해보자.

~~~
PUT /my_index/blogpost/1
{
  "title": "Nest eggs",
  "body":  "Making your money work...",
  "tags":  [ "cash", "shares" ],
  "comments": [ 
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}
~~~

arrays of inner objects 에서 이러한 cross-object matching 의 이유는 json 문서가 다음과 같이 인덱스에서 간단한 key-value 형식으로 평면화되기 때문이다.

~~~
{
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ],
  "comments.name":    [ alice, john, smith, white ],
  "comments.comment": [ article, great, like, more, please, this ],
  "comments.age":     [ 28, 31 ],
  "comments.stars":   [ 4, 5 ],
  "comments.date":    [ 2014-09-01, 2014-10-22 ]
}
~~~

위를 보면 alice - 31 사이의 데이터 상관관계가 손실 된 것을 볼 수 있다.

Multilevel Objects 는 단일 객체를 저장하는데에는 유용하지만, 객체 배열을 저장하는데는 쓸모가 없다.

## 해결
- `"type": "nested"` 


~~~
PUT /my_index
{
  "mappings": {
    "blogpost": {
      "properties": {
        "comments": {
          "type": "nested", 
          "properties": {
            "name":    { "type": "string"  },
            "comment": { "type": "string"  },
            "age":     { "type": "short"   },
            "stars":   { "type": "short"   },
            "date":    { "type": "date"    }
          }
        }
      }
    }
  }
}
~~~

nested-object 문제를 해결하도록 설계된 구조이다.

필드를 type 대tls type comments 으로 매핑하면 중첩된 각 객체가 다음과 같이 숨겨진 별도 document 로 인덱싱된다.

~~~
{ 
  "comments.name":    [ john, smith ],
  "comments.comment": [ article, great ],
  "comments.age":     [ 28 ],
  "comments.stars":   [ 4 ],
  "comments.date":    [ 2014-09-01 ]
}
{ 
  "comments.name":    [ alice, white ],
  "comments.comment": [ like, more, please, this ],
  "comments.age":     [ 31 ],
  "comments.stars":   [ 5 ],
  "comments.date":    [ 2014-10-22 ]
}
{ 
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ]
}
~~~
- 첫번째 nested 객체
- 두번째 nested 객체
- root, parent 문서


:star:
**중첩된 각 객체를 별도로 인덱싱하면 객체 내의 필드간의 관계가 유지**된다.

그리고 동일한 객체 내에서 일치가 발생하는 경우에만 일치하는 쿼리를 실행할 수 있다.

그뿐만 아니라 중첩된 개체가 인ㄷ게싱되는 방식 덕분에 쿼리 시점에 중첩된 문서를 루트 문서에 결합하는 속도가 매우 빠르다.

추가 중첩 문서는 숨겨져 있으므로 직접 액세스할 수 없다. 중첩된 객체를 업데이트, 추가 또는 제거하려면 전체 문서를 다시 인덱싱해야 해야 한다.

검색 요청에서 반환된 결과가 중첩된 객체만이 아니라 전체 문서라는 점에 유의해야 한다.


### 조회 쿼리 예시

~~~
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "eggs" 
          }
        },
        {
          "nested": {
            "path": "comments", 
            "query": {
              "bool": {
                "must": [ 
                  {
                    "match": {
                      "comments.name": "john"
                    }
                  },
                  {
                    "match": {
                      "comments.age": 28
                    }
                  }
                ]
              }
            }
          }
        }
      ]
}}}
~~~