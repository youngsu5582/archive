
### 영상 링크

https://www.youtube.com/watch?v=92NizoBL4uA

### What Is Caching?

- Temporary Location For Speed
데이터 원래 소스보다 더 빠르고 효율적으로 Access 가능한 임시 데이터 저장소

### Redis As Cache

- 단순 key - value 구조
- In-Memory 데이터 저장소 ( RAM )
- 빠른 성능 ( 평균 작업속도 < 1ms , 초당 수백만 건 작업 가능 )

### Caching Stragety - Read
#### Look-Aside
- Lazy Loading

Redis 에 원하는 값이 있다면?
- Cache Hit
바로 원하는 값 반환 

Redis 에 원하는 값이 없다면?
- Cache Miss
DB에 접근 -> DB에서 값 반환 -> Redis에 값 저장

Redis 가 다운되더라도 바로 장애로 이어지지는 않음
Cache 로 붙어있는 Connection 이 많을시?
-> Connection 이 모두 DB로 붙기 때문에 DB에 많은 부하가 몰릴 수 있음

DB에만 새로운 데이터를 저장했다면?
처음에 다량의 Cache Miss 발생!
##### Cache Warming

Cache Miss 를 방지하기 위해서
DB 에서 Cache 로 데이터를 밀어 넣어주는 작업

### Caching Stragety - Write
#### Write Around

DB에만 데이터 저장
Cache Miss 발생하는 경우에만 Cache 에서 데이터를 끌어옴
Cache 의 데이터와 - DB 의 데이터가 다를 수 있다
#### Write Through

Cache 에 저장 후 , DB에 데이터 저장
Cache 는 항상 최신 정보를 가지고 있다
저장할 때마다 항상 두 단계 스텝을 거쳐야 함
- 상대적으로 느릴 수 있음
- 데이터가 재사용 되지 않을 수 있는데도 캐시에 넣으므로 일종의 리소스 낭비도 가능
-> expire time 설정해주는게 좋음

### Redis Data Type

#### Strings
가장 기본적인 데이터 타입
set command 를 사용해 저장 시 , String 형태로 들어감
#### Bitmaps
Strings 의 변형
비트 단위 연산 가능

사용자의 멤버쉽 정보 ( 사용자 온라인 상태 , 광고 클릭 여부 , 이벤트 참여 여부 등 ) 표현할 때 유용

SETBIT online_users 1001 1
해당 online_users key 의 1001 번째 비트를 1로 설정

GETBIT online_users 1001
online_users key 의 1001 번째 비트 값 반환
#### List
데이터를 순서대로 저장
양쪽 끝에 데이터 추가 & 제거 가능
중복된 값도 허용

Queue , Stack 같은 자료 구조로 사용 가능
#### Hashes
Dictionary 처럼 , key - value 로 지정
하나의 key 안에 , 여러 개의 필드-밸류 쌍으로 데이터 저장
#### Set
중복되지 않은 문자열 집합
#### Sorted Set
Set 과 같으나 , score 라는 숫자 값으로 정렬
score 가 같을시? -> 사전 순 정렬
#### HyperLogLog
굉장히 많은 데이터 다룰 때 사용
12KB 고정 + 중복 탐지 가능
#### Stream
Log 를 저장하기 좋은 자료 구조

### Best Practice - Counting

#### String
- 단순 증감 연산
- INCR / INCRBY 등등
```redis
SET score:a 10

INCR score:a
11
INCRBY score:a 4
15
```
#### Bits
- 데이터 저장 공간 절약
- 정수로 된 데이터만 카운팅 가능
```
SETBIT visitors:20210817 3 1
0
SETBIT visitors:20210817 6 1
0
BITCOUNT visitors:20210817
2
```
3bit , 6bit 를 1로 만들고 , 1인 bit 개수 총 count

#### HyperLogLogs

