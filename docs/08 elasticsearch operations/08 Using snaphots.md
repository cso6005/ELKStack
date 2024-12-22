
더 탄탄한 안정성을 위해 최악의 상황을 대비해 어딘가에 전체 색인의 백업을 저장해둬야 한다. 그래서 es는 백업 메커니즘으로 스냅샷을 제공한다. 

- 다른 머신, 다른 데이터 센터 등 외부 스토리지까지 원하는 어디에든 저장할 수 있다.
- **지난 스냅샷에서 변경된 것만을 저장**한다. **자원을 보존하기 위해 증가하는 작업만 저장하는 것**이다. 매번 전체 데이터의 새로운 복사본을 저장하는 것 대신 변경된 부분만 저장하기 때무에 일련의 스냅샷을 매우 빠르고 효율적으로 만들 수 있는 것이다.

### 1. create a repository

- elasticsearch.yml 에 저장소 위치를 설정
	- yml 파일에 es 안 클러스터의 구성의 일부로 설정해줘야 함.
~~~
path.repo : ["/home/<user>/backups"]
~~~

- 클러스터에 명령
	- repo 타입을 지정
	- 스냅샵이 저장될 위치 설정

~~~
PUT _snapshot/backup-repo
{
	"type":"fs",
	"settings": {
		"location": "/home/<user>/backups/backup-repo"
	}
}
~~~

### 2. Using Snapshots

- 모든 색인에 스냅샷을 생성
	- snapshot-1 이라는 스냅샷을 생성

~~~
PUT _snapshot/backup-repo/snapshot-1
~~~

- 스냅샷 정보 얻기

~~~
GET _snapshot/backup-repo/snapshot-1
~~~

- 현재 저장되고 있는 스냅샷의 진행 상황을 모니터링
~~~
GET _snapshot/backup-repo/snapshot-1/_status
~~~

- 스냅샷 복원
	- 먼저 모든 색인을 닫아준다. 
		- 백업에서 복원하는 동안 누군가가 색인에 데이터를 써서는 안 될 것이다.
	- 특정 스냅샷에 있는 대로 복원시켜준다.
~~~
POST /_all/_close
POST _snapshot/backup-repo/snapshot-1/_restore
~~~


# Snapshot Lifecycle Management

SLM 은 es 클러스터와 인덱스의 백업을 완전히 자동화할 수 있다.

SLM 이 언제 백업을 수행할지, 얼마나 자주 백업할지,

그리고 자동으로 삭제하기 전에 각 스냅샷을 얼마나 오랫동안 유지할지를

지시하는 정책을 설정할 수 있다.

### 1. Set up a repository where the snapshots can be stroed

스냅샷을 저장할 수 있는 저장소를 설정해야 함.

1. 공유 파일 시스템 저장소
- 모든 마스터와 데이터 노드에 같은 위치에 마운트. 로컬 하드 드라이브에 저장하는 것 처럼.
- 단일 노드에 작업할 때는 이것이 쉬움.

2. 클라우드 데이터 자장 서비스
- 여러 노드가 관련될 경우 다른 서버에 위치한 동일한 파일 시스템에 접근할 수 있도록 구성해야 함.
- aws S3, azure Storage, GCS 와 같은 서비스에 연결하기 위해 서비스 특정 플러그인을 사용한다.

### 2. Configure Repository
- 원하는 장소를 선택했으면, 이를 es에 설정한다.

1. 공유 파일 시스템
- elasticsearch.yml 파일에 구성 라인을 추가

2. 클라우드 스토리지
- 각 노드에 필요한 저장소 플러그인을 설치해야 함.
- 이러한 클라우드 서비스에 로그인해야 하므로, 필요한 비밀 키를 키 저장소에 추가해야 함.

### 3. Define The SLM Policy

- SLM 에게 백업을 자동화하는 방법을 지시하는 정책을 다음과 같은 매개변수로 정의

1. Schedule
- 데이터 스냅샷의 빈도, 시간을 설정.
- 스냅샷은 증분 방식으로 마지막 스냅샷과 현재 스냅샷 간의 차이만 저장하면 된다. 

2. Name
- 스냅샷을 명명할 때 사용하는 패턴을 정의

3. Repository
- 스냅샷을 저장할 위치를 지정

4. config.indices
- 포함할 인덱스 목록

5. Retention
- 보존은 SLM이 일부 스냅샷을 삭제할 수 있는 시점을 정의하는데 사용할 수 있는 선택적 매개변수
- expire_after
	- 시간 기반 만료
	- 1월1일에 생성된 스냅샷이 30일로 만료 설정했을 경우, 1월 31일 이후에 삭제 대상이 된다.
- min_count
	- 스냅샷의 갯수로 스냅샷을 유지하지 하도록
	- es에게 만료되었더라도 최소한 이 숫자의 스냅샷을 유지하도록 지시하는 것.
- max_count
	- 스냅샷의 갯수로 스냅샷을 유지하지 않도록
	- es 에게 이 숫자 이상의 스냅샷을 절대 유지하지 않도록 지시
	- 100개의 스냅샷 중 하나만 만료되어도, 이 파라미터를 50 으로 설정했다면, 만료되지 않은 것들도 포함해서 가장 오래된 50개의 스냅샷을 삭제하는 것이다.

### 기타 명령어

- execute
	- 스냅샷을 즉시 실행하는 명령어
~~~
POST _slm/policy/back_policy_daily/_execute
~~
