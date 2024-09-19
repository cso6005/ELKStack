# 1. docker 설치

1. git clone 으로 컨테이너 받기

https://github.com/deviantony/docker-elk

```bash
# 8버전
git clone https://github.com/ksundong/docker-elk.git

# ELK_VERSION=7.10.0
# x-pack 애초에 설정 파일에 빠져 있음.
# password 등 중요 변수들 .env 에 따로 빠져있지 않고, config에 바로 적혀져 있음.
# NORI 셋팅 되어 있음.
git clone https://github.com/ksundong/docker-elk-kor.git

cd docker-elk
```

## 2. 환경 설정

- 기본적인 설정만 함.
- 싱글 노드

### docker-compose.yml

- ES_JAVA_OPTS 자바 메모리 사이즈 수정가능
- ELASTIC PASSWORD 등 변수 수정 가능. 비번 지우면 없이 가능.
    
    기본값은 .env 확인
    

### elasticsearch.yml

- xpack basic 버전(무료)으로 변경

```yaml
---
## Default Elasticsearch configuration from Elasticsearch base image.
## https://github.com/elastic/elasticsearch/blob/main/distribution/docker/src/docker/config/elasticsearch.yml
#
cluster.name: docker-cluster
network.host: 0.0.0.0

## X-Pack settings
## see https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html
#
# xpack.license.self_generated.type: basic # basic 무료 라이센스 이용
# xpack.security.enabled: true
```

### .env 파일

변수 설정 바꾸고 싶으면, .env 파일에 다음과 같이 추가하여 쓸 수 있음.

아래와 같이 디폴트 값으로 이미 설정되어 있는 버전도 있음. 최신버전

```bash
ELASTIC_VERSION=8.3.2

## Passwords for stack users## User 'elastic' (built-in)## Superuser role, full access to cluster management and data indices.# https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html
ELASTIC_PASSWORD='changeme'

# User 'logstash_internal' (custom)## The user Logstash uses to connect and send data to Elasticsearch.# https://www.elastic.co/guide/en/logstash/current/ls-security.html
LOGSTASH_INTERNAL_PASSWORD='changeme'

# User 'kibana_system' (built-in)## The user Kibana uses to connect and communicate with Elasticsearch.# https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html
KIBANA_SYSTEM_PASSWORD='changeme'
```

### nori 다운

Dockerfile 에 analysis-nori 추가

```bash
ARG ELASTIC_VERSION

# https://www.docker.elastic.co/
FROM docker.elastic.co/elasticsearch/elasticsearch:${ELASTIC_VERSION}

# Add your elasticsearch plugins setup here
# Example: RUN elasticsearch-plugin install analysis-icu
RUN elasticsearch-plugin install analysis-nori
```

## 3. 실행 등 명령어

### 1. 실행

```bash
sudo docker-compose build && docker-compose up -d
```


### 2. 접속

1. **elastic search**

http://127.0.0.1:9200/

- 디폴트
elastic / changeme 를 입력
    
