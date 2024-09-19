# 1. 명령 방법
1. curl 명령어
2. HTTP REST API, 인터페이스
   1. 실제 현업에서는 curl 이 아닌 해당 방법을 많이 쓴다.

### curl
- Pretty
  - 보기 어려운 cmd 에서 탭과 줄 바꿈을 사용해 결과를 읽기 좋은 형식으로

```jsx
curl -XGET 127.0.0.1:9200/movies/_search?pretty
```

- 요청의 본문

    → -d 와 작은 따옴표를 입력하고 발행하려는 쿼리와 JSON 형식을 입력

```jsx
curl -XPUT 127.0.0.1:9200/movies/_docs/12432 -d '
{
"genre": ["IMAX", "Sci-Fi"],
"title": "Interstellar",
"year": 2024
}'
```


# 2. document 읽기
### 1. document id 를 이용해 조회하는 방법 GET

# 도큐먼트 읽기

```sql
GET item/_doc/3
```

### 2. query DSL 을 이용한 SEARCH
- search 라는 DSL 쿼리를 통해서 해당 인덱스의 모든 도큐먼트에 대해 불러올 수 있다.

```sql
GET item/_search
```

```sql
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "item",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "item_name" : "참이슬",
          "item_price" : 4500,
          "item_category" : "소주"
        }
      },
      {
        "_index" : "item",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "item_name" : "처음처럼",
          "brand" : "롯데칠성음료"
        }
      },
      {
        "_index" : "item",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "item_name" : "진로",
          "item_price" : "5000"
        }
      }
    ]
  }
}
```

# 3. document 삽입
### 1. PUT 인덱싱 
```sql
# document indexing
PUT item/_doc/1
{
  "item_name":"참이슬",
  "item_price":4500,
  "item_category":"소주"
}
```

:information_desk_person: 04 index document 필기 참고

# 4. document 수정
이미 색인화된 문서를 어떻게든 변경하거나 업데이트 하는 방법


### 1. update 에서 주의할 점
ES 문서는 불변이기 때문에 일단 작성된 문서는 실제로 변경할 수 없다.  

### 2. **해결법**

ES 는 입력한 모든 문서에  자동으로 _version 필드를 생성한다. 

따라서 문서를 업데이트하기 위해 증가한 _version 번호로 문서의 새 복사본을 작성한다.

👍 **중요한 것은 문서의 고유ID와 버전을 함께 사용하는 것이다.** 

지정된 문서의 여러 버전을 가질 수 있으며 ES는 자동으로 사용자가 작성한 최신 버전을 사용한다. 

따라서 ES에서 업데이트 요청을 하면 **완전히 새로운 문서가 생성되고, 새 문서는 증가한 버전 번호로 작성된 후 이전 문서는 삭제로 표시**된다.

ES 는 나중에 필요할 때 마다 정리 작업을 수행. 그러면 예전 버전이 사라지게 된다.


### 3. PUT 인덱싱 
- document 하는 indexing 하는 과정에서 같은 document id 로 한다면, 덮어쓰기
- 이미 가지고 있는 문서에 대해서, 모든 필드를 포함하여 전체 문서를 다시 지정하는 것.
- 업데이트 할 때 마다 **_version이 증가**하는 것을 알 수 있다.
  - 이전 버전은 추후 es 가 정리한다.


```sql
PUT item/_doc/1
{
  "item_name":"참이슬 후레쉬", 
  "item_price":4500,
  "item_category":"소주"
}
```

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/_doc
/109487?pretty -d '
> {
> "genres":["IMAX"],
> "title":"Insterstellar foo",
> "year":2014
> }'
```

```bash
{"_index":"movies"
   ,"_type":"_doc"
   ,"_id":"109487"
   ,"_version":3
   ,"result":"updated"
   ,"_shards":{
      "total":2
      ,"successful":1
      ,"failed":0}
   ,"_seq_no":6
   ,"_primary_term":1}
```

❗**⇒ 문서 자체는 불변이다. 하지만 최신 버전이 무엇인지 추적할 수 있도록 _version 필드를 사용하는 것.**


### 4. update API
- POST
- _update 라는 엔드 포인트를 추가
- 특정 필드의 값만 업데이트 할 수 있다.
- 기존 기록을 가져와 하나 이상의 필드를 업데이트 한다는 뜻
- **이전 기존 버전에 있는 다른 필드를 복사해오고, 새 버전 번호를 가진 새 복사본이 생성**된다.
- 그러나 es document update 작업은 비용이 많이 들기 때문에 권장하지 않는다. 보통, 개별 document 를 수정할 일이 많은 경우, 엘라스틱 서치가 아닌 다른 데이터베이스를 이용하는 것이 좋다.

```sql
POST item/_update/1
{
  "doc": {
    "item_name": "참이슬"
  }
}
```

```bash
curl -XPOST 127.0.0.1:9200/movies/_doc/109487/_update -d '
{
"doc" : {
"title":"Interstellar"
}
}'
```

# 5. document 삭제
- 해당 인덱스의 삭제할 ID 를 지정하여, 삭제
- 도큐먼트 수정과 마찬가지로 개별 도큐먼트 삭제 또한 비용이 많이 들어가는 작업인 만큼 사용 시에 주의하자.

```sql
DELETE item/_doc/1
```

```bash
curl -XDELETE 127.0.0.1:9200/movies/_doc/58559
```

