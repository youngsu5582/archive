## 아티클 링크
https://toss.tech/article/engineering-note-4
## 세줄 요약

## 내용
### 서론
Null 은 왜 나쁠까? 에서 핵심은 코드 읽는 사람의 입장에서 생각을 하자!
-> 해당 주제는 코드를 사용하는 입장에서 얘기를 해볼 예정
#### Example
재전송 , 메일 수신자 필터링 , SMS 전송 등 다양한 기능 제공하는 메일 발송 기능 존재
```kotlin
class Mail(
	// ...
) {
  fun send(
    phoneFallback: Boolean?,
    phoneNumber: String?,
    isForceSend: Boolean?,
    recipient: String,
    id: Long,
    mailDomainFilterService: MailDomainFilterService?,
    mailRetryService: MailRetryService?,
    title: String,
    body: String,
    param: Map<Any, Any>,
    reservedAt: Instant?,
  )
}
```
- 해당 메서도를 호출 하려면?
=>
```kotlin
mail.send(
  phoneFallback = null,
  phoneNumber = null,
  isForceSend = null,
  recipient = "jaeeun.na@tosspayments.com",
  id = -1,
  mailDomainFilterService = null,
  mailRetryService = null,
  title = "안녕하세요",
  body = "메일 본문입니다",
  param: emptyMap(),
  reservedAt: null,
)
```
- 막상 보면 , phoneFallback 에 어떻게 넣어줘야 하는지 , param 에는 뭘 넣어줘야 하는지 유추 불가
=> 인자가 많고 , 의미 파악하기 어려우면 메소드 사용성이 떨어짐

