# Meta Model Object Mapping

Meta Model 기반의 접근방식은 ES에서 읽기/쓰기를 위해 도메인의 타입 정보를 사용합니다.

이를 통해서 도메인 타입 매칭을 위한 converter 인스턴스를 등록할 수 있습니다.

MappingElasticsearchconverter는 메타데이터를 사용하여 도큐먼트로의 오브젝트 매핑을 수행합니다.

메타데이터는 애노테이션이 달린 도메인 엔티티의 property에서 가져오고 제공하는 애노테이션은 다음과 같습니다.

- @Document : ES에 매핑할 클래스임을 나타내고 클래스 레벨에 적용합니다. 다음과 같은 속성을 제공합니다.
    - indexName : 이 엔티티를 저장할 인덱스 이름
    - createIndex : repository 부트스트래핑에 인덱스를 부여할지 여부, default = true
    - versionType : 버전 관리, default= EXTERNAL
- @Id : ID 목적으로 사용되는 필드 표기
- @Transient : 기본적으로 모든 필드는 저장 또는 검색 시 도큐먼트에 매핑되는데 이 주석이 포함된 필드는 제외
- @PersistenceConstructor : 데이터베이스에서 개체를 인스턴스화할 때 사용할 생성자(보호된 패키지도 포함). 생성자 argument는 검색된 문서의 키 값에 이름에 의해 매핑
- @Field : 필드의 속성 정의, 대부분의 속성이 Elasticsearch Mapping 정의에 매핑
    - name : ES 문서에 나타낼 필드 이름, 설정되지 않을 경우 Java 필드 이름 그대로 사용
    - type : 필드 유형 정의, [유형 참조 문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)
    - format 및 pattern : Date 타입의 필드의 경우 반드시 format을 정의해야 한다.
    - store : 원래 필드 값이 ES에 저장되야 하는지 판단하는 플래그, default = false
    - 커스텀 analyzer와 normalizer를 지정하여 커스텀마이징하는 analyzer, searchAnalyzer, Normalizer
- @GeoPoint : 필드가 geo_point 데이터 형식임을 표기, 만약 필드가 GeoPoint 클래스의 인스턴스인 경우 생략 가능

> TemporalAccessor에서 파생되는 속성의 FieldType.Date 타입은 @Field 어노테이션이 있어야하며 이 타입에는 커스텀 converter를 등록해야 합니다.
> Elasticsearch 7의 변동으로 인해 사용자 정의 날짜 형식을 사용하는 경우, 연도에 yyyy 대신 uuuu를 사용해야 합니다

### Field의 type

| 데이터 타입 | 설명 |
| --- | --- |
| text | 전문 검색이 필요한 데이터로 텍스트 분석기가 텍스트를 작은 단위로 분리 |
| keyword | 정렬이나 집계에 사용되는 텍스트 데이터로 분석을 하지 않고 원문 통째로 인덱싱 |
| date | 날짜/시간 데이터 |
| btye, short, integer, long | btye : 부호있는 8비트 데이터short : 부호 있는 16비트 데이터integer : 부호 있는 32비트 데이터long : 부호 있는 64비트 데이터 |
| scaled_float, half_float, double, float | scaled_float : float 데이터에 특정 값을 곱해서 정수형으로 바꾼 데이터, 정확도는 떨어지나 필요에 따라 집계 등에 효율적으로 사용half_float : 16비트 부동 소수점 실수 데이터double : 32비트 부동소수점 실수 데이터float : 64비트 부통소수점 실수 데이터 |
| boolean | 참/거짓 데이터로 true/false 값 |
| ip | ipv4, ipv6 타입 ip주소 |
| geo-point, geo-shape | geo-point : 위도, 경도 값geo-shape : 하나의 위치 포인트가 아닌 임의의 지형 |
| integer_range, long_range, float_range, double_range, ip_range, date_rage | 범위를 설정할 수 있는 데이터로 최소, 최대값을 통해 입력 |
| object | 계층 구조를 갖는 형태로 필드 안에 다른 필드들이 들어갈 수 있다. |
| nested | 배열형 객체를 저장한다. 객체를 따로 인덱싱하여 객체가 하나로 합쳐지는 것을 막고 배열 내부의 객체에 쿼리로 접근 가능 |
| join | 부모/자식 관계를 표현할 수 있다. |