대용량 데이터 카운팅 할 때 적절 ( 오차0.81% )
set과 비슷하지만 저장되는 용량이 매우 작음 ( 12KB 고정 )
저장된 데이터는 확인 불가능
```
PFADD crawled:20211024 "http://www.google.com"
1
PFADD crawled:20171124 "http://www.redis.com"
1

PFCOUNT crawled:20211024
8278423
PFMERGE crawled:all cralwed:20211024 crawled:20211023 . .
23905917
```
pfmerge 통해 , 두 key 에 중복되어 있는 값 제외하고 , 고유 요소 파악 가능

### Best Practice - Messaging
#### Lists
- Blocking 기능 이용해 Event Queue 로 사용
##### Client A
```
BRPOP myqueue 0
```
myqueue 에 값이 있을때 까지 blocking ( 불필요 polling 방지 가능 )
##### Client B
```
LPUSH myqueue "hi"
```
Client B가 값을 넣으면?
=> A 가 값을 받아서 작동
```
1) "myqueue"
2) "hi"
```

- LPUSHX 랑 RPUSHX 는 key 가 있을 때만 데이터 저장 가능
	( 이미 캐싱되어 있는 피드에만 신규 트윗 저장 가능 - 불필요한 리소스 낭비 방지 )
- 자주 사용 하는 사용자에만 미리 데이터 캐싱 가능
#### Streams
- 로그 저장 위한 자료구조
- append-only
- 시간 범위 검색 / 신규 추가 데이터 수신 / 소비자별 다른 데이터 수신(소비자 그룹핑)

```
XADD mystream * sensor-id 134 temperature 19.8
```

`*` 은 ID 를 Redis 에서 자동 지정 해주는 기능
mystream 에 key value , key value 로 저장

### Redis에서 데이터 영구 저장

Redis 는 In-memory 데이터 스토어
- 서버 재시작 시 모든 데이터 유실
- 복제 기능 사용하더라도 , 사람의 실수 발생 시 데이터 복원 불가능
- Redis 를 캐시 이외 용도로 사용한다면 적절한 데이터 백업 필요

=> AOF , RDB 두가지 방법 있음
```
set key1 a
set key1 apple
set key2 b
del key2
=>
key1 -> apple
```
#### RDB
key1 -> apple 만 저장
- snapshot
- 자동 : redis.conf 파일에서 SAVE 옵션 ( 시간 기준 )
- 수동 : BGSAVE 커맨드 이용해 , cli 창에서 수동으로 RDB 파일 저장
- 바이너리 파일 형태로 저장
#### AOF
set key1 a
set key1 apple
set key2 b
del key2
이렇게 진행 로그 저장
- Append Only File
- RDB 파일보다 더 큼
- 자동 : redis.conf 파일에서 auto-aof-rewrite-percentage 옵션 ( 크기 기준 )
- 수동 : BGREWRITEAOF 커맨드 이용해 , cli 창에서 수동으로 AOF 파일 재작성
- 레디스 프로토콜 형태로 저장
#### RDB VS AOF 선택 기준

- 백업 필요하지만 , 어느 정도 데이터 손실 발생해도 괜찮은 경우
	-> RDB 단독 사용
	-> redis.conf 파일에서 SAVE 옵션 적절하게 사용 ( SAVE 900 1 )
- 장애 상황 직전 까지 모든 데이터 보장되야 하는 경우
	-> AOF 사용 ( appendonly yes )
	-> APPENDSYNC 옵션 everysec 일 경우 최대 1초 사이 데이터 유실만 가능
- 제일 강력한 내구성 필요할 시
	-> RDB & AOF 동시 사용 가능

### Redis 아키텍처 선택 노하우

총 세가지 존재
Replication VS Sentinel VS Cluster

