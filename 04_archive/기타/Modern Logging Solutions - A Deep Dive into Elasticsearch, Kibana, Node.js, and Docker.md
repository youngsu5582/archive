---
tags:
  - 기타
아티클_링크: https://medium.com/@hussainghazali/modern-logging-solutions-a-deep-dive-into-elasticsearch-kibana-node-js-and-docker-cd530e518c25
아티클_작성자: Hussain Ghazali
---
## 내용

### 서론

- 현재 소프트웨어 개발에서 , 로깅 과 모니터링은 매우 중요한 요소가 되어버림 ( 신뢰성 , 성능 , 보안 효과 위해 )
	=> 로깅 과 모니터링을 통해 실제 환경에서 어떻게 동작하는지 이해 + 문제 신속하게 발견 및 해결 가능
- 해당 글에서는 , 로깅과 모니터링 영역에서 중요한 역활을 하는 도구들을 소개한다.
( Node.js , Docker , Kibana , Elasticsearch )

### 로깅의 필요성

로깅 과 모니터링 실천의 필요성은 과소평가 할 수 없는 반 필수적 존재
애플리케이션의 Health Check & Performance 평가 위한 필수적 이해

#### 1. 실시간 이슈 감지
- 로깅 과 모니터링 도구 통해 개발자는 실시간 이슈 & 이상사항 감지 가능
- Log , Monitoring Metric 검토해 문제 신속 발견 & 해결 가능

#### 2. 성능 최적화
- 모니터링 통해 , 성능 최적화 가능
- 응답 시간 ( Response Time ) , 리소스 사용량 ( Resource Utilization ) , 성능 지표 ( Performance Indicator ) 통해 활용
#### 3. 보안과 준수

- 로깅을 통해 , 보안과 산업 규정 준수에 도움을 받음
- 사용자 활동을 추적해 감시 + 산업 규정에 준수하는지 확인

#### 4. 선제적 문제 해결

- 포괄적 로그 & 실시간 모니터링을 통해 , end-user ( 궁극적 제품 사용자 ) 가 사용하기 전 문제 식별 가능
-> 문제 심각성 & 영향 최소화

#### 5. 사건 이후 분석
- 다양한 사고 ( 트래픽 폭발 , 보안적 이슈 , 기능적 이슈 등 ) 후 , 분석은 매우 중요
-> 원인 이해 & 미래 유사 문제 방지

#### 로깅의 단점?
1. Log Overload : 방대한 양의 로그 데이터가 생성되므로 , 적절한 도구로 파악해야함
2. Data Storage : 로그 & 모니터링 데이터를 효율적이고 안전하게 저장 해야함 ( 용량 문제 , 보안 문제 )
3. Integration : 로그 시스템을 응용 프로그램 과 원할하게 통합해야함


### Elasticsearch

- Distribute ( 분산 ) + RESTful search & analytics engine
- 대량의 데이터를 빠르게 색인화 하고 검색하기 위해 설계
- 전문 검색 , 실시간 데이터 분석 & 데이터 시각화 에 특화
=> 로그 데이터 색인화 하고 쿼리 하는데 이상적

### Kibana

- Elasticsearch 의 Frontend ( 화면 )
- 오픈 소스 데이터 시각화 & 탐색 도구
- 사용자 친화적 인터페이스로 데이터 탐색 과 시각적으로 대쉬보드 만들어줌

### 통합

- 협력하여 , 동적 생태계 생성 가능
- Elasticsearch 가 로그 및 모니터링 데이터 효율적 수집 , Kibana 가 상호 작용 인터페이스 제공
- Real-time Monitoring ( 실시간 모니터링 )
- Scalabilty
- Custom Dashboards
- Search and Analysis

### Setup Elasticsearch , Kiban with Docker

##### 1. Dokcer Install
- [Docker’s official website]([https://www.docker.com/get-started](https://www.docker.com/get-started))
- 도커 공식 홈페이지 참조
##### 2. Pull Elasticsearch & kibana Image
```cmd
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.14.0  
docker pull docker.elastic.co/kibana/kibana:7.14.0
```

- 뒤 버전 숫자는 해당 아티클 작성자가 지정한듯 ( latest 로도 가능 )
##### 3. Create a Docker Network
```
docker network create elastic
```
- kibana - elastic 연결 위한 network 생성
##### 4. Start Elasticsearch Container
```
docker run - name elasticsearch - network elastic -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" 
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" docker.elastic.co/elasticsearch/elasticsearch:7.14.0
```

- 이름 , 네트워크 지정
- 포트 9200 : 9200 , 9300 : 9300 Mapping
- ENV 설정
- 실행할 Image 지정
##### 5. Start Kibana Container
```
docker run - name kibana - network elastic -d -p 5601:5601 -e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" docker.elastic.co/kibana/kibana:7.14.0
```
- 이름 , 네트워크 지정 
- 포트 5601 : 5601 Mapping
- ENV 설정
- 실행할 Image 지정

##### 6. Access Kibana & 7. Explore and Configure
- http://localhost:5601 에 접속하여 확인
![](https://i.imgur.com/aglJiiE.png)
- 구동시 나오는 화면

##### 번외
```yml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
  kibana:
    image: docker.elastic.co/kibana/kibana:7.14.0
    ports:
      - 5601:5601
    environment:  
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
```
- 위 docker setting 직접 compose 로 수정

##### Writed By Obisidan