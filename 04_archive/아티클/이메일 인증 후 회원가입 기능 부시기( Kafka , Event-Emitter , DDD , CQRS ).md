---
tags: 아티클
---
### 깃허브 주소

[RooTrip-Factoring](https://github.com/youngsu5582/RooTrip-Factoring)
### 진행 일지

1. [rootrip-factoring:feat] create-local-user 기능 구현

### 기록할 점
- create-local-user 아키텍쳐 & 기능 구현

### create-local-user Architecture
리팩토링 하며 , 회원가입 과정이 대규모 수정이 되었다.
#### 기존
이메일 인증 완료 -> 서버에 전송 -> DB 에 Insert -> 여부에 따른 분기 처리

큰 문제점은 없었으나 , 이메일 인증을 프론트 단에서 검증하고 , 백엔드로 보낼때는 검증할 수 없다는 점
동기적인 처리로 , 결과 대기 등 비효율적인 점이 문제였다. 
더불어 , MVC Pattern 으로 인해 , 각 Class 들이 의존 하는게 문제였다.

#### 수정안
우선 , 이메일 인증을 임시 회원가입을 완료한 후 발송이 되고 , 
이메일에서 , redirect Code 가 포함된 사이트를 클릭할 시 , 회원가입이 마무리 되게 하였다.

비동기 적이고 각 파일이 각각 역활만을 하는 서버 구현을 위해 , 전체적으로 부분을 나누었다.

1. 이메일 인증은 , Kafka 에 발행 해 다른 서버가 처리
2. 임시 회원가입 payload 는 Redis 에 저장 후 인증시 , DB 에 처리
위 두개를 Event 로 만들어서 , 비동기로 수행을 하게 했다.

하단에 코드랑 같이 자세히 설명하겠다. ( repository 는 단순 Read 작업이므로 생략 )

### Controller

``` typescript
@Controller('auth/register')
export class CreateLocalUserController {
  constructor(private readonly commandBus: CommandBus) {}
  /**
   * 사용자 회원가입 기능
   *
   * Body 를 통해 받은 createUserProps( email , nickname , name , password )를 통해 User 를 만든다.
   *
   * @tag user
   * @param createUserProps
   * @returns
   */
  @TypedRoute.Post()
  async create(
    @TypedBody() createLocalUserProps: CreateLocalUserProps,
  ): Promise<
    | ResponseBase<{ id: string }>
    | EmailAlreadyExistError
    | NicknameAlreadyExistError
  > {
    const command = new CreateLocalUserCommand(createLocalUserProps);
    const result: Result<
      string,
      EmailAlreadyExistError | NicknameAlreadyExistError
    > = await this.commandBus.execute(command);
    return match(result, {
      Ok: (id: string) => new ResponseBase({ id }),
      Err: (error: EmailAlreadyExistError | NicknameAlreadyExistError) => {
        throw error;
      },
    });
  }
}
```
- Controller 당 하나의 Logic 을 수행하기에 , 단일 경로를 Controller Decorator 에 넣었다.
- CQRS Pattern 을 준수하기 위해 , CommandBus 를 사용
	( Controller 는 command 가 어떻게 처리 되는지 알 필요 없기에 느슨한 결합도 )
	참고 : https://velog.io/@dragonsu/%EC%99%9C-CQRS-%EC%9D%B8%EA%B0%80-Nest.js
- result 는 oxide 라는 Library 를 사용하여 , Ok 와 Err 로 받음 ( service 부분에서 더 자세히 설명 )
- result 를 unwrap 하여 , 분기 처리
	- Ok 면 성공적으로 완료
	- Err 면 에러가 발생 ( throw 인 이유는 Exception Filter 에서 Catch 해서 재처리 )
		참고 : https://velog.io/@dragonsu/%EB%AA%A8%EB%93%A0-%EC%97%90%EB%9F%AC-%EC%B2%98%EB%A6%AC%ED%95%98%EB%8A%94-Exception-Filter-%EC%83%9D%EC%84%B1%EA%B8%B0
### Service

``` typescript
@CommandHandler(CreateLocalUserCommand)
export class CreateUserCommandHandler implements ICommandHandler {
  constructor(
    @Inject(USER_REPOSITORY)
    private readonly userRepository: UserRepositoryPort,
    private readonly eventEmitter: EventEmitter2,
  ) {}
  async execute(
    command: CreateLocalUserCommand,
  ): Promise<
    Result<string, EmailAlreadyExistError | NicknameAlreadyExistError>
  > {
    const { email, nickname } = command;
    if (await this.userRepository.findByEmail(email))
      return Err(new EmailAlreadyExistError());
    if (await this.userRepository.findByNickname(nickname))
      return Err(new NicknameAlreadyExistError());

    const user = UserEntity.create(command);
    await user.publishEvents(this.eventEmitter);
    return Ok(user.id);
  }
}

```
- CommandHandler 를 통해서 , 해당 Command 를 처리하는 Handler 임을 정의
- userRepository 를 통해 , 이메일 과 닉네임이 중복인지 검증 ( Err , Ok 는 oxide library)
- 그후에는 DDD Pattern 을 준수하기 위해 , UserEntity 에서 비즈니스 로직을 수행
- 수행 후 , Event 들을 publishing ( Handler 는 역시 어떤 event 가 들어있는지 알 필요 없기에 느슨한 결합도 )
### Entity

``` typescript
export class UserEntity extends AggregateRoot<CreateLocalUserProps> {
  protected readonly _id: AggregateId;
  static create(createProps: CreateLocalUserProps): UserEntity {
    const id = randomId();
    createProps.password = hashingPassword(createProps.password);
    const user = new UserEntity({ id, props: createProps });
    const vertificationRedirectCode = randomCode();
    user.addEvent(
      new SendVertificationEmailDomainEvent({
        aggregatedId: id,
        email: createProps.email,
        nickname: createProps.nickname,
        redirectCode: vertificationRedirectCode,
      }),
    );
    user.addEvent(
      new SaveTemporalRegisterDataDomainEvent({
        aggregatedId: id,
        data: createProps,
        key: vertificationRedirectCode,
      }),
    );
    return user;
  }
```
- 받은 Payload 들을 활용해서 , Entity 를 만든다.
- Entity에 event 들을 추가한다. ( aggregatedId 는 엔티티 일관성을 위해 사용 )
	- 인증코드 발송 이벤트는 ( email , nickname , redirectCode 포함 )
	- 임시 데이터 저장 이벤트는 ( key , data 포함 )

#### SendVertificationEmailEventListener

``` typescript
@Injectable()
export class SendVertificationEmailEventListener {
  constructor(@Inject(PRODUCER) private readonly producer: Producer) {}
  @OnEvent(SendVertificationEmailDomainEvent.name)
  handleSendVertificationEmailEvent(event: SendVertificationEmailDomainEvent) {
    const result = this.producer.send({
      messages: [
        {
          value: JSON.stringify({
            email: event.email,
            code: event.redirectCode,
            nickname: event.nickname,
          }),
        },
      ],
      topic: SEND_VERTIFICATION_EMAIL,
      acks: 1,
    });
    result.then((data) => data);
    result.catch((err) => err);
  }
}
```
- kafkajs 의 Producer 를 DI를 통해 주입 받아서 실행
	참고 : https://velog.io/@dragonsu/Kafka-Kafka-producer-in-Nestjs-2-
- topic 과 message 를 포함해 publishing
- 여기서 , ack 를 통한 재처리는 했으나 , 실제 작동하는지 확인해볼 방법이 없다.
- 아직 실패 , 성공 여부에 따른 로그 처리는 생각하지 않았기에 data , err 는 그대로 처리

#### SaveTemporalRegisterDataEventListener

``` typescript
import { Injectable, Logger } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { SaveTemporalRegisterDataDomainEvent } from '../domain/events/save-temporal-register-data.domain-event';
import { RedisProvider } from '@src/providers/redis.provider';

@Injectable()
export class SaveTemporalRegisterDataEventListener {
  private readonly logger = new Logger(
    SaveTemporalRegisterDataDomainEvent.name,
  );
  constructor(private readonly redis: RedisProvider) {}
  @OnEvent(SaveTemporalRegisterDataDomainEvent.name)
  handleSaveTemporalRegisterDataEvent(
    event: SaveTemporalRegisterDataDomainEvent,
  ) {
    const key = `temporalRegister : ${event.key}`;
    this.redis.saveData(key, JSON.stringify(event.data));
  }
}

```
- Redis 를 주입받아서 실행
- code 를 key 로 임시 data 저장

#### microservice / SendVertificationEmailMessageController

``` typescript
@Controller()
export class SendVertificationEmailMessageController {
  constructor(private readonly emailerProvider: EmailerProvider) {}
  @MessagePattern(SEND_VERTIFICATION_EMAIL)
  async execute(@Payload() payload: SendVertificationPayload) {
    this.emailerProvider.sendVertification(payload);
  }
}
```

- nestjs/microserivices 에서 제공하는 , @MessagePattern 을 사용하여 subscribe
- 사실 여기가 , 에러 처리시 사용자가 느끼는 가장 민감한 부분이기에 재처리를 보장하려고 시도했으나 , 실패했다.
``` typescript
  async execute(@Payload() payload: SendVerificationPayload) {
    try {
      this.emailerProvider.sendVerification(payload);
    } catch (err) {
      console.error(err);
      if (this.count < 3) {
        this.count++;
        this.execute(payload);
      }
    }
  }

```
- 해당 방법 처럼 간단한 재시도를 통해서 , 처리하려고 했으나 실패해서 차후 더 찾아볼 예정

### Infra
![](https://i.imgur.com/VnthqZA.png)
- AWS 나 Docker 부분은 해당 Architecture 설명에 필요가 없으므로 생략
- User Entity 는 포함하려다 가독성 및 서버적 입장에서 생각해서 넣지 않음

### 사담
- 비동기적 이기에 , 매우 빠르게 작동한다.
- 매우 단순할 수 있는 로직이나 , 세분화 하고 분담하니 복잡해지는 것을 느꼈다.
- eventEmitter 의 동작에 대해서 더 자세히 알아볼 예정이다.
	( 주 스레드에서 동작한다고 알려져 있는데 , 이러면 사실상 동기적 처리가 아닌지 고찰 )
	( while 문으로 의도적 block 을 하면 , 서버가 blocking 된다. )
- 전체 코드는 아직 PR 로 branch 를 합치지 않아서 , 나중에 PR 시에 밑에 주소 추가할 예정

##### Writed By Obisidan