#### Replication
![200](https://i.imgur.com/DGh3DX4.png)

- Master 와 Replica 만 존재
- 단순한 복제 연결
- 비동기식 복제 ( 마스터가 복제본에 데이터가 잘 전달됐는지 확인 X )
- HA(High Availability) 기능 없으므로 장애 상황 시 수동 복구
	- replicaof no one ( 리플리카 노드에 직접 접속해 복제 끊어야 함 )
	- 애플리케이션에서 연결 설정 변경후 배포해야함
#### Sentinel
![200](https://i.imgur.com/HqbeALP.png)

- Master , Replica 외 Sentinel 존재
- Sentinel 노드가 다른 노드 감시
- 마스터가 비정상일 시 , 자동으로 페일오버 ( 기존 리플리카 노드가 마스터로 변경 )
- 연결 정보 변경 필요 X
- sentinel 노드는 항상 3대 이상 홀수로 존재
	( 과반수 이상 sentinel 이 동의해야 페일오버 진행 )
-> 자동 페일오버 가능한 HA 구성

![200](https://i.imgur.com/MM24BVd.png)

- 두대 서버에서 일반 레디스 & 센티널 함께 구동
- 최저 사양 서버에는 센티널 노드만 올려 구동
#### Cluster
![200](https://i.imgur.com/ddTxs3k.png)

- 키를 여러 노드에 자동 분할해 저장 ( 샤딩 )
- 모든 노드가 서로 감시 해 , 마스터가 비정상적일 시 자동 페일오버
- 최소 3대의 마스터 노드 필요

![350](https://i.imgur.com/DWqXLvM.png)

해당 분기가 완벽한 아키텍처 선택 기준

### Redis 운영 꿀팁과 장애 포인트

#### 사용하면 안되는 커맨드

Redis 는 싱글 스레드로 동작
( 한 사용자가 오래 걸리는 커맨드 실행 시 나머지는 요청 수행 불가능 + 대기 )

keys * -> scan ( 전체가 아닌 , 순회하며 사용)

Hash , Sorted Set 들에 100만 개 이상 저장 X 
- hgetall -> hscan ( 위 keys* 와 동일 이유 )
- del -> unlink ( 백그라운드에서 삭제 )

#### 변경하면 장애 막을 수 있는 기본 설정값

##### STOP-WRITES-ON-BGSAVE-ERROR = NO
- yes 가 default
- RDB 파일 저장 실패 시 , redis 로의 모든 write 차단
- 모니터링 적절히 한다면 , 꺼두는게 오히려 불필요한 장애 막는 방법
##### MAXMEMORY-POLICY = ALLKEYS-LRU
- redis 를 캐시로 사용할 때는 Expire Time 설정을 권장
- 메모리가 가득 찰 시에는 MAXMEMORY-POLICY 정책에 의한 키 관리
	- noevicition : default , 삭제 안함
	- volatile-lru : expire 설정이 있는 key 중에서만 lru 로 삭제
	- allkeys-lru : 모든 key 중에서 lru 로 삭제

#### Cache Stampede

TTL 을 너무 작게 했을때 발생하는 경우
- Duplicate Read , Duplicate Write 발생!
=> TTL 을 넉넉하게 하자

#### MaxMemory 값 설정

- Persistence / 복제 사용시 주의
- RDB 저장 & AOF rewrite 시 fork() 를 통한 자식 프로세스 생성
- Copy-on-Write 를 통해 , 메모리 두배 사용 하는 경우 발생 가능
=> Persistence / 복제 사용시 MaxMemory 는 실제 메모리의 절반으로 설정하자

#### Memory 관리

물리적 사용되고 있는 메모리 모니터링
- used_memory : 논리적으로 Redis 가 사용하는 메모리
- used_memory_rss : OS 가 Redis 에 할당하기 위해 사용하는 실제 물리적 메모리 양 ( 더 중요 )
=> 이 두개의 차이가 클 때 Fragmentation 이 크다고 말함
삭제되는 키가 많을 때 증가 
- 특정 시점 key 가 피크를 치고 다시 삭제되는 경우
- TTL로 인해 삭제가 과도하게 많은 경우
CONFIG SET activedefrag yes ( 단편화가 커질 때 키자 )
- 메모리 단편화가 발생 시 , 액티브하게 이를 감지하고 조각난 메모리 재배치 + 최적 메모리 상태 유지하려고 시도