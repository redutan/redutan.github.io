---
layout: "post"
title: "IDDD 13장. 바운디드 컨텍스트 통합"
date: "2018-06-16 00:04"
tags:
    - IDDD
    - DDD
    - BOOK
---

## 통합의 기본

1. API & RPC (RPC, SOAP, WEB/API) : 탄력성이 떨어짐(*Legacy*)
2. **메시지**
3. **RESTful API**
    * 1.과 유사하지 않은가? 하지만 필자는 HTTP Verbs 로 차이를 이야기하고 있다.
4. 그 외

### 분산 시스템은 근본적으로 다르다

*분산 컴퓨팅의 원칙*

* 네트워크는 신뢰할 수 없다.
* 언제나 지연은 있다.
* 대역폭은 무한하지 않다.
* 정보와 정책은 다수의 관리자에 걸쳐 퍼져 있다.
* 네트워크 전송은 비용이 든다. 
* 네트워크는 다양성을 갖고 있다.

> **왜 원칙인가?**
> * 반드시 해결해야하는 문제점
> * 계획해야하는 복잡성
> * 일반적으로 저지르는 **실수가 아님**

### 시스템 경계에 걸친 정보의 교환

결국은 발행된 언어<sup>Published Language</sup>에 의존해야한다.

* 정보에서는 인터페이스/클래스가 중요한 것이 아니라 말 그대로 데이터의 특성이 중요

**예를 들어서**

`BacklogItemCommitted` 알림 이벤트를 발행한다고 하면 아래와 같은 *계약*을 의미할 것이다.

| 정의/속성 | 설명 |
|-----|-----|
| 타입 | `Notification` |
| 포멧 | JSON |
| `notificationId` | 식별자 |
| `typeName` | 알림의 타입(`BacklogItemCommitted`) |
| `version` | 알림 버전 |
| `occurredOn` | 발생일시 |
| `event` | 본문(Payload) |

직렬화 & 역직렬화 자바 소스는 **SKIP**

*중요한 것*

* 위와 같은 *계약*을 통해 컨슈머와 프로슈머가 서로 통신(정보의 교환)을 할 수 있다
* 시스템 상호간 의존성이 줄어든다. *공유 커널*과 비교해보아라
    * 버전을 통해서 호환성 이슈가 더 없어진다.
* 위 내용을 **사용자 지정 미디어 타입 계약** 이라 한다.

## 레스트풀 리소스를 사용한 통합

**오픈 호스트**

> 서비스의 집합으로서 여러분의 서브시스템으로의 액세스를 허용하는 프로토콜을 정의하자.
> 여러분과 통합해야 하는 모든 사람들이 사용할 수 있도록 프로토콜을 공개하자.
> 프로토콜을 강화하고 확장해서 새 통합 요구사항을 처리하도록 하자. [Evans]

* 결합도를 느슨하게 하기 위해서(자율성) retry와 같은 기법이 요구된다.

*RESTful API 예를 들면*

특정 사용자의 권한을 조회 `/tenants/{tenantId}/users/{username}/inRole/{role}`

* 200 : 권한 있음
* 204 : 권한 없음 (404도 괜찮지 않을까?)

### 레스트풀 리소스의 구현

현업에서 우리는 *오픈 호스트*라고 생각했지만 실상은 *공유된 커널* 이거나 *순응주의자*인 경우가 많다.

*오픈 호스트*라면 바운디드 컨텍스트가 의존성이 매우 낮지만 *공유된 커널*이나 *순응주의자*는 상당한 결합도를 가지게 되므로 자율성을 위해서 **오픈 호스트** 지향하는 것이 좋다고 본다.

*UserResource.java*
```java
    @GET
    @Path("{username}/inRole/{role}")
    @Produces({ OvationsMediaType.ID_OVATION_TYPE })
    public Response getUserInRole(
            @PathParam("tenantId") String aTenantId,
            @PathParam("username") String aUsername,
            @PathParam("role") String aRoleName) {

        User user = this.accessApplicationService()
                       .userInRole(
                               aTenantId,
                               aUsername,
                               aRoleName);

        Response response = this.userInRoleResponse(user, aRoleName);
        return response;
    }
```

