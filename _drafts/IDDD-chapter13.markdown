---
layout: "post"
title: "IDDD 13장. 바운디드 컨텍스트 통합"
date: "2018-06-14 21:25"
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
interface CollaboratorService  {
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

TODO

### 프로세스 상태 머신과 타임아웃 트래커

### 좀 더 복잡한 프로세스 설계하기

### 메시징이나 시스템을 활용할 수 없을 때

## 마무리
