# 0. 종류
>1. 리프 쿼리
>    특정 필드에서 용어를 찾는 쿼리
> 
>    match, term, range …
    
>2. 복합 쿼리
>    
>    쿼리를 조합해 사용되는 쿼리
>    
>    bool query …
    
<br>

## 1. full text query

- full text query는 full text 검색을 하기 위해 사용되며, 전문 검색을 할 필드는 인덱스 매핑 시, text type으로 매핑해야 한다.
- 

## 2. term level query

- term text query 는 정확히 일치하는 용어를 찾기 위해 사용되며, 인덱스 매핑 시 필드를 keyword 타입으로 매핑해야 한다.

<br>

# 1. Full Text

## full text 검색 동작 과정

1. product 인덱스를 생성하고 해당 필드를 갖는 도큐먼트를 인덱싱한다.
    
    
    ```java
    PUT product/_doc/280a8a4d-a27f-4d01-b081-2a003cc4c098
    {
      "name":"호랑이 막걸리"
    }
    ```
    
2. name 필드는 텍스트 타입으로 매핑된다. (다이나믹은 text 로? )
3. 텍스트 타입으로 매핑된 문자열은 분서긱에 의해 토큰으로 분리되는데, 아래와 같이 **토큰화** 된다.
    
    
    | 번호 | 단어 | 문서 |
    | --- | --- | --- |
    | 1 | 호랑이 | 1 |
    | 2 | 막걸리 | 1 |

1. 전문 쿼리인 match query 를 사용해 전문 검색을 아래와 같이 한다.

```java
PUT product/_search
{
	"query: {
		"match": {
			"name": "감자 막걸리"	
		}
	}
}
```

전문 쿼리를 사용하면 검색어인 감자 막걸리도 분석기에 의해 토큰으로 분리되어,
[감자, 막걸리]로 토큰화된다. 

**그리고 토큰화된 검색와 토큰화된 도큐먼트 용어들 [호랑이, 막걸리] 이 매칭되어 스코어를 계산하고 검색을 한다.**

전문 쿼리는 일반적으로 블로그처럼 텍스트가 많은 필드에서 특정 용어를 검색할 때 사용된다. 

## 주 사용처

- 텍스트 타입 필드에서 검색어를 찾을 때 사용
    
    분석기가 텍스트를 토큰화해서 전문 검색이 가능하다.
    

## 전문 쿼리 종류

대표적으로 아래와 같다.

## 1. match query

- 전문 쿼리의 가장 기본이 되는 쿼리
- 전체 텍스트 중에서 특정 용어나 용어들을 검색할 때 사용
- 매치 쿼리를 사용하기 위해서는 검색하고 싶은 필드를 알아야 한다.

### ex) 하나의 용어를 검색하는 매치 쿼리

```java
GET product/_search
{
  "_source": "name",
  "query": {
    "match": {
      "name": "Soju"
    }
  }
}
```

전문 쿼리의 겨웅, 검색어도 토큰화되기 때문에 검색어 Soju는 soju로 토큰화된다. 분석기에 따라 다르지만 일반적인 분석기를 사용했다면 대문자를 소문자로 변경하기 때문이다. 

### ex) 복수 개의 용어를 검색하는 매치 쿼리

```java
GET product/_search
{
  "_source": "name",
  "query": {
    "match": {
      "name": "감자 막걸리"
    }
  }
}
```

name 필드에서 두용어를 검색하는 매치 쿼리 용청이다.

검색어인 감자 막걸리는 분석기에 의해 [감자, 막걸리] 로 토큰화된다.

**매치 쿼리에서 용어들의 공백은 OR 로 인식한다.**

즉 name 필드에 감자, 막걸리가 하나라도 포함된 도큐먼트가 있다면 매칭되었다고 판단한다.

```java
"hits" : {
    "total" : {
      "value" : 7,
      "relation" : "eq"
    },
    "max_score" : 2.5700645,
    "hits" : [
      {
        "_index" : "product",
        "_type" : "_doc",
        "_id" : "280a8a4d-a27f-4d01-b031-2a003cc4c035",
        "_score" : 2.5700645,
        "_source" : {
          "name" : "감자 막걸리"
        }
      },
      {
        "_index" : "product",
        "_type" : "_doc",
        "_id" : "280a8a4d-a27f-4d01-b031-2a003cc4c888",
        "_score" : 1.5308115,
        "_source" : {
          "name" : "감자 소주"
        }
      },
      {
        "_index" : "product",
        "_type" : "_doc",
        "_id" : "280a8a4d-a27f-4d01-b031-2a003cc4c099",
        "_score" : 1.3042111,
        "_source" : {
          "name" : "옥수수 막걸리"
        }
      },
```

검색어였던 감자, 막걸리 둘 중 하나라도 포함된 모든 도큐먼트가 매칭되는 것을 알 수 있다.

### operator 를 and 로 설정한 매치 검색

```java
GET product/_search
{
  "_source": "name",
  "query": {
    "match": {
      "name": {
        "query": "감자 막걸리",
        "operator": "and"
      }
    }
  }
}
```

```java
"hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.5700645,
    "hits" : [
      {
        "_index" : "product",
        "_type" : "_doc",
        "_id" : "280a8a4d-a27f-4d01-b031-2a003cc4c035",
        "_score" : 2.5700645,
        "_source" : {
          "name" : "감자 막걸리"
        }
      },
      {
        "_index" : "product",
        "_type" : "_doc",
        "_id" : "280a8a4d-a27f-4d01-b081-2a003cc4c777",
        "_score" : 1.8762803,
        "_source" : {
          "name" : "막걸리 감자"
        }
      }
    ]
  }
```

