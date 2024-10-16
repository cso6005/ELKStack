# kibana canvas
인포그래픽이 포함하여 es 데이터를 시각적으로 만들 수 있도록 도와주는 도구

- 필요한 데이터를 실시간으로 추출
- workpad 작업 공간
  - page 페이지 형태의 대시보드
    - element
- 사용법
  - kibana dashboard canvans
  - expression editor 하단에 클릭 -> EL 로 작성 가능 

## 
- 캔버스는 kibana timelion 과 비슷하게 맞춤형 표현언어(EL) 로 구동된다.
- 맥락이라고 알려진 타이핑 결과에 의해 정의도니 기능을 연결한다. 즉, 타이핑은 하나의 기능에서 결과를 가져와 추후 처리될 다른 기능에서 결과를 가져와 추후 처리될 다른 기능에 전달하는 것.

## element 에 표시될 데이터를 추출 하는 방법
### 1. elasticsearch sql 명령
- format=txt : 리턴 결과를 텍스트 기반의 샘플로 보기 위해서
~~~
POST /_sql?format=txt
{
    "query": "SELECT sum(bytes) AS total_transferred FROM nginx GROUP BY remote_ip ORDER BY total_transferred DESC NULLS LAST LIMIT 5"
}
~~~

### 2. timelion 
- 색인 문서로부터 바로 가져온 원시 문서나 시간 기반 데이터를 사용해서 작업
https://coralogix.com/blog/advanced-guide-to-kibana-timelion-functions/

