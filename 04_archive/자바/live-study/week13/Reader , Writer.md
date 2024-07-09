---
tags:
  - 자바
추가_정보: 백기선 라이브 스터디
주차: "13"
---
### Reader , Write

바이트 기반 입출력 스트림 단점을 보완하기 위해 문자기반 스트림 제공
-> 문자 입출력 때는 문자 기반 스트림을 사용하자

- InputStream -> Reader
- OutputStream -> Writer

단순히 2byte 스트림을 처리하는 것뿐만 X
-> 여러 종류 인코딩 과 자바에서 사용하는 유니코드 간 변환 자동적 처리

Reader 는 readLine() 메소드 제공
Writer 는 println() 메소드 제공
