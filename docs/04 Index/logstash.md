# Logstash

> 실습 예제 
> <br> - acess-log file
> <br> - mysql
> <br> - s3
> <br> - csv file

## 1. elasticsearch 로의 색인

### import from just about anything
elasticsearch 를 다양한 생테계가 존재하며, json 요청, 클라이언트 api 를 통해 

큰 정적 저장소든, 실시간 웹 로그 스트리밍이든, 그 어디에 데이터가 있더라도 es 로 불러올 수 있는

독립적 스크립트를 작성할 수 있다.

Logstash나 Beats와 같은 기성 구성 요소를 사용할 수 있다. 둘다 elastic stack 의 일부이며, 

다양한 exporter 와 importer 가 내장되어 있어, 거의 모든 시스템과 연결할 수 있다.

또한, 자바, 파이썬 등 많은 클라이언트를 통해, 우린 데이터를 es 로 색인할 수 있다.

**aws 예시**

aws 의 경우, 다른 aws 구성요소로 부터 데이터를 스트리밍 할 수 있는 시스템을 갖추고 있다.

- aws lamdba 를 이용하여 한 시스템의 데이터를 aws 의 elastic search 로
- AWS Kinesis가 데이터를 끌고 다닌다면 Kinesis Firehose를 사용할 수

## 2. Logstash

- logstash 는 웹서버에 생성되는 로그 파일과 elasticsearch 사이에 위치한다.
- 매우 유연하며, 거의 모든 곳에서 데이터를 불러올 수 있다.
- 전혀 다른 시스템인 Kafka나 S3나 HDFS와 같은 분산 파일 시스템에서, 데이터를 가져와, Elasticsearch를 포함해 AWS, Hadoop 클러스터, db 등에 집어 넣을 수 있다.
- 한 번에 여러 소스에서 불러와 여러 곳으로 출력할 수도 있다.
- logstash 는 최소 한 번의 전달을 보장
- 여러 노드 간에 확장이 가능하다.
- 사용할 수 있는 다양한 입출력 플러그인이 있음.


### 데이터 plumbing 이상의 역할
- 단순한 데이터 plumbing 뿐만 아니라, 구조화되지 않은 데이터를 구조화시키는 작업도 함.
    - 예를 들어 웹 로그를 분석하는 경우, Logstash는 웹 로그의 문장들을 쪼개어, 명명된 필드로 구조화하며 필요없는 데이터를 걸러낼 수 있다.
- 데이터를 변환하고 개인정보가 보이는 즉시 익명화하거나 완전히 제외시키는 등 작업도 가능.
- geo-location 조회 가능
    - 예를 들어, 웹 서버의 엑세스 로그에 기록된 ip 주소만으로 그 로그 항목이 어디서 왔는지 자동으로 파악할 수 있다.
- 데이터를 생성하는 시스템과 데이터를 저장하려는 위치 사이에서 일종의 완충재 역할을 함.
    - 웹 서버 플릿에 많은 부하가 발생하더라도, elasticsearch 나 다른 시스템으로의 데이터 전송에는 그대로 적용되지는 않음.


### 설정 파일 예시 및 구성

logstash.conf

~~~ 

input {
    file {
        path => "/user/log1" # 읽을 파일 경로
        start_position => "beginning" # 해당 로그의 맨 처음부터 작업을 강제 시킴
    }
}

filter {
    grok {
      match => {"message" => "${COMBINEDAPACHELOG}"}
    }
    date {
        #  Logstash가 로그 문장 자체에서 타임스탬프를 추출하여 es에 게시되는 이벤트 시간으로 사용
        match => ["timestamp", "dd/MMM/yyy:HH:mm:ss Z"]
    }
}

output {
	elasticsearch {
        # 로컬 es 로
		hosts => ["elasticsearch:9200"]
	}
    stdout {
        # rebydebug 코덱을 사용해 콘솔로 보내기
        codec => rebydebug
    }
}


~~~

1. input
- logstash 가 입력 데이터를 찾을 위치를 지정하고, 이 시스템에서 로컬 파일을 모니터링할 플러그인을 사용해 logstash 에 데이터를 공급

2. filter 
- 구조화된 데이터를 추출하는 방법을 설정

3. output
- 추출한 구조화된 데이터를 어디로 보낼지


## input config

### 1. start_position

