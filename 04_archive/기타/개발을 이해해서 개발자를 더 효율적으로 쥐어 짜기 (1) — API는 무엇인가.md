---
tags: 기타
---
## 아티클 링크
https://medium.com/@tw.kim913/%EA%B0%9C%EB%B0%9C%EC%9D%84-%EC%9D%B4%ED%95%B4%ED%95%B4%EC%84%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%A5%BC-%EB%8D%94-%ED%9A%A8%EC%9C%A8%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%A5%90%EC%96%B4-%EC%A7%9C%EA%B8%B0-1-%EC%84%9C%EB%A1%A0-api-a8f4fae87487

## 요약
- 개발자와 기획자 사이 커뮤니케이션은 매우 어렵다
- 정해진 규격인 API 를 통해 소통하자!


## 내용

### 개발자는 왜 그럴까?
- 개발에 대해 잘 모르는 인사팀 & 기획팀이 개발자에게 어떤 기능을 만들어 달라고 요구!

- 열정 없고 게으른 김우태씨는 "설계가 상당히 복잡할 것 같아서 상당한 검토가 필요하다."는 핑계를 대며 1주일을 놀면서 보냄
	=> 이럴때 어떻게 혼을 내야 할까? ( 정확한 설계를 해서 요청을 하자 )

### API?
- Application Programming Interface 의 약자
- 표현 상태 전이 응용 프로그램 프로그래밍 인터페이스 ( 역자주 : 물론 REST API 이나 , 간단한 개발 관련 개념 설명을 위해 API 와 REST API 동치 )
=> 매우 딱딱한 느낌


#### 컨테이너에 비유를 해보자
- 컨테이너는 1940 년대 발명 된 물류 시스템의 혁명
- 컨테이너 박스로 인해 물류 가격 인하 및 효율적
- 핵심은 컨테이너의 **'표준화'**!
	-> 규격을 정확히 규정 ( 40ft 표준 컨테이너의 크기? : 40 * 8 * 8'6'' 고정 )

##### EX ) 프랑스 -> 일본 와인 수출
프랑스 사람 : 일본은 운송에 기차가 아닌 트럭을 쓴다고 가정 시 , 기차 화물 상자가 트럭에 실리는가?
일본 사람 : 프랑슨느 와인 운송에 나무 상자를 쓴다고 하는데 , 세관에서 트집을 잡지는 않을까?
=> "사용 화물 트럭 규격은?" , "운송용 상자 나무라 들었는데 다른 상자로 교체 가능?" 등 의미없는 일에 담당자들이 붙어야 함

그렇기에 , 컨테이너 박스가 문제 해결 ( 모든 항구는 컨테이너라는 한 종류 화물만 취급하면 되기 떄문에 )

### In Server
- 서버도 다양한 언어의 서버 프로그램 포함
	-> 서로 다른 통신 방식과 데이터 형식을 쓰는 서버 통신은 매우 골치 아픔 ( 프랑스 와 일본의 사례 )
- HTTP 는 인터넷에서 가장 자주 쓰이는 보편적 통신 방식 + 모든 서버가 HTTP 통신 방식 지원
	-> 그렇기에 , 개발자들은 HTTP 를 써서 서버와 통신하는 표준 방식을 API 로 이름 지음 ( 즉 표준화 )


- API 를 사용하는 유저는 요청하는 주소 ( domain + path ) , 요청 값 ( param ) 을 통해 API 호출 , 응답을 받음

[http://api.dictionaryapi.dev/api/v2/entries/en/digital](http://api.dictionaryapi.dev/api/v2/entries/en/digital) : 단어 조회 API 주소

domain : http://api.dictionaryapi.dev
path : api/v2/entries/en/digital ( param 값이 path 에 포함 )
param : digital

![](https://i.imgur.com/mlt8JmP.png)
-> 이상한 글자들이 나옴 ( 제대로 나온 Response! )
- 웹 브라우저를 통해서 API 요청을 보내고 성공적으로 응답을 받음!
- 이렇게 , 보기 힘들기에 보통 IT 회사들은 API 와 함께 API 문서를 만들어서 사용하는 법을 설명

### 놀자 어때 프로젝트 검토
- 아고다와 익스피디아 같은 사이트에서 호텔 목록을 가져와서 네이버에 싼 호텔 리스트를 제공!

-  아고다와 익스피디아읭 API 문서를 찾아서 , 해당 API 호출 후 서버에 가져오고 새로운 API 를 만들어서 네이버에 리스트를 가져가게 하자!

#### 개발자에게 요청

- 김우태 씨! 네이버에 제공할 아고다 , 익스피디아 API 문서 링크 찾아왔어요! API 를 통해 데이터를 가져와서 호텔 리스트를 만들어 주세요!
![](https://i.imgur.com/KPJX0u6.png)

- 해당 그림 대로 해주시고 , 놀자어떄 API 주소는 'http://[noljaoudea](http://www.nolzaedea/).com/api/v1/hotel/list' 로 ,
  param 값으로 호텔 예약 날짜나, 호텔 위치도 받게 설계 해주시고 , 
  네이버에서 우리 API 를 사용하는데 필요할 테니까 API 문서도 함께 작성해 주세요!


## 결론
- 서로 서로 구체적인 요구 지시 및 수용 가능!
- 개발자도 개발에 집중 할 수 있기에 더욱 편함



## 참고 링크



##### Writed By Obisidan