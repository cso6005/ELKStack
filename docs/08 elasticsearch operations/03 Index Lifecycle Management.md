- 수명 주기 정책을 지정한 각 단계 별 관리
- 오래된 색인을 어떻게 다룰지 제어할 수 있다.
- 색인을 그저 고정된 로테이션 스케줄에 따라 관리하는 것이 아니라, 샤드 사이즈나 퍼포먼스에 기반으로 관리한다.


1. Hot
- 활발하게 업데이트되고 쿼리되고 있음.

2. Warm
- 쿼리는 하나, 더는 업데이트되지 않아서 읽기만 되는 단계

3. Cold
- 색인이 더이상 업데이트 되지 않고, 드물게 쿼리만 되는 단계
- 쿼리가 느려지기 때문에 덜 비싼 하드웨어를 써서 비용을 절약해야 함.

4.  Delete
- 필요없는 색인

## Index Lifecycle Policies

- 새 색인에 적용하고 싶은 사이즈나 수명, 또는 이 색인이 언제 삭제될 수 있는지 등 정책을 각 단계에 만들어 보자.

### ILM 정책 정하기
	- 색인의 크기가 50GB 에 도달하거나 수명이 30일 정도 되면 delete 단계로 롤오버 되어 완전히 지워지기 전에 90 일 정도 남아있게

~~~
PUT _ilm/policy/datastream_policy
{
	"policy": {
		"phases": {
			"hot": {
				"actions": {
					"max_size": "50GB",
					"max_age" : "30d"
				}
			},
			"delete": {
				"min_age": "90d",
				"actions": {
					"delete":{}
				}
			}
		}
	}
}
~~~

### index template 설정
- 위에서 정의한 ILM 정책을 index template 에 설정한다.
	- 'index.lifecycle.name' 에 지정
	- index 자체에 설정하는 것이 아니라 템플릿에서 지정한다.
- 'index_patterns' 에는 rollover 되며 template 이 적용될 index 패턴을 기입한다.
- 'index.lifecycle.rollover_alias'에는 rollover 되며 index 들에 alias 명칭 등록
	- 데이터 처리는 모두 alias 를 통해 처리할 수 있도록 한다.
- alias 는 설정을 더 거쳐야 한다.

~~~
PUT _template/datastream_template
{
	"index_patterns": ["datastream-*"],
	"settings": {
		"number_of_shards": 1,
		"number_of_relpicas": 1,
		"index.lifecycle.name": "datastream_policy",
		"index.lifecycle.rollover_alias": "datastream"
	}
}
~~~