*사용자의 권한을 조회하는 API 응답*

```
HTTP1.1 200 OK
Content-Type: application/vnd.saasovation.idovation+json
...
{
    "role": "Author",
    "username": "zoe",
    "tenantId": "A94A-...",
    "firstName": "Zoe",
    "lastName": "Doe",
    "emailAddress": "zoe@saasovation.com"
}
```

### 부패 방지 게층을 통한 REST 클라이언트 구현

![ID와 액세스 컨텍스트와 협업 컨텍스트의 부패 방지 계층 사이의 통합을 위해 사용된 오픈 호스트 서비스](/images/2018/06/IDDD-figure-13-1.png)

`그림 13-1` *ID와 액세스 컨텍스트와 협업 컨텍스트의 부패 방지 계층 사이의 통합을 위해 사용된 오픈 호스트 서비스*

```java
interface CollaboratorService {
    Author authorFrom(Tenant tenant, String identity);
    Creator creatorFrom(Tenant tenant, String identity);
    Moderator moderatorFrom(Tenant tenant, String identity);
    Owner ownerFrom(Tenant tenant, String identity);
    Participant participantFrom(Tenant tenant, String identity);
}
```

**Java Code**
* [TranslatingCollaboratorService](https://github.com/VaughnVernon/IDDD_Samples/blob/master/iddd_collaboration/src/main/java/com/saasovation/collaboration/port/adapter/service/TranslatingCollaboratorService.java)
* [UserInRoleAdapter](https://github.com/VaughnVernon/IDDD_Samples/blob/master/iddd_collaboration/src/main/java/com/saasovation/collaboration/port/adapter/service/HttpUserInRoleAdapter.java)
* [CollaboratorTranslator](https://github.com/VaughnVernon/IDDD_Samples/blob/master/iddd_collaboration/src/main/java/com/saasovation/collaboration/port/adapter/service/CollaboratorTranslator.java)
* [Author](https://github.com/VaughnVernon/IDDD_Samples/blob/master/iddd_collaboration/src/main/java/com/saasovation/collaboration/domain/model/collaborator/Author.java)

리파지토리를 사용할 수도 있었지만 `Author`는 엔터티가 아닌 값 객체이므로 부패 방지 계층이 더 어울리다.
반대로 부패 방지 계층에서 애그리게잇(가변성)을 생성한다면 리파지토리가 더 어울린다.

## 메시징을 사용한 통합

레스트풀 리소스를 이용한 방식보다 **더 높은 수준의 자율성**을 가짐.

**도메인 이벤트**를 외부로 발행할 때 좋다.

### 제품 소유자와 팀 맴버의 정보를 계속해서 제공받는 것

사용자에게 권한을 할당

1. 권한에 사용자를 추가
    * `AccessService#asignUserToRole(AssignUserToRoleCommand)`
    * `Role#assignUser(User)`
2. 사용자에게 권한 할당됨 이벤트 발행 : `new UserAssignedToRole()`
4. 컨텍스트 외부로 이벤트 발행
3. AgilePM 컨택스트에서 이벤트를 구독
    * `TeamMemberEnablerListener#filteredDispatch`
    * `TeamService#enabledProductOwner(EnableProductOwnerCommand)`

### 당신은 책임을 감당할 수 있는가

외부 바운디드 컨텍스트에서 정보를 (이벤트를 통해서) 복제한 후 그것을 지속적으로 동기화한다. 즉, **데이트를 복사하는 복잡한 책임이 생긴다.**

이벤트가 항상 순차적으로 구독자에게 온다는 보장이 없다. 고로 이벤트 처리 시 항상 *발생시각*을 확인해서 처리해야한다.


```java
class TeamService {
    @Transactional  // 구독 처리
    public void disableTeamMember(DisableTeamMemberCommand command) {
        TenantId tenantId = new TenantId(command.getTenantId);
        TeamMember teamMember = 
                teamMemberRepository.teamMemberOfIdentity(
                        tenantId, command.getUsername());
        if (teamMember != null) {
            // 여기가 중요하다!!!
            teamMember.disable(command.getOccurredOn());
        }
    }
}

class Member {
    // 위에 이어서 disable 메서드를 보면
    public void disable(Date asOfDate) {
        if (this.changeTracker().canToggleEnabling(oaOfDate)) {
            this.setEnabled(false); // enable() 라고 하지..
            this.setChangeTracker(
                this.changeTracker().enablingOn(asOfDate));
        }
    }
    public void enable(Date asOfDate) {
        if (this.changeTracker().canToggleEnablig(asOfDate)) {
            this.setEnabled(true)); // disable() 라고 하지..
            this.setChangeTracker(
                this.changeTracker().enablingOn(asOfDate));
        }
    }
}
```

* 위 처리를 통해서 이벤트 순서가 뒤바껴도 처리에 문제가 없다.
* 또한 멱등하게 만드는 역할도 한다.

새로운 책임을 알아보자 : **정보 복제는 사소한 책임이 아니다**

사용자 정보 복제 시 영향을 미치는 이벤트들 

* `PersonContactInformationChanged`
* `PersonNameChanged`
* `UserAssignedToRole`
* `UserUnassignedFromRole`

하지만 이외에도

* `UserEnablementChanged`
* `TenantActivated`
* `TenantDeactivated`

위와 같은 이벤트들이 존재한다.

= *가능하면 바운디드 컨텍스트 전반에서 정보의 중복을 최소화하거나 완전히 제거하는 것이 최선이다*

= 필요할 때 마다 오픈 호스트 서비스와 같은 것을 이용해서 정보를 조회하는 것이 나을수도 있다

> 어쩌란 건지 명확히 이해가 안된다.???
> 필요한 이벤트 발행을 막을 필요는 없지만 굳이 데이터 중복을 통해서 관리하지 말고 이벤트는 이벤트대로 사용하고, 데이터는 필요할 때 마다 조회하는 것이 낫다는 것인가?

### 장기 실행 프로세스와 책임의 회피

다른 바운디드 컨텍스트에게 데이터 생성 책임을 지게 하고, 단순하게 레코드 시스템이 그 고유한 정보를 처리하게 할 것이다. 
즉, **책임을 네트워크 건너로 차버린다.**

*제품을 생성하는 유스케이스를* 살펴본다

1. 사용자는 제품 설명 정보를 제공한다.
2. 사용자는 팀토론에 대한 의사를 표시한다.
3. 사용자는 정의된 제품을 만들도록 요청한다.
4. 시스템은 포럼과 **토론**이 들어간 제품을 만든다.

**애자일 프로젝트 관리 컨텍스트에서**

![애자일 프로젝트 관리 컨텍스트에서 흐름도](http://www.plantuml.com/plantuml/svg/ZP7DIaGn38NtUOg-m7q15yF0k5JmP-4wqnwzWTfUcrJnxGrSos0HTDdpdPEVzAZ6pVfh9evMMpXbGJ7QN9Ge6nSBTwsc7kqHxLqYVaFa4R7FyJmri25HhCLQpKE-5erTLMfvm5k7kkL6r53GVXIzXIg_O4yvIsnyPiK0znqTj0yQbiCqNxWA1H_VsfFOUcbBa_EItKE3EvXMSRxrSnPTQGABU__Up_FFaWqDoLqRMrpf7wdbC1_32obAebbUXdMSv-Wk_zKl)

`ProjectService#newProductWithDiscussion(NewProductCommand)`

* `Product`와 `ProductDiscussion`을 생성
* `ProductCreated` 이벤트 발행
    * `Discussion#availablity#isRequested`(토론 가능 상태)가 참이면 **장기 프로세스를 시작** 한다.
    * 장기 실행 프로세스 == 상품 토론을 생성
* 추후에 구매를 통해서 토론을 생성할 수도 있다. `Product#requestDiscussion(DiscussionAvailablity)`
    * `ProductDiscussionRequested` 이벤트 발행

그렇다면 사용 가능 상태가 `REQUESTED`가 아니라면 이 이벤트의 발행의 의미가 있는가?

\> 이것에 대한 책임은 이벤트 구독자가 갖는다.

`ProductDiscussionRequestedListener#listensToEvents`

* `ProductCreated`, `ProductDiscussionRequested` 이벤트를 수신한다.

`ProductDiscussionRequestedListener`

```java
    protected void filteredDispatch (String type, String textMessage) {
        // 토론 사용 가능상태가 아니면 이벤트 처리 없음
        if (!reater.eventBooleanValue("requestingDiscussion")) {
            return;
        }
        ...
        // CreateExclusiveDicussion 커맨드를 만들어 협업 컨텍스트에 메시징 인프라를 통해서 이벤트 발행
        this.messageProducer().send(...);
    }
```

`여기서 잠깐`

* 애자일 프로젝트 관리 컨텍스트에서 `Product` 애그리게잇에서 발행된 이벤트(`ProductCreated`)를 구독할 리스터를 만들 필요가 있을까?
    * 차라리 협업 컨텍스트에서 `ProductCreated` 이벤트를 바로 구독하는 것이 낫지 않을까?
    * 그런데 `ProductCreated`는 협업 컨텍스트의 유비쿼터스 언어와 어울리지 않고, 이 이벤트가 `Forum`과 `Discussion`을 생성하게 한다는 것을 연결시키는 것이 힘들다.
    * 고로 협업 컨텍스트 입장에서 `CreateExclusiveDicussion` 명령을 이벤트로 받는 것이 어색하지 않다.

**협업 컨텍스트에서**

![협업 컨텍스트에서 흐름도](http://www.plantuml.com/plantuml/svg/ZP5DJaCn38JtFaKkq7S05gWIFojOe9uWBsy4bjBaAROBt1u72JNbWTHLf9dFav6z5urDxPXfYHhdA0ZF48clU34OADMYhURmy96o2Pzmpv9CX6kvQuZgxnEBeg3HwacSU8r5msDjTZoWdJXXQrmevqH2KTRFGJdqTbY8nb9Xjxkzfb2u2MApfCOpw1hUOyVUVRx_FqtJq74a_flurlucVv3VYHquQquLlDCWkBrPYrEhpPdbZRQUB-dob7kKnG_z1G00)

`ExclusiveDiscussionCreationListener#filteredDispatch(String, String)`

* 위에서 이어 애자일 프로젝트 관리 컨텍스트에서 발행한 `CreateExclusiveDicussion`를 구독
* `ForumService#startExclusiveForumWithDiscussion`를 호출해서 `Forum`과 `Discussion`을 생성
    * 생성과 동시에 `DiscussionStarted` 이벤트를 발행한다.

**다시 애자일 프로젝트 관리 컨텍스트로 돌아와서**

![다시 애자일 프로젝트 관리 컨텍스트에서 흐름도](http://www.plantuml.com/plantuml/svg/VP1D2i8m48NtSufSe1TmKRfm8oWeFK5-7ZfGavAP2DxU24gaj2uV-RwybmoYDckvJnIiMcS5vWGHUyMbe81yYfhJPFOileXmYkDRG3YoA28opJMovzb6DUUSGl4w8Z_OO-s849Nr-OtjsaDaPQi8HBy3JDVrs-LcPwGuyPaTQ9lg-iMowl6dhrcqO9hr5s_SsckgEXStiTneG0prery0)


`DiscussionStartedListener#filteredDispatch(String, String)`

* 협업 컨텍스트에서 발행한 `DiscussionStarted` 이벤트를 구독한다.

이어서 `ProductService#initiateDiscussion(InitiateDiscussionCommand)`

* `Product#initiateDiscussion` 를 실행해서 아까 생성한 `Product`에서 `Discussion` 생성 상태 변경
    * 당연히 멱등하다. 이미 `Discussion`을 생성해서 `READY` 상태라면 아무일도 일어나지 않는다. 즉 `REQUESTED` 상태에서만 이하 작업이 실행된다.
    * 그리고 `ProductDiscussionInitiated` 이벤트를 발행한다.

`여기서 잠깐`

* 이 장기 실행 프로세스는 네트워크를 건너서 통신하고 메시징 인프라에 의존하는데 여기에 문제가 생기면 어떻게 될까?
* 즉 프로세스가 끝까지 실행한다고 확신할 수 있을까?

### 프로세스 상태 머신과 타임아웃 트래커

위 문제점 해결을 위해서 타임아웃 트래커가 필요하게 된다.

*타입아웃 트래커*

트래커는 완료까지 부여된 시간이 만료된 프로세스를 감시하는데. 이런 프로세스는 만료되기 전까지 몇번이고 재시도될 수 있다. 
트래커는 원하는 경우 고정된 시간 간격을 재시도하도록 설계할 수 있고, 재시도를 전혀 하지 않거나 정해진 횟수만큰을 재시도한 후에 타임아웃을 발생시키도록 할 수도 있다.
*기술적 서브도메인의 일부다*

![상품의 토론 생성 흐름도 with 타임아웃 트래커](http://www.plantuml.com/plantuml/svg/XP5HIiSm38VVUufUm0liGGRwCX2KkGkK9W_1BTa_IGLlRmiAhbHzAQ7zVUdNT3PFwkNOGnPsbJs-g439_aYMYna9htWhQ8xmH7Lbr71MX3ATYVqx_ehwJXdxeunccwRyXhhYAKOk-d49RNJWWx2v9cA4AnD7LuNmlsAyk-_CuXJRKtz02vDJybg5BbhXlxMcVaeiNmgDW-VYWvQ_ZQDsIm1ZeEqqSnnwBn1cPAY_zma0)

`ProductService#startDiscussionInitiation`

```java
package com.saasovation.agilepm.application;

class ProductService {
    @Transactional
    public void startDiscussionInitiation(
            StartDiscussionInitiationComman command) {
        Product product = productRepository.findById(...)
        
        TimeConstrainedProcessTracker tracker = new TimeConstrainedProcessTracker(
                product.tenantId().id,
                ProcessId.newProcessId(),
                "Create discussion for product : " + product.name(),
                new Date(),
                5L * 60L * 1000L, // 5분 마다 재시도
                3,                // 총 3회 재시도한다.
                ProductDiscussionRequestTimedOut.class);

        processTrackerRepository.add(tracker);  // 타임아웃 트래커 등록
        
        product.setDiscussionInitiationId(
                tracker.processId().id());
    }
}
```

**상품의 토론 타임아웃 or 재시도 흐름도**

![상품의 토론 타임아웃 or 재시도 흐름도 with 타임아웃 트래커](http://www.plantuml.com/plantuml/svg/dOzTgi8m44RViufiuDu5-20jj8L2GL4t49EvC90Vd9aKkdldHG8g2lSb9EISR-RhM1n9JT6uA5OmGQbYh3rI2TNBWEmhCvPy0g5jGHR8GFPd_o3EG2jwiBidkNszXHbo69F3E1Mwg1WELHJomFmfGCq_bTfQSqP19tfhMkEbWMgsHxzgYBjYHDb-ftvUni50PB04sl9VzPlwlJp1hGBBon03EPXEZvhY7G00)

**마지막에 상품과 토론이 정상적으로 생성되면 흐름도**

![상품과 토론이 정상적으로 생성 시 흐름도](http://www.plantuml.com/plantuml/svg/ZOun3i8m34Ntd28Nu08Cg1AC34Zj1IBd3nQ94zaEvwTC5HKWHlk_x-V9FAcFMW8rSMqbNjXec76J-HKXNzaS0Wrz7Pcu9_5uqvO7-GnzCE5JzBPRkEBSn5mJ2_AA4CmMJNI7Xl3L6G-ddIeU8mix9yVM2ZjcQ_sB_tnmFKAjzW973XCaZrgU)

* 애자일 프로젝트 관리 컨텍스트에서 마지막으로 상품의 토론이 정상적으로 생성되면 **타임아웃 트래커를 완료** 시켜야 한다.
* 이것으로 프로세스는 끝난다.

*위 장기 실행 프로세스의 문제점과 해결책*

* 동일한 `CreateExclusiveDiscussion` 커맨드가 여러 번 발송된다.
    * 멱등성을 지켰기 때문에 문제가 없을 것 같다.
    * 하지만 애자일 프로젝트 관리 컨텍스트가 실패한다면 문제가 발생할 수 있다.
    * 결국 협업 컨텍스트의 `ExclusiveDiscussionCreationListener`가 멱등한 애플리케이션 서비스 메서드로 작업을 위임할 수 있다면 많은 문제점을 해결할 수 있다.

![협업 컨텍스트에서 흐름도](http://www.plantuml.com/plantuml/svg/ZP5DJaCn38JtFaKkq7S05gWIFojOe9uWBsy4bjBaAROBt1u72JNbWTHLf9dFav6z5urDxPXfYHhdA0ZF48clU34OADMYhURmy96o2Pzmpv9CX6kvQuZgxnEBeg3HwacSU8r5msDjTZoWdJXXQrmevqH2KTRFGJdqTbY8nb9Xjxkzfb2u2MApfCOpw1hUOyVUVRx_FqtJq74a_flurlucVv3VYHquQquLlDCWkBrPYrEhpPdbZRQUB-dob7kKnG_z1G00)

* 위 흐름도 기준에서 `startForum`, `startDiscusssion` 행위가 멱등하면 해결된다.

### 좀 더 복잡한 프로세스 설계하기

복잡한 상황에서 다수의 완료 단계가 필요하게 된다면 **상태 머신**이 좋은 선택

```java
Proces process = new TestableTimeConstrainedProcess(
        TENANT_ID,
        ProcessId.newProcessId(),
        "Testable Time Constrained Process",
        5000L); // 재시도 없이 5초 내로 완료되야 한다.

TimeConstrainedProcessTracker tracker = 
        process.timeConstrainedProcessTracker();
    
process.confirm1();

assertFalse(process.isCompleted());
assertFalse(process.didProcessingComplete());
assertEquals(process.processCompletionType(), ProcessCompletionType.NotCompleted);

process.confirm2();

assertTrue(process.isCompleted());
assertTrue(process.didProcessingComplete());
assertEquals(process.processCompletionType(), ProcessCompletionType.CompletedNormally);
assertNull(process.timedOutDate());

tracker.informProcessTimedOut();    // 어짜피 이미 완료되어 있으므로 아무처리가 없을 것
assertFalse(process.isTimedOut());  // 즉 타임아웃 되지 않았으므로 언제나 false
```

* `TestableTimeConstrainedProcess`는 재시도 없이 5초 내로 완료되야 한다.
* `confirm1()`, `confirm2()`만 호출된다면 완전히 완료 상태로 표시된다. 
    * 내부적으로 2가지 상태를 속성으로 지니고 있다.
    * 즉, `confirm1`, `confirm2` 둘 다 `true` 여야지 완료 상태

### 메시징이나 시스템을 활용할 수 없을 때

* **고가용성 확보**가 최선이다.
* 메시징인프라가 죽었다가 다시 살아났을때 *자동으로 리스너가 재활성화* 될 수록 해야한다.

## 마무리

* 분산 컴퓨팅에서의 통합
* RESTful VS 메시징인프라
* 장기 수행 프로세스
    * Retry와 타임아웃
    * **멱등성**
    * 복잡한 상태를 해결해주는 *상태 머신*