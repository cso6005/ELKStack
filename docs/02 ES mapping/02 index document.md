
# 1. index

- 인덱스 관련 기본 명령어
```sql
# 인덱스 생성
PUT item

# 생성된 인덱스 조회
GET item

# 인덱스 삭제
DELETE item
```

- mapping put 
```sql
curl -XPUT 127.0.0.1:9200/movies -d '
{
"mappings: {
"properties": {
"year":{
"type":"date"
}
}
}
}'

# mappings 확인
curl -XPOST 127.0.0.1:9200/movies/_mappings
```

# 2. document
- 반드시 하나의 인덱스에 포함돼야 한다.

### document indexing
- 엘라스틱서치에서 도큐먼트를 인덱스에 포함시키는 것을 인덱싱이라고 한다.

아래 요청하면 존재하지 않았던 item 이라는 인덱스를 생성하면서 동시에 item 인덱스에 도큐먼트를 인덱싱한다.

```sql
# document indexing
PUT item/_doc/1
{
  "item_name":"참이슬",
  "item_price":4500,
  "item_category":"소주"
}
```

- item : 인덱스 이름
- _doc : 엔드포인트 구분을 위한 예약어
- 숫자 1 : 인덱싱할 도큐먼트의 고유 아이디
- 도큐먼트에는 item_name, item_price, item_category 라는 필드가 있고, 각 필드에는 값이 있다.

### _doc 의 의미
- _doc 는 이전 버전에 잔여물이다. 이전 버전에서는 유형이라는 것이 있었으나, 최신 버전에서는 없으므로 _doc를 기본값으로 가져간다.

위에 만든 인덱스의 설정값을 확인해보자.

```sql
# item 인덱스의 설정값 등을 확인함.
GET item
```

결과는 아래와 같다.

```sql
{
  "item" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "item_category" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "item_name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "item_price" : {
          "type" : "long"
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "item",
        "creation_date" : "1699269590564",
        "number_of_replicas" : "1",
        "uuid" : "nvtuCyU2Ruic_jb26MSATg",
        "version" : {
          "created" : "7100099"
        }
      }
    }
  }
}
```

여기서 주목해야 할 부분은 mappings 부분이다. 

### dynamic mapping

위 결과를 통해, 우리가 데이터 타입을 지정하지 않아도 엘라스틱 서치는 도큐먼트의 필드와 값을 보고 자동으로 지정하는 것을 알 수 있다. 이러한 기능을 다이내믹 매핑이라고 한다.

### 새로운 필드 추가

새로운 필드가 추가된 도큐먼트를 인덱싱이 가능하다.

기존의 필드를 사용하지 않더라도 문제 없이 인덱싱이 된다.

```sql
# 새로운 필드가 추가된 도큐먼트를 인덱싱 가능
# 매핑이 변경된 것을 알 수 있다.
PUT item/_doc/2
{
  "item_name":"처음처럼",
  "brand":"롯데칠성음료"
}
```

### 데이터 타입을 잘못 입력한 도큐먼트를 인덱싱 시도할 시,

RDB에서는 오류가 발생했겠지만 스키마에서는 유연하게 대응하는 엘라스틱서치는 타입을 변환해 저장한다.

```sql
PUT item/_doc/3
{
  "item_name":"진로",
  "item_price": "5000"
}

# 도큐먼트 읽기
GET item/_doc/3
```

```sql
{
  "_index" : "item",
  "_type" : "_doc",
  "_id" : "3",
  "_version" : 1,
  "_seq_no" : 4,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "item_name" : "진로",
    "item_price" : "5000"
  }
}
```

엘라스틱 서치는 혹시 모를 사용자 입력 실수를 고려해 자동으로 데이터 형변환을 진행한다.

만약 형변환이 되지 않는 타입일 경우, **mapper_parsing_exception** 이 발생한다.

- 대표적인 형변환 규칙
    - 숫자 필드에 문자열이 입력되면 숫자로 변환을 시도 “age”: “10” → 10
    - 정수 필드에 소수가 입력되면 소수점 아래 자리를 무시 “age”: 10.0 → 10
    

