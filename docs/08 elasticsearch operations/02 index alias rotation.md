존재하는 색인을 재 인덱싱하지 않아도, 샤드를 추가할 수 있는 방법이 있다.

application 에 완전히 새로운 색인을 생성하고, 그 색인에 검색 요청을 보내는 것이다.

즉, 색인의 데이터를 포함하는 여러 개의 색인을 만드는 것인데,

이때 **index alias 를 통해 어떤 색인이 런타임에 사용될지 관리하는 것**이다.

이 방법은 이전의 색인을 건드리지 않고도 새 색인에 용량을 추가할 수 있다.

### index alias
- 분리된 색인을 만드는 것
- 보편적으로 현재와 관련된 특정 기간을 쿼리할 시간 기반의 요청이 있다면 매우 좋은 전략이 될 것.
- 새 데이터가 alias 에 사라진다면 클러스터 공간을 확보하기 위해 그것들을 지우며 비교적 일정하게 서버 용량을 유지하며 관리하면 좋다. 스냅샷 백업

### alias rotation example
- 6월의 데이터를 'logs_last_3_months' 에일리어스에 더하고 3월의 데이터를 지워서 삼개월 데이터 유지
- 'logs_current' 에일리어스를 여기에 6월의 데이터를 추가해고 5월을 지워서 가장 최근 달을 가르키게 유지.
~~~
POST /_aiases
{
	"actions": [
		{"add" : {"alias": "logs_current", "index": "logs_2024_06"}},
		{"remove" : {"alias": "logs_current", "index": "logs_2024_05"}},
		{"add" : {"alias": "logs_last_3_months", "index": "logs_2024_06"}},
		{"remove" : {"alias": "logs_last_3_months", "index": "logs_2024_03"}}
	]
}
~~~