~~~
start_position => "beginning"
~~~

파일의 처음부터 읽으라고 지시
해당 설정은 **처음 읽는 파일에만 적용**
프로그램은데이터가 주기적으로 추가되는 파일을 처리하기 때문에 기본적으로 파일을 끝에서부터 읽는다.
그러면 csv 파일의 끝에 추가되는 데이터만 가져올 수 있다. 
이전에 해당 파일을 본적이 있는 경우 sincedb_path 변수를 사용해 수행할 작업을 결정한다.


### 2. sincedb_path 
~~~
sincedb_path => "dev/null"
~~~

db path 는 입력 파일에 분석된 마지막 줄을 추적하는 데이터베이스 파일을 가르킨다.
꼭 데이터베이스가 아니더라도, 입력받는 모든 파일을 뜻한다.
이 입력 파일이 나중에 다시 분석되면 인식된 db 파일에 기록된 위치부터 작업이 계속 진행된다.

- dev/null 로 설정
존재하지 않음으로 설정.

그렇다면, logstash 는 해당 파일에서 마지막으로 읽힌 줄을 기록해 놓을 수 없기에

매번 처음부터 전체 파일이 처리되게 되는 것이다.


## output config

### 1. stdout
~~~
stdout {}
~~~

터미널에서 상태, 출력, 로그 정보를 가져오며 표시되도록 함.

- 코덱 설정
~~~
stdout {
    # rebydebug 코덱을 사용해 콘솔로 보내기
    codec => rebydebug
}
~~~
코덱 설정을 할 수 있다.

### 2. elasticsearch
~~~
	elasticsearch {
		hosts => ["elasticsearch:9200"]
    index => "demo-json-drop"
	}
~~~

Elasticsearch 하위 섹션에서는 프로그램에 데이터를 Elasticsearch로 보낼 것을 지시

호스트(hosts) 옵션은 ElasticSearch가 들어오는 연결을 수락하는 위치를 지정


## filter

### 1. logstash JSON Filter

- logstash 의 기본 플로그인에는 JSON 필터 플러그인이 포함되어 있다.
- 해당 플러그인은 json 데이터를 구문 분석하고 logstash 이벤트에 해당하는 데이터 구조를 생성한다.
    - 해당 구문 분석 단계가 없으면 es 에 json 데이터가 모두 한 줄의 텍스트로 들어 갈 것.

- 예시1
~~~
filter {
  json {
    source => "message"
  }
}
~~~

- 위 설정은 json  구문 분석에 필터를 사용하기 위한 최소한의 설정임.
- json 필터 섹션 내 json 데이터를 가져올 소스를 정의한다. 이 경우엔 message 필드인데, 이는 logstash 가 모든 이벤트를 메시지라는 필드에 저장되기 때문이다.


- 예시2
~~~
  split {
    filed = "[pastevents]"
  }
  mutate {
    remove_field => [
      "message", "@timestamp"
    ]
  }
~~~

~~~
"name" => "dldl",
"pastEvents" => {
    "id" => "324",
    "eventId" => 10
}
~~~


필드가 list 등 collection type 일때, 이를 개별 필드로 나누고 싶을 때

=> split 필터 사용하기

split 필터는 필드 중 하나를 분할하고, 각각 분할된 값을 복제 클론 내에 배치하는 방식으로 이벤트를 복제한다.

상위 이벤트 또는 문서가 n 번 복사된다.여기서 n 은 집합 내 항목의 수이다.


## grok 을 사용한 logstash 구문 분석 및 필터링
json, csv 파일 데이터의 경우 es가 분석할 수 있게 형식에 맞추어 완벽하게 정리 되어 있기에 쉽게 분석 가능한 것이다.

일반 텍스트 로그와 같은 비구조화 데이터를 작업하기 위해선 데이터를 구문분석해 구조화데이터로 변환해야 한다.

이를 위해 logstash 에서 제공하는 filter 를 사용한다.

### grok filter
- 구문 분석에서 각 필드가 무엇을 나타내는지 감지가 가능.
- 각 텍스트를 분석하고 지시한 패턴과 일치하는지 확인함으로 동일한 작업을 수행할 수 있음. => **w정규식 regular expressions, REGEX 을 찾는다.**
- 미리 정의된 패턴 이름를 사용하면 된다.

### Generic Grok Syntax
%{PATTERN:identifier}






