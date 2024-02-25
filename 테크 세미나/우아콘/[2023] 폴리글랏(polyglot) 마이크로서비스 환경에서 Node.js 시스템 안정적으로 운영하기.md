### 테크 링크

https://www.youtube.com/watch?v=Z0d7ZrxY-i0

### 폴리글랏?

- Polyglot
- 다양한 언어를 알고 , 이로 구사를 할 수 있음

![500](https://i.imgur.com/lugL5gV.png)

수많은 언어로 이루어진 MSA

Java-Spring 에서 가이드라인을 통해 , 최소한의 품질이 지켜지게 만듬

-> JavaScript-Node 에서도 해당 최소 품질을 통과해야만 새로운 개발환경으로 추가 도입이 가능했다!

### Observability

 ![450](https://i.imgur.com/GQUReby.png)
현재 , 배민의 Log System 은 매우 복잡하다

![500](https://i.imgur.com/txsaR6j.png)

에러를 Slack 에 배포후 , 클릭시 에러 코드 및 상세정보로 이동하게 해주는 고도화 기술 도달!
-> 이를 , NodeJs에서 전부 다 새로 구축해야한다면? ( 매우 비효율적 )

개발 단위 테스트를 해도 , 실 서비스에서는 에러 발생 가능성은 예측 못한다!
-> 에러를 캐치하고 잡아야만 함 ( 발생한 에러를 확인하고 , 방지 해야 하기 때문에 )

하지만 , MSA 에서 Obserability 는 매우 힘들다!

![450](https://i.imgur.com/SYY829M.png)

원격 측정( Telemetry ) 의 구성 요소

#### Metrics

특정 시점에 측정값으로 정의
초당 HTTP 요청 수 , 지연 시간 + GC 지연 시간 등 측정 가능
해당 값이 임계치를 넘었을 때 Trigger 가능 ( Prometheus 활용 가능 )

##### Prometheus

CNCF ( Cloud Native Computing Foundation ) 의 두 번째 시스템 ( 쿠버네티스를 만든 )

Application 에서 Metric을 측정하고 , 특정 EndPoint로 노출 시 , Prometheus 가 로그 수집 통계를 전담한다
-> prom-client 로 JavaScript SDK 제공

```typescript
import { Inject, NestMiddleware } from "@nestjs/common";
import { NextFunction, Request, Response } from "express";
import {InjectMetric} from '@willsoto/nestjs-prometheus';
import {Histogram} from 'prom-client';


export class RequestDurationCollectorMiddleware implements NestMiddleware{
    private static readonly REQUEST_DURATION_HISTOGRAM_NAME = "REQUEST_DURATION_HISTOGRAM_MSA_NODE_CUSTOMER";
    constructor(@InjectMetric(RequestDurationCollectorMiddleware.REQUEST_DURATION_HISTOGRAM_NAME) private histogram : Histogram){}
    use(req: Request, res: Response, next:NextFunction) {
        const {method,path} = req;
        const end = this.histogram.startTimer({method});

        res.on('finish',()=>{
            end({statusCode : res.statusCode,url:res.req?.route?.path ?? path});
        });

    }
}
```
단순 , 실행 시간 측정 Middleware

![450](https://i.imgur.com/mrqtnrY.png)

앞서 수집한 로그들 Grafana 통해 시각화

- GC 지연시간이나 , Event Loop 지연시간 등을 통해 , System 상태도 측정 가능!


#### Logs

가장 기초적인 에러 해결법을 위한 도구
( console.log ! )
하지만 , MSA 나 매우 큰 규모의 경우 Log 가 섞이므로 한계점에 도달한다

#### Distributed Trace

Client 의 시작부터 끝까지 발생하는 Event 들의 모음을 추적
![500](https://i.imgur.com/iDxVHRI.png)

단일 서버에서는 필요 없으나 , 여러 서비스들이 수행해주는 MSA에서는 매우 중요

물론 , 이 Trace 를 얻는게 쉬운 일이 아님 ( 모든 코드에서 다 작성해야 한다면 매우 비효율적 )
=> 이 역시 제공 오픈 소스 라이브러리가 있음

##### Open Telemetry

이 역시 , CNCF 프로젝트
Telemetry 를 생성 & 수집 그리고 내보내는 SDK 제공
```typescript
import {NodeSDK} from '@opentelemetry/sdk-node';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import {OTLPTraceExporter} from '@opentelemetry/exporter-trace-otlp-http';
import { CompositePropagator, W3CBaggagePropagator, W3CTraceContextPropagator } from '@opentelemetry/core';
import {HttpInstrumentation} from '@opentelemetry/instrumentation-http';
import {NestInstrumentation} from '@opentelemetry/instrumentation-nestjs-core';
import {PgInstrumentation} from '@opentelemetry/instrumentation-pg';
function initOpenTelemetrySdk() {
    const sdk = new NodeSDK({
        spanProcessor : new BatchSpanProcessor(new OTLPTraceExporter()),
        textMapPropagator : new CompositePropagator({
            propagators : [new W3CTraceContextPropagator(), new W3CBaggagePropagator()],
        }),
        instrumentations: [
            new HttpInstrumentation(),
            new NestInstrumentation(),
            new PgInstrumentation(),
        ]
    })
    sdk.start();
    process.on('SIGTERM', ()=>{
        sdk.shutdown()
        .then(
            ()=>console.log('SDK shut down successfully'),
            (error)=> console.log('Error shutting down SDK',error),
            )
        .finally(()=>process.exit(0));
    });
}
initOpenTelemetrySdk();
```

해당 코드는 Telemetry 를 위한 , SDK 설정 코드


### Correlate Telemetry Data

![500](https://i.imgur.com/Hj7Pzv4.png)

CNCF Community 를 통해 일관성 있는 인프라 설계

서비스 이상 있을시 쉽게 파악이 가능!

### Scalability & High Availability

결국 , MSA 의 가장 큰 장점! : 확장성 & 가용성

배민은 쿠버네티스로 이전하고 있는 과도기 상태

대부분의 기업은 EC2 활용해 사용

위에 말한 기술들을 docker-compose 처럼 사용하기 위해 Container 가 필요했다

#### 멀티코어에서 서버의 가동 환경?

배민은 Node.js Cluster 나 PM2 를 사용하지 않는다!

![500](https://i.imgur.com/DDtN41H.png)

이렇게 , Instance 당 Node Process 하나만 구동
-> Obserability 를 위해서

### MonoRepo

- 하나의 버전에 의존!
- 취약점에 대응하기에도 용이
	-> 버전 최신화로만 해도 , 자연스럽게 대응
- 하나의 설정을 쉽게 공유 가능

### Good Code
##### CI/CD
![450](https://i.imgur.com/TLZytBE.png)

뒤에서 버그를 발견할 수록 어마어마하게 시간 / 비용이 발생한다
=> CI / CD 통해 , 미리 발견 및 방지 가능

##### Static Analysis
![450](https://i.imgur.com/lVJ3bc0.png)

- SonarCloud 통해 Code Smell 분석
##### Dynamic Analysis
![450](https://i.imgur.com/DyKakXQ.png)

NestJS 는 Modular 와 DI 때문에 테스트 코드 작성이 매우 용이

##### CI / CD In 배민
![500](https://i.imgur.com/fx31NBq.png)

매우 복잡하며 , 순차적으로 이뤄지게 했다

### 결론

Node.js 라는 볼모지에서 시작하는게 몹시 힘들었다.
( Spring에 비해 매우 부족 )
사내 백엔드 인프라 - 모니터 인프라등 현실을 인지 -> Node 의 적정 기술들 과 규칙들 검토
=> 결국 , 앞으로 높은 생산성을 유지하면서도 높은 품질의 시스템을 만들수 있게 반듯하게 닦을 예정