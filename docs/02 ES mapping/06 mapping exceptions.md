
# 1. mapping

매핑은 기본적으로 두 부분으로 이뤄져 있음.

1. process
   - json documents가 색인에 저장되는 방법을 정의
2. result
    - mapping process 과정에서 발생하는 metadata 

### mapping process

1. Explicit Mapping 명시적 매핑
- 변수와 함께 저장할 필드 및 필드 유형을 정의

2. Dynamic Mapping
- es 가 자동으로 적절한 데이터 유형을 확인하고 그에 따라 매핑을 업데이트

### mapping result

- mapping process 의 결과는 **필드와 데이터 유형을 통해 색인화할 수 있는 항목을 정의**하고,
- **관련 변수에 의해 색인화가 수행되는 방식도 정의**

# 2. mapping 에서 발생할 수 있는 문제

## 1. 명시적 매핑에서 필드가 일치 하지 않을 때 mapper_parsing_exception
- 단일 해결책은 없음... 

<br>

### 해결 1 ignore_malformed
- **ignore_malformed** 매핑 변수를 정의하여 이 문제를 부분적으로는 해결 할 수 있음.
- 색인을 만들 때 설정하거나,
- 1색인을 닫고 2설정값을 변경한 후 3다시 열어야 함.

<br>

- close
~~~
curl --request POST 'http://localhost:9200/microservice-logs/_close'
~~~

- ***index.mapping.ignore_malformed = ture**
~~~
curl --location --request PUT 'http://localhost:9200/microservice-logs/_settings' \
--data-raw '{
   "index.mapping.ignore_malformed": true
}'
~~~

- open
~~~
curl --request POST 'http://localhost:9200/microservice-logs/_open'
~~~

- 이때, 유형이 맞지 않는 필드가 들어가면, 해당 exceptions 은 터지지 않고, 해당 필드 유형 규칙을 무시하고 그대로 색인 된다.

<br>

- 이는 모든 필드를 대상으로 적용됨.
- 그러나, 'ignore_malformed'는 JSON 개체 입력을 처리할 수 없다.
  - json 을 색인하면 mapper parsing exception 이 발생함..

### 해결 2
- 이렇게 발생하는 mapper parsing exception 문제로 색인되지 않아 데이터가 유실되는 문제에 대해 인지하고 있어야 한다.
- 해결 지침을 만들기
- 오류 document 를 별도의 대기열에 저장하는 DLQ 패턴을 사용하여, DLQ 를 통해 실패한 문서를 계속해서 처리할 수있도록 한다. 애플리케이션 또는 logstash 를 사용하여 처리.

## 2. 매핑 필드 수 제한 문제 illegal_argument_exception
- 기본적으로 매핑 필드 수는 1000개로 제한
- 늘릴 순 있으나, 이는 성능 저하와 메모리 부담이 발생할 수 있다는 것을 인지 주의해야 한다.

<br>

**동적 설정 변경으로 늘리기**

~~~
index.mapping.total_fields.limit": 1001
~~~


## 3. dynamic mapping 에서 많은 필드 추가로 발생하는 mappings explosions 문제


