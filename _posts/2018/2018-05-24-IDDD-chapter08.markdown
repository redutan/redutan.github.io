---
layout: "post"
title: "IDDD 8장. 도메인 이벤트"
date: "2018-05-24 00:59"
tags:
    - IDDD
    - DDD
    - BOOK
---

Publish-Subscribe

## 언제 그리고 왜 도메인 이벤트를 사용할까?

* 도메인 내 어떤 사건이 발생했을 때
* 한 트랜잭션에는 한 애그리게잇만 커밋

## 이벤트의 모델링

어떤 명령에서 어떤 사건이 발생했었음

* Command : `BacklogItem#commitTo(Spring)`
* Event : `BacklogItemCommitted`
* 백로그 항목이 커밋됐다.(과거형)

```java
package com.saasovation.agilepm.domain.model.product;

@Value
class BacklogItemCommitted implements DomainEvent {
    Date occurredOn;
    BacklogItemId backlogItemId;
    SpringId committedToSprintId;
    TenantId tenantId;
}
```

```java
package com.saasovation.agilepm.domain.model;

interface DomainEvent {
    Date occurredOn();
}
```

*추가사항*

* 멱등성
* 더 풍부한 상태 전달 (이벤트 소싱!)

### 애그리게잇의 특성과 함께하기

이벤트를 애그리게잇을 통해서 영속화

### 식별자

* 이벤트를 애그리게잇으로 모델링하면 식별자가 필요하다.
* 도메인 이벤트를 바운디드 컨텍스트 외부로 발행 시 식별자가 필요(With rabbitmq)
    * 외부 구독자 입장에서는 멱등성 관리를 위해서 식별자를 할당할 수 있음
    * `equals`로 일정 부분 해결할 수도 있다 (in 로컬 바운디드 컨텍스트)

## 도메인 모델에서 이벤트 발행하기

Light-Weight Publish-Subscribe

