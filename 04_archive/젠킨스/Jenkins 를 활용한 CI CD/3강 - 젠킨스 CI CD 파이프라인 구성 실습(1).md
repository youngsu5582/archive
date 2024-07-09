---
tags:
  - 젠킨스
강의명: "[토크ON세미나] Jenkins를 활용한 CI/CD"
강의_링크: https://www.youtube.com/watch?v=GOLHN3FHjpI
---
### Jenkins 설치

강사는 EC2 에 직접 설치하는 식으로 했으나 , 
docker-compose 를 통한 자동 Pulling 으로 설치

```docker-compose
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - 8080:8080  # Jenkins 웹 인터페이스에 사용됩니다.
      - 50000:50000 # Jenkins 에이전트 연결에 사용됩니다.
    volumes:
      - jenkins_data:/var/jenkins_home # Jenkins 데이터 보존을 위한 볼륨
    environment:
      - JENKINS_OPTS=--httpPort=8080

volumes:
  jenkins_data: {}

```


### AWS EC2 설정

![](https://i.imgur.com/soQNAoR.png)


- 단순 테스트 용이므로 , 
- AMI 프리티어 용으로 생성

![450](https://i.imgur.com/sFWEF7f.png)

해당 기능을 통해 키 페어 생성
=> 생성한 키 ( In Macbook 기준 ) ~/.ssh 로 이동


![450](https://i.imgur.com/bn3VqZ7.png)

Instance 에서 퍼블릭 IP 확인

.ssh 에서 config file 에 해당 Option 추가

```cmd
Host test_jenkins
    User ec2-user
    HostName 52.79.233.177
    IdentityFile ~/.ssh/jenkins_test_key.pem
```


![450](https://i.imgur.com/DBPHbhS.png)

보안 그룹 - 인바운드 규칙에 8080 Port 추가

https://github.com/frontalnh/temp

해당 코드 git clone 후 , 자신에게 맞게 rm -rf .git + repo 명으로 변경 후 git init


### User

1. 사용자 등록
![450](https://i.imgur.com/Iz4C7XD.png)

2. 권한 등록
![450](https://i.imgur.com/JuyWMk8.png)

![450](https://i.imgur.com/aL4Igkh.png)
-> 이 부분에서 , 액세스 키 만들기를 통해 Key 생성 ( CLI )

### Credentials

![](https://i.imgur.com/DOpb4kQ.png)

해당 부분에서 Domains 에 있는 global 클릭 후 -> Add Credentials

![450](https://i.imgur.com/3N16pDz.png)


- ID 에는 Jenkins 에서 사용할 임의 값
- Password 는 받은 Key 값
- Username 에는 사용자 명

![450](https://i.imgur.com/2JWuWR5.png)

세개 입력 완료


### S3

![450](https://i.imgur.com/GQSKSIe.png)

- 모든 퍼블릭 액세스 차단 버튼 해제

### Emial Send - Jenkins

![450](https://i.imgur.com/1sjynWW.png)

- Use TLS 도 체크
- 2차 비밀번호 설정 + 2차 비밀번호로 넣어줘야 함 ( 계정 비밀번호 X )



현재 , Docker 와 Git 을 사용하므로 해당 Plugin 도 깔아줘야 한다 ( Git 은 자동 설치 해주는듯 )

Manage Jenkins -> Plugins -> Available Plugins 에서 검색 후 설치

( Docker , Docker pipeline )


![450](https://i.imgur.com/7f07VIS.png)



### Item 생성

1. Pipeline 선택

- Github Project Check -> Project URL 기입
- Poll SCM Trigger Check

2. Pipeline Script 로 , 기존 Script 삽입

깃허브 단위에서 올리는 Jenkins File 관리하고 싶으면
Pipeline Script from SCM