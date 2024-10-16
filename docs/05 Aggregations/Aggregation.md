### aggregation buckets metrics pipelines matrix
- es 클러스터를 사용해 색인 내의 구조 및 숫자 데이터를 효율적으로 분석할 수 있다.
- 해당 데이터의 metrics 를 매우 쉽고 빠르게 자동 계산 할 수 있다.
- 또한, 데이터를 매우 빠르고 효율적으로 버킷할 수 있다.
- 파이프라인 기능을 통해 더 복잡한 작업도 수행할 수 있다.
- 행렬 처리도 가능 즉, 데이터가 2차원 이상일 때도 가능하다.

### 장점
- hadoop, spark 는 여전히 많은 오버헤드가 남아있고, es에 비해 오래 걸림. 
- es는 검색과 집계를 결밯해 분석하려는 항목을 줄이고 더 복잡한 분석이 가능하다.


~~~
{curl -XGET '127.0.0.1:9200/ratings/_search?size=0&pretty}
~~~


**{size=0}**
쿼리에서 원하는 결과 이외에는 반환되지 않도록 하는 것

검색 자체에 관한 결과가 아닌 집계 결과만 확인하려고 하기때문에

사이즈를 0으로 설정하여 검색 결과 리스트를 0 으로 설정한다. 검색 결과를 아예 받지 않겠다는 뜻


### histogram
- 이 구문은 데이터에서 쉽고빠르게 그래프를 그릴 수 있게 데이터를 응답받음.
  
- 구간 별 총계를 합산한 json 응답 버킷을 받음
~~~
{
    "aggs": {
        "whole_ratings" : {
            "histogram": {
                "field": "rating",
                "interval":10
            }

        }

    }
}
~~~

### time series
- 흔히 로그 처리가 시계열 데이터 분석의 대표적인 예이다.

- 1시간 별, 문서의 수 doc_count를 응답한다. 즉, 1시간에 로그가 얼마나 찍혔는지, 알 수 있다.
~~~
{
    "aggs": {
        "timestamp" : {
            "date_histogram": {
                "field": "@timestamp",
                "interval":hour
            }

        }

    }
}
~~~

### nested aggregations
- 집계 결과를 또 집계

- 상위 : 각 영화 (title 제목 별)마다 평점을 집계. 각 영화에 대한 모든 평점을 취합한 상위 레벨에서 버킷
- 하위 : 각 영화의 모든 평점을 가지고, 평균을 집계
~~~
{
    "query" : {
        "match_phrase": {
            "title" : "Star wars"
        }
    },
    "aggs" : {
        "titles" : {
            "terms" : {
                "field" : "title.raw"
            },
            "aggs" : {
                "avg_rating": {
                    "avg" : {
                        "field" : "rating"
                    }
                }
            }
        }
    }
}

~~~

### text 필드를 집계할 때의 문제 해결
1. fielddata 
   1. text 필드는 기본적으로 집계 정렬, 스크립팅할 수 없음. 기본적으로 비활성화되어 있어 **fielddata=true** 를 설정해준다.
   2. 안 하면 에러 떨어짐.
2. text 필드를 집계에 사용하기 위한 서브 필드를 keyword만들어 준다.
   1. 분석되어 토큰화된 text type 필드를 집계로 쓸 순 없을 것이다.

~~~
{
    "mappings" : {
        "properties" : {
            "title" : {
                "type" : "keyword",
                "fielddata" : true,
                "fields" : {
                    "raw" : {
                        "type" : "keyword"
                    }
                }
            }
        }
    }
}
~~~   

- fielddata
  - 필드 데이터를 메모리에 로드하면 상당한 메모리를 소모할 수 도 있음.