도큐먼트에 감자, 막걸리가 모두 포함된 도큐먼트만 찾는다. 

- 순서와 상관없이 감자, 막걸리가 모두 들어가 있다면, 해당 응답을 내본다.
- **operator 파라미터**는 기본값이 or이기 때문에 operator 를 명시적으로 지정하지 않으면 or로 인식된다.

### _source

- name 필드만 보여달라는 요청

## 2. match phrase query

- 매치 쿼리와 마찬가지로 전문 쿼리의 한 종류인 match phrase query 는 phrase 를 검색할 때 사용한다.
- 구는 동사가 아닌  2개 이상의 단어가 연결되어 만들어지는 단어다.
    
    빨간색 바지, 65인치 텔레비전 같이 여러 단어가 모여서 뜻을 이루는 단어다
    
- 매치 프레이즈 쿼리는 검색 시, 많은 리소스를 요구하기 때문에 자주 사용하는 것은 좋지 않다.
    
    
- **검색하려는 모든 단어가 있어야 하며, 순서가 중요가 중요하다.**
    
    빨간색 바지를 검색한다는 것은 바지 빨간색을 검색하려는 의도가 아니다. 이것이 match query와 다른 점이다.
    
    ⇒  [감자,막걸리] 로 토큰화되는 것까지는 매치 쿼리와 같지만, 매치 프레이즈 쿼리는 검색어에 사용된 용어들이 모두 포함되면서 용어의 순서까지 맞아야 한다. 
    
- 용어의 순서가 바뀐 막걸리 감자나, 중간에 다른 단어가 포함된 감자 최고 막걸리는 매칭되지 않는다.

```java
GET product/_search
{
  "_source": "name",
  "query": {
    "match_phrase": {
      "name": "감자 막걸리"
    }
  }
}
```

## 3. multi-match query

```java
GET product/_search
{
  "query": {
    "multi_match": {
      "query": "감자",
      "fields": [
        "name",
        "description"
        ]
    }
  }
}
```

- 2개의 필드에 대해 감자라는 용어로 매치 쿼리를 하고 3개의 필드에서 **개별 스코어를 구한 다음에 그중 가장 큰 값을 대표 스코어로 구한다.**
- 대표 스코어 선택 방식은 사용자가 결정할 수 있지만, 특별한 설정을 하지 않으면, 기본으로 가장 큰 스코어를 대표 스코어로 사용한다.
- explain 파리미터를 true 로 설정하면 개별 필드의 스코어가 어떻게 계산되었는지 알 수 있고, 대표 스코어가 어떻게 선정되엇는지도 알 수 있다.

```java
GET product/_search
{
  "query": {
    "multi_match": {
      "query": "감자 막걸리",
      "fields": [
        "name",
        "description",
        "rawMaterial.text"
        ]
    }
  }
  , "explain": true
}
```

- 검색하려는 필드가 너무 많을 때는 필드명에 * 같은 와일드카드를 사용해 이름이 유사한 복수의 필드를 선택할 수 있다.

```java
GET product/_search
{
  "query": {
    "multi_match": {
      "query": "감자 막걸리",
      "fields": [
        "product*name"
        ]
    }
  }
  , "explain": true
}
```

### 필드에 가중치 두기 : Boosting

- 여러 개의 필드 중 특정 필드에 가중치를 두는 방법

- 특정 필드의 스코어 값을 n배 gownsm sgyrhk
    
    name 에서 얻은 스코어가 descprition 보다 2배 더 높게 책정된다.
    

```java
GET product/_search
{
  "query": {
    "multi_match": {
      "query": "복순도가",
      "fields": [
        "name^2",
        "description"
        ]
    }
  }
}
```

- 대표 스코어는 각각의 필드에서 얻은 스코어 중에서 가장 큰 스코어로 정한다고 했는데, 이럴 경우 최종 대표 스코어는 name에서 얻은 결과를 채택할 확률이 높아진다.

## 4. query string query

## 5. 그외

### 1. match_all

특별한 조건없이 지정된 index의 모든 document를 검색하는 방법

```jsx
GET _search
{
"query": {
"match_all": {}
}
}
```

# 2. Term Level

## Term Level 검색 동작 과정

1. product 인덱스를 생성하고 category 필드를 갖는 도큐먼트를 인덱싱 한다. category 필드는 키워드 타입으로 매핑하는데, 키워드 타입은 인덱싱 과정에서 분석기를 사용하지 않는다. 
    
    ```java
    PUT product/_doc/1
    {
    	"category":"Tech"
    }
    ```
    
    | 번호 | 단어 | 문서 |
    | --- | --- | --- |
    | 1 | Tech | 1 |
    
2. 검색은 Term 쿼리를 사용하는데, 검색어 tech는 분석기를 거치지 않고 그대로 사용한다.
    
    ```java
    GET product/_search
    {
    	"query: {
    		"match": {
    			"category": "tech"	
    		}
    }
    ```
    

1. 이렇게 분석되지 않은 검색어(tech)와 분석되지 않은 도큐먼트 용어 (Tech)를 매칭하는데, 위 상황의 경우, Tech와 tech의 대소문자 차이로 매칭에 실패한다.
    
    term level query 는 full text query 와 달리 정확한  용어르 ㄹ검색할 때 사용된다.
    

## Term level Query 종류

대표적으로

### 1. term query

### 2. terms query

### 3. fuzzy query

## 주 사용처

- 일반적으로 숫자, 날짜, 키워드, 숫자형, 범위, 범주형 데이터를 정확히 일치하는 도큐먼트들을 검색할 때. 사용되며 관계형 데이터 베이스의 WHERE 절과 비슷한 역할을 한다.