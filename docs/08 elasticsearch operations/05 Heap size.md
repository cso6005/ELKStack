
es운영 시, es의 힙 크기를 정하는 것은 중요하다.

- es 의 기본 힙 크기는 1GB. 기본 설정으로 두었다면 나머지 63GB 는 낭비되지 않고 운영체제가 디스크를 캐시 하는데 사용할 것이다. 이는 이상적인 구성이 아님.
- 적어도 메모리의 절반이나 그 이하를 es에 할애해야 함. 일반적으로 물리적인 RAM 의 절반 정도를 es 자체에 할당하고 나머지를 운영체제에 할당
	- 그래서 Lucene 과 운영 체제가 그걸 캐시 목적으로 사용할 수 있다. 데이터를 캐시하는 데는 메모리가 많을수록 좋다.
	- ES 는 디스크 면에서 무겁기 때문에 디스크에 있는 것을 메모리에 캐시를 하는데, 그래서 많은 메모리를 갖는 게 낫다. 
	- 그래서 반반
- 만약, 분석된 문자열 필드에서 어떠한 집계도 하지 않는다면, 메모리가 절반보다 적어도 됨.
- 힙이 더 작으면 garbage collection 도 더 빨라진다. 그래서 그냥 캐시에 더 많은 메모리를 할당하는 게 아니라 garbage collection 또한 효율적으로 진행된다.
- es 는 java 위에 만들어졌다. java의 단점은 garbage collection 를 통해 메모리를 관리한다는 건데 한 번씩 모든 것을 싹 정리한다. 그래서 퍼포먼스가 일어나는 도중에 방해가 되기도 한다.
	- 그래서 작은 힙을 가지고 있다면 더 빠르다. 
- es의 힙 크기가 32GB만 넘지 않게 해야 함. 퍼포먼스가 너무 커지게 된다. 32GB 가 넘어가는 순간 메모리에 포인터를 저장하느라, 더는 바이트에 맞지 않게 않고, 더 많은 메모리가 필요하다. 
	- 32기가를 Elasticsearch에 할당하고 나머지 32GB를 OS와 Lucene에 할당 한 64GB가 안정적이라고 했던 이유. 
- 힙의 크기를 확인하고 이걸 실행하는 하드웨어에 맞게 설정해라.

### Heap 크기 지정 방법

- es 가 설정되면, 아래 설정이 둘 다 작동됨.

1. ES_HEAP_SIZE 환경 변수 내보내는 방법

- ex. 24GB 체제에서 실행하고 있다면 10GB 가 적당.
~~~
export ES_HEAP_SIZE=10g
~~~

2. ES_JAVA_OPTS 
- 특정 JAVA 가 힙 크기를 정하도록 지정. 약간 낮은 수준에서 추가하고 싶을 때 사용하면 됨.
- es 가 설정되면 