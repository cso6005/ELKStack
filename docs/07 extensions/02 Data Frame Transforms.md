#

- 도구 상자에 있는  중ㅛ한 기능
- 고객 아이디나 아피 등의 특정한 것들로 데이털르 집계하는 메커니즘을 정의하게 해주고 요약된 2차 지수를 새로 만들어줌.

- 변환 기능이 집계 쿼리를 대체하는 것이 아니라, 제약들을 해겨해주는 것.

# why transform?
1. performance
	1. 복잡한 대시코드의 경우 캐시된 경우를 제외하고 집계 쿼리가 매번 재실행 되면서 성능 문제를 일으키기도 한다. -> 클러스터의 요청을 계산하고 연속적인 메모리를 처리할 수 있다. 
2. result limitations
	1. 성능 문제로 어느정도 제한을 두는 것.
	2. 반환된 버킷의 최대 수치나 집계의 순서나 필터 제한
3. dimensionality of data
	1. 높은 수준의 모니터링과 인사이틀 위한 데이터의 차원성. 

# how to transform?

- transfrom 하는 방법
	- kibana
	- 변환 api 생성

### 변환 api 생성
### 1. DEFINE
1. 식별자를 제공해야 함.
	1. 패스 변수로 평군 url 규칙을 따름
2. 원시 데이터가 입력될 '소스' 색인을 지정
3. 실제 변화 메커니즘 '피벗' 을 정의해야 함.
4. 변환된 데이터가 저장될 destination index 를 지정해야 함.
### 2. RUN
- '배치 변환' : 변환을 한 번 실행할 수 있는 옵션
- '연속 변환' : 일정을 정해놓고 실행
	- 일정대로 실행되는 연속 변환은 변환이 실행될 때마다 '증분 체크 포인트' ㅡ새성한다.
- '변환 api 미리보기'를 사용하여 변환을 실행하기 전에 쉽게 미리 볼 수 있음.
- '변환 api 시작하기'로 변환을 시작할 수 있음.

1. destination index 가 존재하지 않다면 이를 생성해야 함.  
2. 그리고 target maping 은 assurance validations 을 수행하고, 데이터의 부분 집합이나 전체 색인에 변환을 실행한다.
3. 이 프로세스는 checkpoint 를 만들 게 된다. 이는 변환이 처리되었다는 걸 표시한다.

### The Transformation Definition

1. pivot.group_by
- 하나 또는 그 이상의 집계를 정의
- terms 는 텍스트 필드에서 찾은 개별 단어를 기반으로 집계
- histogram 은 숫자 값의 간격을 기반으로 집계
- data histogram 은 날짜 와 시간 기반으로 집계

3. pivot.aggregations
- 데이터 버킷에서 계산하고 싶은 실질적인 집계를 정의
- 평균, 최소, 최대, 합계, value 카운트, 맞춤형 메트릭 등을 계산할 수 있음.

### ex
1. 변환 정의
~~~
curl --location --request POST 'http://localhost:9200/_transform/_preview' \
--data-raw '{
   "source": {
       "index": "nginx"
   },
   "pivot": {
       "group_by": {
           "ip": {
               "terms": {
                   "field": "remote_ip"
               }
           }
       },
       "aggregations": {
           "bytes.avg": {
               "avg": {
                   "field": "bytes"
               }
           },
           "bytes.sum": {
               "sum": {
                   "field": "bytes"
               }
           },
           "requests.total": {
               "value_count": {
                   "field": "_id"
               }
           },
           "requests.last": {
               "scripted_metric": {
                   "init_script": "state.timestamp = 0L; state.date = '\'''\''",
                   "map_script": "def doc_date = doc['\''time'\''].getValue().toInstant().toEpochMilli();if (doc_date > state.timestamp){state.timestamp = doc_date;state.date = doc['\''time'\''].getValue();}",
                   "combine_script": "return state",
                   "reduce_script": "def date = '\'''\'';def timestamp = 0L;for (s in states) {if (s.timestamp > (timestamp)){timestamp = s.timestamp; date = s.date;}} return date"
               }
           },
           "requests.first": {
               "scripted_metric": {
                   "init_script": "state.timestamp = 1609455599000L; state.date = '\'''\''",
                   "map_script": "def doc_date = doc['\''time'\''].getValue().toInstant().toEpochMilli();if (doc_date < state.timestamp){state.timestamp = doc_date;state.date = doc['\''time'\''].getValue();}",
                   "combine_script": "return state",
                   "reduce_script": "def date = '\'''\'';def timestamp = 0L;for (s in states) {if (s.timestamp < (timestamp)){timestamp = s.timestamp; date = s.date;}} return date"
               }
           }
       }
   }, 
   "description": "Byttes and request dates doverview for remote_ip",
   "dest": {
	   "index": "nginx_transformed"
   }
}'
~~~

2. 변환 시작 - start transform API
~~~
curl --request POST 'http://localhost:9200/_transform/nginx_transform/_start}

~~~
 