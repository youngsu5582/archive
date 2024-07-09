---
tags:
  - 젠킨스
강의명: "[토크ON세미나] Jenkins를 활용한 CI/CD"
강의_링크: https://www.youtube.com/watch?v=3WZoVkvLE4A
---
### 개발 환경의 종류

- 개발자가 개발 하는 Local 환경 
-  개발자들끼리 개발 내용 대해 통합 테스트 하는 Development 환경
-  개발 끝나고 QA 엔지니어 & 내부 사용자들이 사용해보기 위한 QA 환경
-  실제 유저가 사용하는 Production 환경

=> 쉽게 , DEV & QA & PROD 환경

### 개발 프로세스

1. 개발자가 자신 PC 에서 개발 진행 ( Pre-commit 역시 가능 )
2. 다른 개발자가 작성한 코드와 차이 발생하지 않는지 내부 테스트 진행
3. 진행 내용을 다른 개발자와 같이 공유 위해 SCM 에 올림 ( 일반적으로 , DEV Branch )
4. DEV branch 내용을 개발 환경 배포 전 Test & Lint 등을 통해 코드 포맷팅
5. 배포하기 위한 빌드과정 거침
6. 코드 배포
7. 테스트 진행
8. 위 모든 과정을 DEV , QA , PROD 환경에서 모두 하고 , 각각 맞는 환경에 배포


=> 결국 , 인프라를 모듈화 해 변수만 상황에 맞게 잘 설정하고 설계하자!
( DB Password , Server URL 등등 )

- 인프라별 키 관리를 위해 aws system manager 나 parameter store 같은 키 관리 서비스를 사용하자!
	( 요새는 개발도 , 클라우드 리소스에서 사용하므로 )

### 배포 환경 ( 예시 )

1. 웹사이트 코드 작성
2. 웹사이트 코드를 린트 , 웹팩 빌드 해서 AWS S3 Bucket 에 html 파일 업로드
3. Node.js 백엔드 코드를 Typescript 로 작성
4. 위 코드를 JavaScript Compile 하여 , 테스트 코드를 돌려서 도커 이미지를 만들어 ECR 에 올린다.
5. 업로드한 ECR 이미지로 ECS 서비스를 재시작


### ECS

- Elastic Container Service
- 도커 컨테이너 기반으로 서비스 운용 가능하게 해주는 간단 서비스
- 무중단 배포 ( rolling update ) 제공 + scale up 이 가능
- LB 가 이들 사이 밸런싱 해줌
=> ECS or k8s 통해 rolling & deploy 되므로 , jenkins 는 그냥 배포 명령만 내려주면 된다.




DEV 용  , PROD 용 AWS 계정 애초에 분리

![500](https://i.imgur.com/eig0gUn.png)
