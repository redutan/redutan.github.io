---
layout: "post"
title: "IDDD 11장. 팩토리"
date: "2018-06-07 11:12"
tags:
    - IDDD
    - DDD
    - BOOK
---

애그리게잇을 생성하는 책임을 가지는 메소드나 객체를 말한다.

* 애그리게잇 생성을 캡슐화

## 도메인 모델 내의 팩토리

> 복잡한 객체와 애그리게잇 인스턴스를 생성하는 책임을 별도의 객체로 이동시키자. 여기서의 책임은 도메인 모델과 관련이 있지 않지만, 여전히 *도메인 설계를 구성하는 한 요소*다. 모든 복잡한 조립 과정을 캡슐화하고, 클라이언트가 인스턴스화된 객체의 구체적 클래스를 참조할 필요가 없도록 인터페이스를 제공하자. 전체 애그리게잇을 하나의 조각(원자성)으로 생성하고 고정자(Invariant)를 지정하자 [Evans]

* 팩토리 메소드 : 이 책은 주로 여기만 나옴
* 팩토리 객체(클래스)
    * 애그리게잇 생성이 매우 복잡하고 다른 객체의 도움이 필요하면 클래스 분리를 하는 것이 좋은 것 같다.

*가능한 팩토리 위치*

* 애그리게잇 루트
    * 정정 생성자를 통해서 도메인 의도가 나오면 더 좋다. `static Task#forDraft()`
    * 아니면 상위 루트에서 하위 엔터리 생성도 가능 `Project#createTask()`
* 도메인 서비스 : 서비스 기반 팩토리
    * `ProjectService#createProject()`
* 팩토리 : 이 책은 다루지 않음
    * `ProjectFactory#create()`

## 애그리게잇 루트상의 팩토리 메소드

| 바운디드 컨텍스트 | 애그리게잇 | 팩토리 메소드 |
|---------------|---------|-----------|
| 식별자와 액세스 컨텍스트 | `Tenant` | `offerRegisterationInvitation()` |
| | | `provisionGroup()` |
| | | `provisionRole()` |
| | | `registerUser()` |
| 협업 컨텍스트 | `Calendar` | `scheduleCalendarEntry()` |
| | `Forun` | `startDiscussion()` |
| | `Discussion` | `post()` |
| 애자일 PM 컨텍스트 | `Product` | `planBacklogItem()` |
| | | `scheduleRelease()` |
| | | `scheduleSprint()` |

### `CalendarEntry` 인스턴스 생성하기

```java
@Entity
class Calendar extends AbstractAggregateRoot {
    CalendarEntry scheduleCalendarEntry(...) {
        CalendarEntry calendarEntry = new CalendarEntry(...);
        this.registerEvent(new CalendarEntryScheduled(...));
        return calendarEntry;
    }
}
```

* 유비쿼터스 언어에 부합되는 도메인이 표현됨 : `scheduleCalendarEntry`
* 보호절이 없다 : 어짜피 `new CalendarEntry(...)`가 책임짐
* `Setter`를 사용하지 않는다. > **원자성** + **부수효과 줄어듬** + **불변식 강제**가 쉬움
* 이벤트 발행 : `CalendarEntryScheduled`
* 인자의 갯수가 줄어든다.
    * `scheduleCalendarEntry(...)`에서는 11개가 요구되지만, `new CalendarEntryScheduled(...)`에서는 9개로 줄어든다.
    * 개인적으로는 이것도 인자를 캡슐화 시켜서 객체로 말아버리는 것이 좋은 것 같다. 9개의 인자라도 너무 많은 것 같음.

**하지만**

* `CalendarEntry`를 생성하기 위해서는 꼭 **`Calendar` 인스턴스**가 요구됨
    * 이는 DB 조회라는 부하가 추가됨

### `Discussion` 인스턴스 생성하기

```java
@Entity
class Forum extends AbstractAggregateRoot {
    Discussion startDiscussion(
        DiscussionId discussionId,
        Author author,
        String subject) {

        if (this.isClosed()) {
            throw new IllegalStateException("Forum is closed");
        }
        Discussion discussion = new Discussion(
            this.tenant(),
            this.forumId(),
            discussionId,
            author,
            subject);
        registerEvent(new DiscussionStarted(...));
        return discussion;
    }
}
```

* 포럼이 열린 경우에만 토론을 시작할 수 있다. : `this.isClosed()` 
* 인자 5개 중 3개만 있으면 된다. : 나머지 2개는 포럼에서 제공
* 역시나 유비쿼터스 언어가 표현된다. : `startDiscussion`

## 서비스의 팩토리

```java
package com.saasovation.collaboration.domain.model.collaborator;

interface CollaboratorSerice {
    Author authorFrom(Tenant tenant, String identity);

    Creator creatorFrom(Tenant tenant, String identity);

    Moderator moderatorFrom(Tenant tenant, String identity);

    Owner ownerFrom(Tenant tenant, String identity);

    Participant participantFrom(Tenant tenant, String identity);
}
```

```java
// infrastructure.services !!!
package com.saasovation.collaboration.infrastructure.services;

class UserRoleToCollaborationService implements CollaboratorSerice {
    @Override
    public Author authorFrom(Tenant tenant, String identity) {
        return (Author) userInRoleAdapter.toCollaborator(
            tenant, identity, "Author", Author.class);
        )
    }
}
```

```java
package com.saasovation.collaboration.domain.model.collaborator;

class Author extends Collaborator {
    ...
}
```

* 기술적 구현이므로 *인프라 계층*의 모듈에 위치한다.
* 어댑터에 의존한다.
    * `UserInRoleAdapter`는 외래 컨텍스트와 의사소통 책임만 갖는다.
    * `CollaboratorTranslator`는 바운디드 컨텍스트 내 도메인 객체로 변환 책임만 갖는다.
* 협업에서는 `identity` 식별자와 액세스에서는 `username`

## 마무리

* 유비쿼터스 언어로 애그리게잇을 생성
* 애그리게잇의 팩토리 메서드 VS 서비스의 팩토리 메서드