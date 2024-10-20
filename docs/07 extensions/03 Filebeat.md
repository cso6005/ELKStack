# Filebeat is a lightweight shipper for logs

- filebeat 는 로그 파일과 logstash 사이에서 로그 데이터를 수집하고 전송하는데 더 가벼운 인터페이스를 제공
- 매우 가벼운 버퍼 레이어를 추가로 제공해서 개개인의 웹 서버에서 filebeat 를 실행할 수 있음.
- 웹 서버에서 로그 파일의 변화를 찾아, 효율적이고도 간단하게 logstash 또는 es 로 전송함.
- 추가 레이어를 가진다는 것은 추가적인 보호장치를 가지고 있다는 것과 똑같다. -> **backpressure**
- 로그가 백업을 시작하면 filebeat 에서 backpressure 도 영향을 미치기 시작한다. 속도를 조금 줄이게 해줌. 
	- es 가 삽입에서 백업하기 시작하면 logstash 에게 속도를 늦추라고 말함. 그리고 filebeat 에도 속도를 늦추하고 하는 것.
	- 이렇게 데이터를 캐싱할 수 있고 병목 현상이 일어날 때 완충해줌으로서 유연성을 제공하고 더 가벼운 솔루션을 제공하게 되는 것.
- 웹 서버의 경우 로그 파일이 생성됨과 동시에 전송하는 **logstash 같은 무거운** 시스템이 필요하지 않을 수 있다. 
- apache, nginx, mysql 등 그 어떠한 종류의 로그도 filebeat 에서 소비될 수 있다.

### 주기능
1. 모든 로그 행이 queue 의 역할을 할 수 있게 각 로그에서 read pointer 를 유지하는 것.
2. filebeat 는 이벤트의 queue 로 인지하고 logstash 나 es 또는 설정한 무언가로 보내는 것이다.

### backpressure
- 추가 완충 레이어가 es 와 로그 사이에 있는 것
- 자료가 파이프라인 너머로 백업되면 자료들을 중간에 잡아주는 기능이다.
- filebeat와 logstash 를 연결해서 사용해야 하는 가장 큰 이유인 기능이다.
- 주로 들어오는 데이터를 계산 처리하고 마지막 목적지로 보내기 전에 변환하는 데 사용함.
- 문제는 logstash 가 로그로부터 들어오는 모든 데이터를 따라갈 용량이 안된다. 이때 속도를 제어하지 않는다면 시스템이 다운되거나 트래픽이 터지는 경우가 발생할 수 있다.

1. 처음 부터 filebeat 는 각 머신에 나타나는 실제 로그 파일 안에 **read pointer** 를 유지하는 것이다.
	1. filebeat 가 이 read pointer 를 logstash 가 감당할 수 있는 것보다 빠르게 움직이지만 않으면 됨.
2. filebeat 는 로그가 생성되는 실제 머신의 에지에서 실행되고 있는데, 추후 처리를 위해 정보들을 logstash 클러스터로 보낸다. 이건 logstash - filebeat 사이에 있는 backpressure sensitive protocol 을 통해 작동됨.
3. 만약 백업이 시작되는 것을 logstash 가 보게 되면, sensitive protocol가  filebeat 속도를 늦추게 한다.
4. 그러면 filebea는 logstahs 가 백업을 받을 때까지 로그로 read pointer 를 움직이는 곳에서 속도를 늦추고 logstash 가 괜찮다고 생각하면 다시 filebeat 가 정상 속도로 돌아오게 된다.

- 기본적으로 시스템에 가까운 Logstash 에 전력을 공급하는 머신의 일시적인 트래픽 초과를 해결하려 과하게 확장하는 일을 막아준다.

### 실제 생성 환경의 설정 예시
- 수집 관점에서 데이터를 모니터하고 추후 처리를 위해 logstash 로 보낸다.
- beats가 logstash 노드 그룹의 균형을 맞춘다. 그래서 logstash 노드 하나가 다운되어도 시스템은 작동한다. (높은 가용성을 위해 최소 두 개의 logstash 노드가 있는 게 좋음.)

- logstash 로 보내진 데이터들이 logstash 가 준비될 때까지 메모리에 저장된 **persistent queues** 기능이 있음. 
	- 최소 전달 보증 도구
	- 이 데이터를 디스크에 대신 저장할 수 도 있어서 logstash 노드가 다운되어도 그 queue 에 있는 정보는 유실되지 않음.
	- logstash 가 다시 시작되면 persistence queue 에 있는 메시지를 전달하려고 한다. 전달이 최소 한 번은 성공할 때까지.

- es 는 최소 세개 의 마스터 노드를 가져야 한다.
- 클러스터에 읽기 쓰기를 담당하는 data 노드
	- warm data nodes : 쓰기만 함. hot data 노드가 다운됐을 때 바로 그 자리를 채우기도 함.
	- hot data nodes

### ex
- apache 엑세스 로그를 모니터링 하는 예시
1. apache.yml.disabled 파일을 apache.yml 으로 수정
	1. apache 패턴 모듈이 설정되어 있음.
2. filebeat 에게 로그가 어디에 있는지 경로 설정
	1. var.paths 로 가서 경로 목록 적어줌.
	2. error logs 밑에도
3. filebeat 작동
	1. sudo /bin/systemctl start filebeat.service
4. 엑세스 로그를 가져오고 자동으로 es 에게 넣음.