## 매핑방법

### 1. 애노테이션로 매핑하는 방법

```java
public class MemberDocument {

    @Id
    @Field(type = FieldType.Keyword)
    private Long id;

    @Field(type = FieldType.Text)
    private String name;
}
```

## 2. Json 파일로 매핑

json 을 통해 따로 매핑관련 정의를 작성한 뒤 @Setting 애노테이션을 클래스에 부착하고 json 파일 경로로 매핑하는 방법

- 복잡할 경우, json 파일로 만들어서 하기
- json 파일을 매핑하더라도 date타입의 경우에는 @Field 애노테이션으로 설정해줘야 한다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Document(indexName = "performance")
@Mapping(mappingPath = "elastic/performance-mapping.json")
@Setting(settingPath = "elastic/performance-setting.json")
public class PerformanceDocument{
```

```json
{
  "properties" : {
    "id" : {"type" : "keyword"},
    "title" : {"type" : "text", "analyzer" : "korean"},
    "musicalDateTime" : {"type" : "keyword"},
    "place" : {"type" : "text", "analyzer" : "korean"},
    "imageUrl" : {"type" : "text"},
    "performanceType" : {"type" : "text"},
    "startDate" : {"type" : "date", "format": "uuuu-MM-dd'T'HH:mm:ss.SSS||epoch_millis"},
    "endDate" : {"type" : "date", "format": "uuuu-MM-dd'T'HH:mm:ss.SSS||epoch_millis"}
  }
}
```

### 아무것도 매핑하지 않았을 시, ES가 아래와 같이 자동으로 데이터 타입에 맞춰 인덱스를 매핑함.

| 원본 소스 데이터 타입 | 다이내믹 매핑으로 변환된 데이터 타입 |
| --- | --- |
| null | 필드를 추가하지 않음 |
| boolean | boolean |
| float | float |
| integer | long |
| object | object |
| string | string 데이터 형태에 따라 data, text, keyword |

### 텍스트 타입과 키워드 타입 설명 및 예

- 텍스트 타입
    - 문장이나 여러 단어가 나열된 문자열
    - 기본적으로 **집계나 정렬을 지원하지 않습니다.**
    - 텍스트 타입으로 지정된 문자열은 분석기에 의해 토큰으로 분리되고, 토큰들이 인덱싱 되는데 이를 역인덱싱이라고 하고 역인덱싱 된 토큰들을 용어(term)이라고 합니다.
- 키워드 타입
    - 카테고리, 사람 이름, 브랜드 등 규칙성이 있거나 유의미한 값들의 집합
    - 텍스트 타입과 달리 분석기를 거치지 않고 문자열 전체가 하나의 용어로 인덱싱
    - 데이터 부분 일치 검색은 어렵지만 완전 일치 검색을 위해 사용할 수 있으며 집계, 정렬에 사용 가능
- 멀티 필드
    - 단일 필드 입력에 대해 여러 하위 필드를 정의하는 기능으로 fields라는 매핑 파라미터가 사용됩니다.
    - fields는 하나의 필드를 여러 용도로 사용할 수 있게 만들어줍니다.
    - 예를 들면, 문자열의 경우 전문 검색이 필요하면서도 정렬도 필요한 경우가 있는데 여기에 텍스트 타입과 키워드 타입을 같이 적용하는 방식입니다.
    

**ex ) 멀티 필드 설정 예제**

```java
// 인덱싱 해놓고 http://localhost:9200/인덱스이름?format=json&pretty 에 접속해보면
"name" : {
  "type" : "text",
  "fields" : {
    "keyword" : {
      "type" : "keyword",
    }
  }
}
```

name 필드가 text 텍스트 타입이면서도 fields로 인해 키워드 타입으로도 동시에 사용되고 있는 것을 확인할 수 있습니다.

### Array type

[Elasticsearch Array 형식으로 저장하기](https://stdhsw.tistory.com/entry/Elasticsearch-Array형식으로-데이터-저장하기)