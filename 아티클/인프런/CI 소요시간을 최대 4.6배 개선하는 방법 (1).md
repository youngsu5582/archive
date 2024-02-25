## 아티클 링크
https://tech.inflab.com/20231101-optimizing-ci-pipeline/#yarn-%EA%B3%BC-pnpm-%EB%B9%84%EA%B5%90

## 세줄 요약

- CI 에서 install 로 인한 시간 증가
- Jenkins Pipeline 최적화
- 몹시 어려움
### 서론

- CI 파이프라인이 너무 오래 동작해 PR 리뷰에 영향을 받은 적 있나요?
- 프로젝트 규모가 커질수록 늘어나는 CI / CD 파이프라인 소요 시간에 고민해본 적이 있나요?

인프런 역시 이런 고민을 가졌고 , 소요시간을 최대 4.6배 까지 개선을 해나갔습니다.

### 최적화의 배경
![450](https://i.imgur.com/ST50H58.png)

모노레포를 도입함에 따라 , 패키지 단위로 코드를 관리하고 있었습니다.
프로젝트의 규모가 커질수록 코드 베이스 와 패키지가 늘어나며 CI 시간이 급증했습니다.
( 코드 사용 라이브러리가 많아지고 , 의존성 설치 소요시간 역시 증가 )

빌드 - 테스트를 순차 진행하여 소요시간이 병렬적으로 증가해
빌드 파이프라인에 직접 병렬 시도등의 시도를 했으나 , 
NPM 의 자랑이자 단점인 의존성으로 인해 복잡한 관계와 동시성 문제가 발생했습니다.

=> pnpm 과 turborepo 를 도입해 프로젝트 생상선을 개선하게 되었습니다.

### yarn vs pnpm

![450](https://i.imgur.com/7od4iFH.png)
( pnpm , Node 20 버전 테스팅 결과)

그래프에 보이는 것 처럼 , yarn 과 비교해서도 2배 가량 빠른 모습을 보여줍니다.

##### yarn
- 의존성 설치 소요 시간 : 최대 120 초
- 복잡한 node_modules 구조 인해 디스크 I/O , CPU 자원 소모
##### pnpm
- 의존성 설치 소요 시간 : 최대 63초
- 심볼릭 링크 통해 전역 캐시 ( .pnpm-store ) 에 링크 -> 불필요한 파일 복사 감소 -> 디스크 I/O 감소

```text
node_modules
└── .pnpm
    ├── bar@1.0.0
    │   └── node_modules
    │       └── bar -> <store>/bar
    │           ├── index.js
    │           └── package.json
    └── foo@1.0.0
        └── node_modules
            └── foo -> <store>/foo
                ├── index.js
                └── package.json
```

.pnpm-store 에 모든 의존 패키지를 설치하고
패키지 내의 디렉토리는 심볼릭 링크만 설정하는 식으로 개선해
디스크 쓰기가 감소하고 , 의존성 설치 소요시간을 감소시킵니다

### turborepo

터보레포를 설명하기 전 모노레포에 대해 간단히 설명하겠습니다.
##### monorepo
![450](https://i.imgur.com/bKVSXdd.png)

폴리 레포는 , 여러개의 프로젝트를 만들어 사용하는 방식입니다.
이는 , 수많은 저장소를 만들고 관리해야 하는 불필요한 시간을 투자하게 했습니다.

모노레포는 관계가 잘 정의된 여러 개의 개별 프로젝트를 포함한 저장소입니다.
모노레포 도입시 , 매번 수많은 저장소를 만들고 관리하는 노력을 줄일 수 있습니다.

하지만 , 모노레포의 코드 베이스 규모가 커지면 어떻게 될까요?

- 순차적 빌드와 테스트 실행으로 인한 소요시간 증가
- 변경되지 않은 부분도 불필요한 빌드 & 테스트 반복적 실행

```groovy
stage('Docker Build') {
    parallel {
        stage('Package A') {
            when {
                anyOf {
                    changeset "cmd/package-alpha/**/*"
                    changeset "docker/package-alpha/**/*"
                }
            }
            steps {
                sh 'docker build -t package-alpha -f docker/package-alpha/Dockerfile .'
            }
        }

        stage('Package B') {
            when {
                anyOf {
                    changeset "cmd/package-beta/**/*"
                    changeset "docker/package-beta/**/*"
                }
            }
            steps {
                sh "docker build -t package-beta -f docker/package-beta/Dockerfile ."
            }
        }
```

이렇게 병렬적으로 실행을 할 수도 있으나
결국 꽤나 큰 공수가 들어갑니다.

- 패키지가 추가될 때 마다 사람이 CI 파이프라인을 수정해야 합니다.
	-> 매우 번거로운 일 + 휴먼 에러 발생 가능
- 패키지가 서로 복잡한 의존 관계로 얽혀 있을 시 , 모두 CI 파이프라인에 구현하기 어렵습니다.
	-> 병렬은 전부 다같이 실행 되므로 , 단계를 전부 지정해줘야함.

=> 이럴때 필요한 게 , turborepo , nx 같은 모노레포 도구들입니다.

####  turborepo?

https://turbo.build/repo

turborepo 는 JavaScript & TypeScript 코드를 위한 고성능 빌드 시스템입니다.
- Incremental Builds
- Content-aware hashing
- Parallel execution
- Remote Caching
- ... 기타 등등 ( Zero runtime overhead , Pruned subsets , Task pipelines )

##### Incremental Builds

단순 번역하면 , 증가 빌드 입니다.
늘어난 부분만 , 변경한 부분만 테스트 & 빌드 하여 CI 소요시간을 크게 줄여줍니다.

##### Content-aware hashing

코드 내용을 구분 & 식별하여 해시값으로 코드 변경을 감지합니다.
 ( Git 에 의존하는 changeset 과 대조적입니다. )

##### Parallel Execution

기존에는 빌드를 순차적 실행하므로 CPU 자원이 낭비 됐습니다.
빌드 병렬 실행해 CPU 코어를 효율적으로 사용하고 빌드를 더 빨리 끝낼 수 있습니다.
![400](https://i.imgur.com/p9BVYdI.png)

##### Remote Caching

- Local Computation Caching : task 처리 결과를 로컬에 보관 & 재사용 하는 기능
	( 같은 머신에서는 같은 형상 빌드와 테스트를 두 번 이상 하지 않습니다. )

- Distributed Computation Caching :
	- Local Computing Caching 의 확장한 개념
		task 처리 결과를 한 머신이 아닌 여러 환경 걸쳐 공유
	- CI 에이전트 중 하나가 이미 빌드 & 테스트 했을 시 , 다른 에이전트는 절대 이를 재실행 X

Remote Caching 은 두번째 Distributed Computation Caching 과 유사한 개념입니다.

같은 코드 베이스를 수정하는 작업자가 여러명 있다고 가정
- 작업자 A 가 먼저 빌드 테스트 수행 이력이 있을 시 , 해당 결과는 공용 캐시 서버에 저장합니다.
- 작업자 B 가 빌드 와 테스트 실행할 때 , 공용 서버에 캐시된 결과를 불러주고 , 불필요한 빌드 실행을 건너뜁니다.

공용 캐시 서버에는 빌드 결과 파일 또한 같이 저장하므로 , 즉시 배포에 활용이 가능합니다.
( 반복적 작업이 여러 번 수행되는 CI 파이프라인을 효과적 개선 가능 )

### CI 소요시간 최적화

인프랩에서는 젠킨스를 이용해 , CI / CD 파이프라인으 구축했습니다.

CI Job 이 트리거 되면 , EC2 에이전트에서 도커 이미지를 실행해 다음 과 같이 순서로 Job을 실행합니다.
![](https://i.imgur.com/ujS3zaw.png)

#### 의존성 설치
```shell
pnpm i --frozen-lockfile
```

- 필요한 의존성 라이브러리 버전 확인 ( frozen-lockfile 을 통해 의존성 일관성 유지 )
- 패키지 저장소에서 의존성 다운로드 & 설치
- post-install 스크립트 실행

의존성 설치 소요시간에서 60초 가량을 단축하고 
CI 환경에서 의존성 캐시를 도입하면 추가적인 시간을 절약 가능합니다.

#### 환경별 의존성 설치

로컬 환경에서는 pnpm i 를 실행하면 
전역 의존성 캐시 디렉토리에서 캐시 로드해 불필요한 설치를 건너뛰나 ,
CI 환경은 실행 서버가 매번 달라지므로 , 별도의 의존성 캐시를 설정해야만 합니다.

#### CI 환경에서 의존성 캐싱
1. 압축 : node_modules 경로를 tar & gzip 으로 압축
2. CI 용 캐시 이미지 만들기 : 의존성 미리 설치한 도커 이미지를 만들어 빌드에 활용
3. 캐시 볼륨 마운트 : build agent 에 남아 있는 global cache 활용

##### 압축
```yaml
- id: cache
      uses: actions/cache@v3
      with:
        path: '**/node_modules'
        key: ${{ runner.os }}-node-${{ hashFiles(format('**/{0}', steps.hash-file.outputs.hash-file)) }}
```

의존성이 담긴 node_modules 디렉토리를 tar 로 묶고 , gzip 으로 압축하는 방법입니다.

압축 방법은 생각보다 여러 단점이 발생할 수 있습니다.

1. 캐시는 pnpm-lock.yaml 같은 Manifest 파일 해시로 구분되므로
	필요 의존성이 변경될 때 마다 새로 설치를 진행해야 합니다.
2. node_modules 디렉토리를 압축하는 과정은 기본적으로 싱글 스레드로 동작해 시간이 많이 소요됩니다.
3. 프로젝트 규모가 커질 수록 디렉토리가 복잡해지고 , 캐시 파일 다운 & 압축 해제에 오히려 시간이 더욱 소요됩니다.

=> 즉 , 프로젝트 규모에 따라서 압축 방법을 선택해야 합니다.

```yaml
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'pnpm'
    - name: Install dependencies
      run: pnpm install
```

또한 , Github Action 에서 이렇게 cache : pnpm 을 통해서도 캐싱이 가능합니다.

##### CI용 캐시 이미지 만들기

CI 용 캐시 이미지를 만드는 방법입니다.
```dockerfile
FROM node:18-slim
# pnpm 설치과정 생략

# pnpm fetch does require only lockfile
COPY pnpm-lock.yaml ./

RUN pnpm fetch --prod
```

의존성 설치 전 이미지를 pull 받아서 , 의존성을 설치하면 캐시를 활용해 바로 빌드할 수 있습니다.

```bash
docker build -t rallit-frontend-dep:02d93ab24 -f dep.Dockerfile .
docker push rallit-frontend-dep:02d93ab24

# when build
docker run -i --rm -v $WORKSPACE:/workspace rallit-frontend-dep:02d93ab24 'pnpm i --frozen-lockfile'
```

이미지 태그로 매니페스트 파일 ( pnpm-lock.yaml ) 해시값을 지정하며 
앞선 방법과 마찬가지로 해당 파일 변경 시 다시 빌드 해야 합니다.

즉 , `pnpm fetch --prod` 명령어로 설치한 의존성을 이미지 빌드하고 , pull 받는 과정에서 오버헤드가 발생합니다.
이 역시도 , 단순 install 보다 더욱 시간이 소요됐습니다.

##### 캐시 볼륨 마운트

같은 프로젝트 더라도 서로 다른 브랜치 와 PR 간에 의존성 캐시를 공유할 수 없다는 것을 의미하며
새로운 PR이 만들어질 때마다 의존성을 처음부터 설치해야 한다는 단점이 있었습니다.

###### 기존 ( AS-IS )

- CACHE_PATH 가 JENKINS_WORKSPACE 에 위치
- 서로 다른 프로젝트 , 브랜치 , PR 사이 의존성 캐시 공유 불가

```bash
# 경로 비교
JENKINS_WORKSPACE=/workspace/rallit_frontend_PR_15242
CACHE_PATH=/workspace/rallit_frontend_PR_15242/.pnpm-store
```

#15242 CI 잡이 완료될 시?
-> /workspace/rallit_frontend_PR_15242/.pnpm-store 여기에 남아 있습니다.
#15243 CI 잡이 게시되면?
-> /workspace/rallit_frontend_PR_15243/.pnpm-store 이므로 의존성 캐시를 전혀 사용할 수 없습니다.

###### 변경 ( TO-BE )

- 에이전트 내 Global Dependency Cache 경로 설정
- 같은 에이전트에서 수행하는 빌드는 Dependency Cache 활용

```bash
# 경로 비교
JENKINS_WORKSPACE=/workspace/rallit_frontend_PR_15242
CACHE_PATH=/tmp/.pnpm-store	
```

모든 CI 작업 실행하는 컨테이너의 pnpm store-path 옵션을 동일 경로로 지정합니다

즉 #15242 , #15243 모두 같은 경로를 바라보게 됩니다
이렇게 되면 , 설치한 의존성을 재활용하게 되어 60초의 의존성 설치를 소요시간 3초로 단축하게 됩니다

```dockerfile
# set global node cache directory
ARG CACHE_PATH=/cache/node
RUN yarn config set cache-folder ${CACHE_PATH}/.yarn
RUN pnpm config set store-dir ${CACHE_PATH}/.pnpm-store
```

Dockefile 에서 이렇게 경로를 적용해 줍니다.

```groovy
env.DOCKER_OPTION = '-v /tmp:/cache/node -e CI=1'

# (중략)

stage('Install') {
  steps {
    script {
      sh "docker run --rm -i -v $WORKSPACE:/workspace $DOCKER_OPTION $NODE_IMAGE 'pnpm i --frozen-lockfile'"
    }
  }
}
```

해당 부분은 Jenkins 이나 , 핵심은 단순 Docker 를 통해 , 실행하고 설치하는 것



## 결론

솔직히 해당 부분은 잘 모르겠다.
핵심은 시간을 줄여야 하는거나 , 프로젝트 규모와 상황에 맞게 선택을 잘 해야 한다.
Jenkins 에 대해 더욱 알아봐야 , 해당 내용에 대해 이해가 가능할 거 같다.

##### Writed By Obisidan