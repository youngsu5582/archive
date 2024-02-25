## 아티클 링크
https://techblog.woowahan.com/9478/

## 세줄 요약
- Auto Vacuum 이 호출되는 경우
- Vacuum 이 실패한다면?
- 결론

## 내용
### Auto Vacuum 은 언제 호출될까?
- Vacuum 동작을 DB단에서 임계치에 수행되는 것이 AutoVacuum
	- Dead Tuple 의 개수 누적치가 임계 도달할 때
	- Table 이나 Tuple age 가 누적되 임계치 도달할 떄
#### Dead Tuple 의 누적치가 임계치에 도달했을 떄
```
vacuum threshold = autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor * number of Tuples
```
- Dead Tuple 발생량이 위 공식 결과값을 넘으면 AutoVacuum 호출

- autovacuum_vacuum_scale_factor : 테이블 전체 Tuple 중 Dead Tuple 이 몇 % 차지할 때 Trigger 할 지 설정
- autovacuum_vacuum_threshold : 트리거 되기 전 테이블에 있어야 하는 최소 deat tuple 의 수
	( 너무 적은 deat tuple 일때도  , vacuum 이 발생하는걸 방지하기 위해 )
-> Default : scale_factor 는 20% , vacuum_threshold 는 50 rows
#### Table 이나 Tuple 의 age 가 누적되 임계치 도달했을 때

![500](https://i.imgur.com/0LY8sEA.png)
1. Table 의 age 가 autovacuum_freeze_max_age( default : 2억 ) 파라미터 임계치를 초과한 경우
2. vacuum_freeze_table_age < table age < autovacuum_freeze_max_age
	=> 테이블 age 가 위 두 파라미터 값 사이 속하며 해당 테이블에 vacuum 이 호출 되는 경우

- autovacuum_freeze_max_age : 해당 값 초과하는 age 테이블 대해 Anti Wraparound AutoVacuum 수행
	( AutoVacuum off 를 해도 강제로 수행 - 시스템 전체 중단 할 우려가 있으므로! )
	
- vacuum_freeze_min_age : 해당 값 초과하는 age 의 Tuple 들에 vacuum 작업 시 Transaction ID Freeze 대상
	-> 테이블 age 는 AutoVacuum 수행 이후 최대 vacuum_freeze_min_age 값으로 설정됨
	
- vacuum_freeze_tabe_age : 해당 값 초과하는 age 테이블에 대해 vacuum 호출 시 fronzen 작업도 같이 수행
	-> 다수 테이블들이 autovacuum_freeze_max_age 에 걸려 동시 Anti Wrapround AutoVacuum 수행 되기 전 , 
		그전 호출된 테이블이 돌도록 분산하는 효과

### Vacuum 관련 파라미터는 어떻게 튜닝할까

- 해당 요소는 DBA 적 관점이므로 , 지금은 생략

### Vacuum이 실패하고 있다면?
#### 1. Long transaction 이 수행되는 경우
- AutoVacuum 이 수행되고 있는 시점에 오래 수행중인 쿼리가 있는 경우
	-> Dead Tuple 은 해당 Tuple 들이 다른 Transaction 에서 참조하지 않아야 정리 가능한데 , 
		long transaction 수행 중 참조되는 Tuple 들 , commit 안된 idle 상태 transaction 인해
		Dead Tuple 정리 못하는 경우 발생할 수 있음!
```
1041608 are Dead but not yet removable, oldest xmin: 10674654, xmin 10674654 
```
	->1041608 이 처리하지 못함 ( 10674654 , 10674654 통해 )

=> long , not commited transaction 에 대한 모니터링이 필요하다!
- statement_timeout , idle_in_transaction_session_timeout 같은 파라미터로 쿼리 or 트랜잭션이 오래 방치되지 않게 예방
	- statement_timeout : SQL 문 실행되는 최대 시간 , 초과 시 타임아웃 오류 발생
	- idel_in_transaction_session_timeout : 트랜잭션 시작 후 idle 에서 허용되는 최대 시간 ,
		시간 동안 아무런 SQL 문 실행되지 않으면 트랜잭션 자동 종료 , 타임아웃 되고 rollback
#### 2. 미사용의 버려진 replication slot 이 있을 때
- Replication 방식 중 하나인 physical replication slot 사용하는 streaming replication 구성에서 발생 가능
- standby 서버에 복제 지연이 있거나 , standby server 가 down 되는 경우
	-> replication slot 은 복제 필요한 데이터 계속 담아두고 , 남겨진 slot 은 가지고 있는 데이터가 vacuum 정리되는 것을 방해함
#### 3. hot_standby_feedback = ON
- PostgreSQL Replication Architecture 에서 발생할 수 있는 이슈
- Master 에서는 Dead Tuple 로 확인해 vacuum 정리
			+ Replication 에서는 복제 지연 & Long Transcation 수행으로 여전히 데이터 필요할 수 있음
=> reflication conflict 발생! ( hot_standby_feedback 을 키는 이유 )

Replica 서버에서 수행한 쿼리와 현재 Replica 서버의 가장 오래된 트랜잭션 정보 -> Master 로 Feedback 을 줌
Master 는 해당 Dead Tuple 을 정리 못함!
( Aurora PostgreSQL 은 ON 이 default + 변경 불가능 )
## 결론

- PostgreSQL 만의 MVCC 때문에 vacuum 이라는 동작은 필수!
- vacuum 동작을 설정된 임계치를 통해 자동 수행해주는 것이 AutoVacuum
	- 임계치 이상 발생한 Dead Tuple 정리해 FSM 으로 반환
	- Transaction ID Wraparound 방지
	- 통계 정보 갱신
	- visibility map 갱신해 index scan 성능 향상

- PostgreSQL 이 강력한 개발 편의성을 제공하나 , Vacuum 이라는 허들 역시 존재함
=> 하지만 , 서비스에 맞는 적절한 스토리지 사용성이 중요해지는 지금 , Vacuum 이라는 허들로 배제하는 것은 너무 아쉽다!

## 참고 링크


##### Writed By Obisidan