![발행자](https://camo.githubusercontent.com/afd4f6f7085bc7bdc1db8595fd4e3e62fccb12d6/687474703a2f2f7777772e706c616e74756d6c2e636f6d2f706c616e74756d6c2f7376672f5a4c31393369386d33427074354a643238482d654b307a4b74393575475039514f3937344c50654b7a56556141774b42756331506358644673315231616d723657616b34796b484f685836694a734158527a584c4536795a4c79514532616a5846486c31696f5272584539496a746635725a6c49387a55316a6f30687652337278627276446a3266523653466e704f4a512d35583258687748726347344d5950665a6b67354a6a5638524e6d4d6c627a784a585739787635356c334642392d59356e556350503051676d55334a6878714d6c765a362d50504a59522d6b38644432356a344d7143727a323132444e5f4f4b68625a55553868755570494e7a704576636c35366d3030)

* Reference : [https://github.com/redutan/redutan.github.io/wiki/Domain-Event-on-Springframework](https://github.com/redutan/redutan.github.io/wiki/Domain-Event-on-Springframework)

### 발행자

```java
class BacklogItem extends AbstractAggregateRoot<BacklogItem> {
    void commitTo(Sprint spint) {
        // Some domain logic
        super.registerEvent(new BacklogItemCommitted(
                this.tenantId(),
                this.backlogItemId(),
                this.sprintId()
        ));
    }
}
```

```java
class BacklogItemService {  // Application Service
     @TransactionalEventListener
    void commitBacklogItem(...) {
        backlogItem.commitTo(sprint);   // Publish BacklogItemCommitted event
    }
}
```

### 구독자

```java
class BacklogItemService {  // Application Service
     @TransactionalEventListener
    void handleBacklogItemCommitted(BacklogItemCommitted event) {
        // BacklogItemCommitted 구독 후 처리
    }
}
```

중요한 것은 **결과적 일관성**

## 뉴스를 원격 바운디드 컨텍스트로 전파하기

*시스템 간* 결과적 일관성 확보

### 메시징 인프라의 일관성

*구현방법*

1. 도메인 모델과 메시지 인프라 저장소 공유
2. 원격 DB with XA
3. 이벤트 저장소

### 자치 서비스와 시스템

자치 서비스 : 이벤트를 통해서 시스템 간 결합도(독립성!)를 줄이는 기법 (No-RPC)

자치 서비스의 단위는 바운디드 컨텍스트가 되면 좋은 것 같다.

### 지연 시간 허용

*결과적 일관성*을 위해서

도메인 별로 그 때 그 때 달라요.

시스템의 허용치를 만족시키면서도 잘 수행되도록 아키텍처 품질을 높여야 한다.

## 이벤트 저장소

이벤트의 상태를 유지하기 위해서 저장하는 것이 요구되는 경우가 많다.

![IdentityAccessEventProcessor](http://www.plantuml.com/plantuml/svg/bP7FIWGn3CRlVOeUzT0Na4LMq8Cd1_O9-YSmaKuxDEcAFhsrmx0Pg8Atv4k-xqVQCx4jN9UeNWCaHlvyyXw8Ngwjcqh-gNFHvb4_vyLYwlgbEl857HJze1Dy_CSxLHUHvcwbFUVkNbdFUBKCmrqrXf_CRycpJI52brjsWB_J1rE16SBxMPl4kK13sdM558OqwHGuuLSYgWM_kNVmV862DkBNzbPxqmZ7vLw4hctVSGE8aJITZ3cC0WmTDn68CASZTrU5MqYZ6y-GGbtYDm00)

```java
@Aspect
class IdentityAccessEventProcessor {
    @Before("execution(* com.saasovation.identityaccess.application.*.**..))")
    public void listen() {
        DomainEventPublisher.instance()
            .subscribe(new DomainEventSubscriber<DomainEvent>() {
                public void handleEvent(DomainEvent aDomainEvent) {
                    store(aDomainEvent);
                }

                public Class<DomainEvent> subscribedToEventType() {
                    return DomainEvent.class;   // 모든 도메인 이벤트
                }
            });
    }
    private void store(DomainEvent aDomainEvent) {
        EventStore.instance().append(aDomainEvent);
    }
}
```

```java
class StoreEvent {
    void append(DomainEvent aDomainEvent) {
        String eventSerializatoin = 
            EventStore.objectSerializer().serialize(aDomainEvent);
        StoredEvent storedEvent = 
            new StoredEvent(
                aDomainEvent.getClass().getName(),
                aDomainEvent.occuredOn(),
                eventSerialization);
        this.session().save(storedEvent);
        this.setStoredEvent(storedEvent);
    }
}
```

```sql
CREATE TABLE tbl_stored_event (
    event_id int(11) NOT NULL auto_increment,
    event_body varchar(65000) NOT NULL,
    occurred_on datetime NOT NULL,
    type_name varchar(100) NOT NULL,
    PRIMARY KEY (event_id)
)
```

## 저장된 이벤트의 전달을 위한 아키텍처 스타일

### 레스트품 리소스로서 알림 발행하기

1. 이벤트를 REST/WEB API로 발행한다. (이벤트 아이디나 애그리게잇 아이디를 전달)
    * 이벤트 저장소를 통해서 조회하거나, 애그리게잇을 직접 조회한다.
2. 구독 측에서 발행측에 다시 상세 이벤트 정보를 조회한 후 처리한다.

### 메시징 미들웨어를 통한 알림 발행

1. 메시징 미들웨어를 통해서 이벤트를 발행한다.
2. 메시징 미들웨어를 구독한 구독 측에서 발행 측의 상세 이벤트 정보를 조회한 후 처리 한다.

발행과 구독은 exchange나 queue 개념으로 연결된다.

## 구현

Skip

* 멱등 수신자 처리가 중요하다 : At least once

> 멱등성 : 오퍼레이션이 두 번 이상 수행되어도, 한 번만 수행했을 때와 같은 결과에 이르는 동작을 의미

궁극적으로 이벤트를 추적해야하는 경우가 생기기 때문에 이벤트 저장 기능이 거의 필수적으로 요구된다.
그리고 추적 일관성을 위해서 이벤트 저장은 도메인 로직과 트랜잭션으로 묶이는 것이 중요하다.

## 마무리

* 이벤트 만의 고유 식별자가 요구된다.
    * 이벤트 저장소도 필요
* 어짜피 발행-구독!
* 멱등성 또는 중복 발행 or 수신 제거 확보
