
- 퍼포머스의 향상과 그래프 세팅 변경 등에 따라, 색인 디자인을 조정하고 싶을 수 있는데, ES 는 매우 유연하고 색인 설정을 쉽게 바꿀 수 있음.
- 색인 수명 주기 도중에 높은 인제스트 부하, 다양한 강도에 따른 쿼리 볼륨 보고와 집계 요건, 그리고 데이터 유지 변경과 데이터 제거까지 변경할 수 있다.


### Type of Index Changes
- Index Settings
- Sharding
- Primary Data
- Low-Level Change
	- 내부 구조의 저준위 변경은 숫자나 부분을 고정시키는 것.
	- 'bit.ly/TuneELK'

## Index Settings
### Dynamic Settings
- 색인을 생성한 후에도 변경할 수 있음.
- 컨피규레이션이 내부 데이터 색인에 직접적으로 영향을 미치지 않는다.

> PUT requests to the/{index}\_settings endpoint

1. number_of_relicas
- 색인이 가진 복사본의 수
2. refresh_interval
- 인제스트 된 데이터가 쿼리를 위해 사용 가능해지는 간격
3. blocks
- 색인 읽기/쓰기 능력을 불가능하게 만든다.
4. \_pipline
- 문서에 첨부된 선처리 파이프라인을 선택
### Static Settings 
- 색인 생성후에는 변경할 수 없음.
- 이 설정들은 색인을 구성하는 실제 구조에 영향을 미침

1. number_of_shards
- 프라이머리 샤드의 수
2. codec
- 데이터의 압축 계획을 정의

# Sharding
- 우리가 더 쉽게 클러스터를 확장할 수 있게 해 주고 데이터의 더 높은 가용성과 복원성을 제공

### High Availability
- 높은 가용성이란 여러 개의 노드에 데이터와 서비스를 제공하며 긴 시간 동안 중단 없이 작업하는 수준의 서비스이다. 노드에 문제가 생겨도 다운되지 않고, 계속해서 정상적으로 작동할 수 있는 인프라 수준인 것이다.

### High Resiliency
- 충분한 복사본 보유로 어떤 장애가 생겨도 데이터 손실을 예방한고, 인프라 자체가 특정 에러를 막고 에러부터 회복할 수도 있다.

### How Sharding Work
1. 여러 개의 샤드를 가진 색인이 있을 때, 그 중 하나의 샤드가 다운되어도 다른 샤드가 계속해서 색인 기능을 유지할 수 있어야 하고, 또한 손실된 샤드의 요청도 수행해야 한다. 이게 바로 높은 가용성과 높은 복원성을 뜻한다.

2. 게다가 더 빠른 스피드르 원한다면 더 많은 샤드를 추가하고 여러 개의 샤드에 작업을 분해서 빠르게 만들 수 있다. 작업을 빠르게 완료하는 것을 떠나 개별 샤드의 작업량이 줄어들기 때문에 각각의 샤드에게 주어지는 부담도 덜하다. 규모를 확장하는 것도 마찬가지다. 작업은 동시에 더 빠르게 완료되고 각 개별 서버에 부담을 덜 수 있다.

## Changing Number of Shards
- 프라이머리 샤드의 수는 Static Settings 로 마스터 데이터의 구조에 영향을 끼치기 때문에 그냥 막 변경할 수가 없다. 
- 하지만 프라이머리 샤드와 반대로 레플리카 샤드의 수는 마스터 데이터에 여향을 끼치지 않기 때문에 색인이 생성된 후에도 변경할 수 있다. 
- 프라이머리 샤드의 수를 바꾸고 싶다면 수동적으로 새 색인을 생성하고 alias 와 읽기 전용 색인을 사용하여 모든 데이터를 다시 인덱싱하는 방법이 있다.
- 위 방법보다 더 빠르게 할 수 있는 다른 방법으로는 Helper API 가 있다.


## change shards by Healper API
- 두 가지 모두 다 새로운 목표 색인 이름을 입력해주어야한다.

- TO decrease
~~~
POST /{source-index}_shrink/{target-index-name}
~~~

- TO increase
	- 계수 3으로 분할하여 샤드 수 늘리기
	- 하나의 샤드가 3이 되고, 새로운 색인에도 다른 모든 정의된 색인 설정은 그대로 남는다.
	- 샤드 세 개에 분배
~~~
POST /{source-index}_split/{target-index-name}

{
"settings" : {
	"index.number_of_shards":3
	}
}
~~~

## Split API
- 더 많은 노드에 작업을 나누기 위해 샤드의 수를 올리고 싶다며 split API 를 사용할 수 있다.
- 그냥 샤드를 추가하는 것이 아니라, 곱셈이다. **원래 샤드를 곱해서 샤드를 분할**한다.

	split 2 shards into 4 (factor of 2)
	split 2 shards into 6 (factor of 3)
	split 1 shard into any number fo shards
	can't split 2 shards into 3 (It's not an even coefficient)

**분할하기 위한 조건**

- 분할을 하기 위해서는 색인은 읽기 전용이 되어야 하고
~~~
	"index.blocks.write" = true
~~~

- 클러스터 health는 green 이 되어야 한다.

### force merge API
- 결합작업이 자동으로 실행되어 데이터의 사이즈를 줄일건데, 기다리기 싫으면 force merge API 로 바로 결합할 수 있다.
- 생성 시스템에서 해당 api 는 주의해서 사용해야 한다. 몇몇 변수는 예상치 못한 결과를 불러오기에 문서를 꼭 읽고 사용하는 것이 좋음.

## Shrink API 
- 샤드를 분할하는 작업이 원래 샤드를 곱하는 것으로 이뤄진다면, **Shrink API 는 샤드의 수를 줄이기 위해 샤드를 나누는 걸**로 작동한다.
- 샤드를 빼는 것이 아니고 나눈다는 것이다.

	shrink 8 shards down to 4, 2, or 1
	shrink 15 shards down to 5, 3, or 1

**조건**

- 색인은 읽기 전용이 되어야 하고
~~~
	"index.blocks.write" = true
~~~

- 클러스터 health는 green 이 되어야 한다.
- 색인 내 모든 샤드의 복사본이 같은 노드에 있어야 한다.

### When does change primary data

1. 물리적인 자원의 변경
- 자원의 한계은 올 수 밖에 없다. 초당 몇 백개의 문서를 인제스트하다보면 결국 스토리지 한계가 생길 것이다.

2. 데이터 가치의 하락
- 시간이 지났을 때 중요성이 떨어지는 데이터가 있다. 이렇게 데이터의 중요성 가치는 변한다.
	- 이러한 데이터의 가치는 변하기 때문에 es가 데이터를 roll up 해서 집계된 화면을 생성하고 그것들을 다른 장기 색인에 저장하도록 하는 이유이다.