### 생산성 악마의 미소
##### In Gmail?
![500](https://i.imgur.com/2u1PMsF.png)

- 받는 사람 입력 , 본문 내용 ( Optional ) 으로 메일 발송 가능
- 어떤 값이 필수인지도 바로 알 수 있음!

-----

- send 메소드에는 입력할 '필수' 인자가 너무 많다!
	- 잘 모르는 인자 , 사용할 필요 없는 인자에 null 채워 넣는 것 도 꺼림칙함..!
	- mailDomainFilterService 객체를 어떻게 만들지??
	- isForceSend 의 의미는?
	-> git blame 해버려야 하나...? ( 누가 짠 거여 )
##### git blame
- 어떤 코드를 누가 숴정 했는지 확인하는 명령어
- git blame <파일경로>
	-> commit id , 파일 이름 , 수정한 사람 , 수정 시간 순서 , 라인 번호 + 코드 

#### 다른 선택지 고려!
##### send 메소드는 다른 곳에서 어떻게 쓰고 있을까?
=> 비슷하게 따라 해야겠다.

- 메소드 실행이 과연 성공할까? 따라 한 방법이 올바른지 확인할 방법이 있나?
- 따라 한 코드 맥락과 사용하는 맥락이 같을까?
##### 아무 인자자 넣고 일단 실행해 볼까?
=> 실행에 성공하면 좋고 , 예외가 생긴다면 그때 확인해보자!

- 이 상태로 라이브 배포를 할 수 있을까?
- 내가 만드는 코드를 정확히 이해하지 못하는데 괜찮나?
##### send 메소드는 구현은 어떻게 되어있지?
=> 각 인자 의미를 파악하자!

- 내부 구현을 정말 이해할 수 있을까?
- 그 시간에 다른 기능을 만드는게 낫지 않을까?

=> 어떤 선택지 고르더라도 모두 생산성이 낮다!

### 함께 사용하는 인자는 하나로 묶자

- phoneFallback 인자?
	-> 메일 발송 실패했을 때 메일에 있는 내용을 문자로 대신 보내기 위한 인자
	->  그래서 phoneNumber 가 있구나!
	=> phoneFallback 이 true 면 , phoneNumber 에 적힌 번호로 메일 내용 전송
∴ phoneFallback 과 phoneNumber 는 항상 같이 사용되는 인자!

```kotlin
// fallbackFeatureOption 인자를 삭제했다
fun send(
  // ... 
) {
  sendInternal(
    phoneFallback = null,
    phoneNumber = null,
    // ...
  )
}

// 메서드 이름으로 의도를 표현했다
fun sendWithFallback(
  fallbackFeatureOption: FallbackFeatureOption, // non-nullable
  // ...
) {
  sendInternal(
    phoneFallback = fallbackFeatureOption.phoneFallback,
    phoneNumber = fallbackFeatureOption.phoneNumber,
    // ...
  )
}
```
- send 메소드를 분리해서 구현
- send 메소드가 노출하는 phoneFallback 맥락 사라짐
=> 여전히 이상한 점이 존재

- phoneFallback 이 false라면?
	-> 상관없이 , phoneNumber 를 넣어야 함! ( 로직상 이상함 - Fallback 을 안하는데 번호를 왜 넣지? )
### 관련 없는 것 분리

- isForceSend?
	-> 동료에게 물어보니 , 메일 수신 거부한 이메일 주소에도 발송 길고 남기는 정책 인자!
	-> true 로 설정하면 수신 거부한 주소에도 , 메일을 발송할 수 있다

- MailDomainFilterService 에서 DB 에 접근해서 발송할지 결정
	-> isForceSend 이 true면 , 해당 인자 사용하지 않음!
```kotlin
class MailDomainFilterService {
  fun isFiltered(domain: String): Boolean { 
    // database 에서 목록을 가져온 뒤, 발송 여부 결정
  }
}
```
=> 메일 한 통 보내는 데 , 이런 세부 사항 까지 알아야 하나?

#### 개선점
```kotlin
class Mail(
  private val mailDomainFilterService: MailDomainFilterService,
) {
fun send()
fun sendWithFallback()
fun sendWithFallbackAndForced(

    fallbackFeatureOption: FallbackFeatureOption?,   
    recipient: String, 
    id: Long,
    mailRetryService: MailRetryService?,
    title: String,
    body: String,
    param: Map<Any, Any>,
    reservedAt: Instant?,
  ) { 
    sendInternal(
      phoneFallback = fallbackFeatureOption?.phoneFallback,
      phoneNumber = fallbackFeatureOption?.phoneNumber,
      
isForceSend = true,

      recipient = recipient,
      id = id,
      
mailDomainFilterService = null,

      mailRetryService = mailRetryService,
      title = title,
      body = body,
      param = param,
      reservedAt = reservedAt,
    )
  }
}
```
#### Is It Okay?
- fallbackFeatureOption 기능과 isForceSend 기능이 서로 겹침!
- 다섯 개의 메소드를 만들어야 하는 상황이 되어 버림!
	( send , sendInternal , sendWithFallbackAndForced , snedWithFallback , sendWithForce )

#### Method Chaninig
```kotlin
class Mail(
  private val mailDomainFilterService: MailDomainFilterService
) {
  
private var enableForceSendFeatureForCompliancePurpose: Boolean = false
private var smsFallbackFeaturePhoneNumber: String? = null
  
fun enableSmsFallbackFeature(smsFallbackFeaturePhoneNumber: String): Mail {
	this.smsFallbackFeaturePhoneNumber = smsFallbackFeaturePhoneNumber
}


  
fun enableForceSendFeatureForCompliancePurpose(): Mail { 
	this.enableForceSendFeatureForCompliancePurpose = true
}

fun send(
    id: Long,
    recipient: String,
    mailRetryService: MailRetryService?,
    title: String,
    body: String,
    param: Map<Any, Any>,
    reservedAt: Instant?,
  ) {
    sendInternal(
	  phoneFallback = this.smsFallbackFeaturePhoneNumber != null,
	  phoneNumber = this.smsFallbackFeaturePhoneNumber,
	  isForceSend = this.enableForceSendFeatureForCompliancePurpose,
      recipient = recipient,
      id = id,
	      
	mailDomainFilterService = if (this.enableForceSendFeatureForCompliancePurpose) {
		this.mailDomainFilterService
		      } else {
		null
		},
	    mailRetryService = mailRetryService,
	    title = title,
	    body = body,
	    param = param,
	    reservedAt = reservedAt,
    )
  }
}
```

```kotlin
mail
  .enableSmsFallbackFeature("010-1234-5678")  // 이 메서드를 호출하지 않아도 괜찮다
  .enableForceSendFeatureForCompliancePurpose()  // 이 메서드를 호출하지 않아도 괜찮다
  .send(...)
```
=> Builder Design Pattern 과 매우 유사
- 사용자가 깊게 고민하지 않아도 된다!
- mail . 찍으면 사용할 수 있는 메소드들이 나옴 + 메소드 이름 보고 충분히 기능 유추 가능
- send() 메소드의 주의 사항을 더 기억하지 않아도 됌! ( 사용하고 싶은 기능 호출하기만 하면 되므로 )

### 가장 중요한 인자만 남긴다
```kotlin
import kotlin.random.Random
import java.time.Instant

class Mail(
    private val mailDomainFilterService: MailDomainFilterService,
    private val mailRetryService: MailRetryService,
    private val param: Map<Any, Any>?
) {

    private var enableForceSendFeatureForCompliancePurpose: Boolean = false
    private var smsFallbackFeaturePhoneNumber: String? = null
    private var scheduleSendReservedAt: Instant? = null
    private var enableRetryFeature: Boolean = false

    fun enableSmsFallbackFeature(smsFallbackFeaturePhoneNumber: String): Mail {
        this.smsFallbackFeaturePhoneNumber = smsFallbackFeaturePhoneNumber
        return this
    }

    fun enableForceSendFeatureForCompliancePurpose(): Mail {
        enableForceSendFeatureForCompliancePurpose = true
        return this
    }

    fun enableScheduleSendFeature(reservedAt: Instant): Mail {
        scheduleSendReservedAt = reservedAt
        return this
    }

    fun enableRetryFeature(): Mail {
        enableRetryFeature = true
        return this
    }

    fun send(
        recipient: String,
        title: String,
        body: String,
    ) {
        sendInternal(
            phoneFallback = smsFallbackFeaturePhoneNumber != null,
            phoneNumber = smsFallbackFeaturePhoneNumber,
            isForceSend = enableForceSendFeatureForCompliancePurpose,
            id = Random.nextLong(),
            mailDomainFilterService = if (enableForceSendFeatureForCompliancePurpose) {
                mailDomainFilterService
            } else {
                null
            },
            mailRetryService = if (enableRetryFeature) {
                mailRetryService
            } else {
                null
            },
            title = title,
            body = body,
            param = param ?: emptyMap(),
            reservedAt = scheduleSendReservedAt,
        )
    }

    fun templateSend(
        recipient: String,
        title: String,
        body: String,
        param: Map<Any, Any>
    ) {
        sendInternal(
            // ...
            param = param
            // ...
        )
    }

    private fun sendInternal(
        phoneFallback: Boolean,
        phoneNumber: String?,
        isForceSend: Boolean,
        id: Long,
        mailDomainFilterService: MailDomainFilterService?,
        mailRetryService: MailRetryService?,
        title: String,
        body: String,
        param: Map<Any, Any>,
        reservedAt: Instant?,
    ) {
        // ...
    }
}
```
- send 는 가장 중요한 제목 , 수신자 , 내용만 담으면 된다!
```kotlin
// 가장 간단하게 사용할 때
mail.send(
  recipient = "jaeeun.na@tosspayments.com",
  title = "안녕하세요",
  body = "메일 본문입니다"	
)

// 지원하는 모든 기능을 사용할 때
mail
  .enableSmsFallbackFeature("010-1234-5678")
  .enableForceSendFeatureForCompliancePurpose()
  .enableScheduleSendFeature(Instant.now().plus(Duration.ofHours(2)))
  .enableRetryFeature()
  .send(
    recipient = "jaeeun.na@tosspayments.com",
    title = "안녕하세요",
    body = "메일 본문입니다"
  )
```

- 이렇게 하면 , Mail 생성자에 의존성이 생긴게 아닌가?
	-> 조삼모사 아닌가? ❌

- 함수 인자에 의존성을 주입하는 것은 코드 사용자에게 부담을 준다!
- 객체에서 의존성을 관리하면 코드 작성자는 그 부담을 받지 않는다!
		+ 중복 호출하는 의존성 주입 코드 한 곳으로 모으는 효과 기대 가능!
		+ 객체 간 표현도 더욱 풍부해짐!
## 결론
### 지금까지 과정
1. 메일 보내기 위해 mail.send() Method 를 재사용 하려 했다.
2. 사용자에게 친절하지 않은 코드 때문에 mail.send 를 사용하려면 너무 많은 맥락을 알아야 함
3. 팀 내부에서 도메인 지식을 습득
4. 습득한 지시을 가지고 코드 개선 ( 리팩토링!! )

=> 네번째 과정이 가장 중요하다!
- 만든 개발자만 이해하는 코드는 좋지 않다.
- 개발자가 요구사항 위해 사용하는 코드를 쉽게 읽을 수 있는 상황이 가장 바람직하다.

#### Q. 이 프로젝트에서 메일을 보내려면?
##### Before
- Mail Class 의 send() 를 통해서 보낼 수 있는데요 , id 는 꼭 랜덤으로 넣어주시고
	예약 메일이 아니면 reservedAt 에 null을 넣고 , 수신자 메일 도메인도 알아야 해요! 강제로 보내는 경우도 있어서요
	##### After
- Mail 클래스가 제공하는 공개 메소드 목록을 보세요!

=> 도메인 지식과 코드가 아주 간단해짐

### 느낀점

함수 인자에 의존성을 주입하는게 아닌 , 객체에서 의존성을 관리하자는 것에 격렬히 공감!
send 를 쓸 때마다 매우 많은 인자를 넣는 것이 아닌 , 필수적인 것 과 선택적인 것을 잘 파악해서 분리하자
Builder Pattern 에 대해서 좀 더 공부해보고 , 도메인을 생성하거나 사용할 때 깔끔하게 사용해봐야 겠다.

## 참고 링크



##### Writed By Obisidan