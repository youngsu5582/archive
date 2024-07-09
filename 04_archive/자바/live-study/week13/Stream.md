---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "13"
---
### Stream

단방향 통신
-> 입력 과 출력을 수행 위해선 두개 필요

먼저 보낸 데이터를 먼저 받음 - 중간 건너뜀 없이 데이터 주고 받음
-> FIFO

#### Input/Output Stream

바이트 단위로 데이터 전송

- File(Input/Output)Stream : 파일
- ByteArray(Input/Output)Stream : 메모리 ( byte 배열 )
- Piped(Input/Output)Stream : 프로세스 ( 프로세스간 통신 )
- Audio(Input/Output)Stream : 오디오 장치

=> 어떠한 대상에 대해 작업 할 지에 따라 선택 가능

Sytem.in 은 InputStream 사용
System.out 은 OutputStream 사용
#### Method

##### Input

abstract int read()
int read(byte[] b)
int read(byte[] b, int off, int len)
##### Output

abstract void write(int b)
void write(byte[] b) : 매개 변수 받은 바이트 배열 저장
void write(byte[] b, int off, int len) : 매개 변수 받은 바이트 배열 특정 위치 부터 지정 길이 만큼 저장
void flush() : 버퍼에 쓰려고 대기하는 데이터 강제 쓰게 함
void close() : 쓰기 위해 연 스트림 해제

#### 보조 Stream

스트림 기능을 보완하기 위해 제공
실제 데이터 주고받는 스트림 아니기 때문에 데이터 입출력 기능 X
-> 스트림 기능 향상 시키거나 새로운 기능 추가 가능
=> 데코레이터 패턴을 활용

```java
FileInputStream fileInputStream = new FileInputStream("test.txt");
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

기반 스트림을 통한 보조스트림 생성 가능
-> 실제 입력 기능은 연결된 fileInputStream 이 수행
##### Buffer

입출력 스트림에서 데이터를 일시적으로 저장하는 메모리 영역
-> 입출력 동작이 수행될 때 마다 바로 전송이 아니라 , 모아두었다가 일정량 쌓이면 한 번에 전송

- 입출력 성능 향상 : 데이터 한 번에 모아 입출력 -> 시스템 콜 횟수가 줄어드므로 성능 향상
- 데이터 일관 처리 : 버퍼 사용해 데이터 일정량 모아 처리 -> 입출력 일괄 처리 가능
- 프로그램 응답 시간 감소 : 입출력 동작 적게 발생하므로 프로그램 응답 시간 역시 감소


