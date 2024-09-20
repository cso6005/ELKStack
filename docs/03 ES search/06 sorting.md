# sorting
- sort 파라미터 설정

~~~
sort=year
~~~

- 여러 개일 경우, 리스트에 담기
- order : asc(default), desc
~~~
 "sort": [
    {
        "title": {
            "order": "desc"
        }
    }
]
~~~

## 주의 사항
### 분석된 text 필드를 다룰 때 정렬이 까다로움
필드에 대해 full text search 도 해야 하고, 정렬도 해야 할 때, 문제가 발생한다.

text 필드의 경우, 구문의 용어들로 쪼개서 개별단어와 단어 순서만 역색인에 저장한다. 

즉, 전체 문자열 자체가 저장되지 않기에 따라서 영화 제목 자체로 정렬이 되지 않는다.

하면, illegal_argument_exception. 으로 해당 키워드 필드의 경우 절렬 할 수 없다고 응답한다.

### 해결법
분석되지 않는 하위필드를 설정한다.

필드를 text 유형으로 설정하여 full text search 를 위해 분석되게 한다.

그리고 그 안에 하위 필드를 두어, 해당 필드는 keyword(분석되지 않고 필드 전체를 그대로 저장하는 타입) 로 설정한다.

~~~
{
    "mappings": {
        "properties": {
            "title": {
                "type": "text",
                "fields": {
                    "raw": {
                        "type":"keyword"
                    }
                }
            }
        }
    }
}
~~~

>전체 텍스트 검색할 때는 title 사용.
>
>정렬 작업이나 분석되지 않는 필드를 필요로 하는 작업에는 title.raw 필드 사용.
>
>sort=title.raw

- 이 방법은 애초에 색인 전에 매핑 설정을 해줘야 하므로, 이미 데이터가 있다면 기존 색인을 삭제하고 새로운 매핑의 색인을 다시 만들어야 한다. 그렇기에 항상 이런 점을 설계할 때 고려해야 한다.