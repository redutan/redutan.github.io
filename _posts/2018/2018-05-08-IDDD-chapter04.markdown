---
layout: "post"
title: "IDDD 4장. 아키텍처"
date: "2018-05-08 01:36"
tags:
    - IDDD
    - DDD
    - BOOK
---

> 실제 요구(도메인)가 아키텍처 스타일과 패턴의 사용을 유도해야 한다. 

* 도메인이 기술 보다 먼저다.

## 성공한 CIO와의 인터뷰

1. Server-Client Architecture
2. **계층형 아키텍처**
    * DIP, QP, PSA, DDD-Lite
3. **헥사고날 아키텍처**
    * To Mobile, Cloud
4. **CQRS**
    * Materialized Views
5. Event-Driven Architecture 
    * Pipeline + Filter
6. Saga 패턴
    * Long-lived transaction as a sequence of subtransactions.
    * In a distributed system
7. 이벤트 소싱
    * 모든 변경을 추적 (ex: 보안 감사 등)

**이러한 아키텍처 덕분에 결국 회사에 돈이 된다**

## 계층

![DDD Layred Architecture](https://i0.wp.com/www.ajlopez.com/images/articles/dddlayered.png)

`그림 4.1` DDD가 적용된 전통적인 계층 아키텍처

[출처] https://ajlopez.wordpress.com/2008/09/12/layered-architecture-in-domain-driven-design/

* 도메인 모델과 비즈니스 로직을 도메인 계층에 격리
* 하위 계층에만 의존 - **아래에서 위쪽으로 직접 참조 불가** (하지만 Observer 사용 가능)
    * 느슨한 연결 : UI가 애플리케이션(바로 하위) 뿐만 아니라 도메인이나 인프라에도 의존 가능

### UI 계층

* View나 API 제공
* 도메인 모델이 아닌 표현 모델(DTO)를 사용 추천

### 애플리케이션 계층

UI로 부터 매개변수를 받아 리파지토리를 사용해 애그리게잇을 획득하고, 커맨드를 위임

* 보안과 트랜젝션 담당 (일반적으로 보안을 UI로 올리는 것 같음)
* 도메인 로직 없으며 도메인 모델(애그리게잇, 도메인 서비스 등)에 위임
* 도메인 모델에서 발행한 도메인 이벤트를 구독(subscribe)

```java
@Transactionl
void commitBackLogItemToSpring(
        String aTanantId, String aBackLogItemId, String aSprintId) {
    // 재료(Aggregate) 준비
    BacklogItem backlogItem = 
        backlogItemRepository.backlogItemOfId(aTanantId, aBacklogItemId);
    Sprint sprint sprintRepository.sprintOfId(aSprintId);
    // 커맨드 위임
    backlogItem.commitTo(sprint);
}
```

### DIP

> * 상위 수준의 모듈은 하위 수준 모듈에 의존해선 안된다. 둘 모두는 반드시 추상화에 의존해야 한다.
> * 추상화는 세부사항에 의존해선 안 된다. 세부사항은 추상화에 의존해야 한다.

**하위 구현 컴포넌트가 상위 콤포넌트가 정의한 인터페이스에 의존해야한다.**

어라 그러다 보니 계층이 없어진다. **여기에 대칭성을 더하면 어떻게 될까?**

## 헥사고날 또는 포트와 어댑터

![Hexagonal Architecture](http://alistair.cockburn.us/get/2304)

*헥사고날 아키텍처*

[출처] http://alistair.cockburn.us/Hexagonal+architecture

![Texi handling Hexagonal Architecture](https://camo.githubusercontent.com/440658bfd34094b4b705c40ae0517e0b00a0becb/68747470733a2f2f7777772e6e67696e782e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031352f30352f47726170682d30312d65313433313937383039303733372e706e67)

*Texi handling Hexagonal Architecture*

[출처] https://github.com/seongminwoo/study/blob/master/7-part_series_about_microservices.md

### 헥사고날의 장점

* 기능적 요구사항에 따라 애플리케이션 내부를 설계
    * UI, Infra는 그 다음이다.
    * 애플리케이션 내부가 캡슐화 된다.
* Adapter를 통해 기술과 분리

## 서비스 지향(SOA)

**중요한 것은** 기술 보다 비즈니스(도메인) 가치가 우선해야 한다

## REST: 표현 상태 전송

![The glory of REST](https://martinfowler.com/articles/images/richardsonMaturityModel/overview.png)

[출처] https://martinfowler.com/articles/richardsonMaturityModel.html

### RESTful HTTP 서버의 주요 특징

1. Resource : URI
2. Verbs : GET, POST, PUT, DELETE, ...
3. Hypermedia : 연결된 리소스를 제공. 상호작용, 클라이언트 무상태

### RESTful HTTP 클라이언트의 주요 특징

Hypermedia를 바탕으로 상호 작용한다.

### REST와 DDD

**도메인 모델을 RESTful로 바로 노출하는 것은 좋지 않다.**

* 도메인 모델의 변경이 API와 연결되기 때문에 변경에 취약해지며, 클라이언트와 호환성이 깨진다.

**해결책**

1. 도메인 모델 -> 표현 모델을 조립 (추천)
    * Assembler + DTO (PoEAA)
2. 각 상황 별 공유 도메인 모델 사용
    * 공유커널

### 왜 REST인가

느슨함

## CQRS

사용자가 필요로하는 데이터 뷰를 리파지토리로 쿼리하기란 어려울 수 있다.

`CQRS = 객체 설계 원칙 + CQS`

1. Command : 객체의 상태를 수정, Void 형
2. Query : 값을 반환, 수정 X

**구현**

* 애그리게잇은 오직 커맨드 메소드만 가지고 있음.(커맨드 모델)
* 쿼리를 위한 뷰 전용 모델(쿼리 모델)을 생성

### CQRS의 영역 살펴보기

![CQRS Architecture](https://docs.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/images/cqrs-logical.svg)

출처 : [https://docs.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/cqrs](https://docs.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/cqrs)
참고 : [https://docs.microsoft.com/ko-kr/azure/architecture/patterns/cqrs](https://docs.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/cqrs)

#### 클라이언트와 쿼리 처리기

#### 쿼리 모델(읽기 모델)

* 필요한 수 만큼 뷰를 지원하기
* 실용적으로 하라
* 데이터베이스 테이블 뷰가 오버헤드의 원인이 되지 않을까?

#### 클라이언트가 커맨트 처리를 주도한다.

UI to Argument

#### 커맨드 처리기

* 카테고리 스타일 : 1 class n methods
* 전용 스타일 : 1 class 1 method

#### 커맨드 모델(쓰기 모델)은 행동을 수행한다.

```java
public void commitTo(Sprint aSprint) {
    ...
    DomainEventPublisher
        .instance()
        .publish(new BacklogItemCommitted(
            this.tenant(),
            this.backlogItemId(),
            this.sprintId()
        ));
}
```

* 어떤 경우든 쿼리 모델을 업데이트시키기 위해선 도메인 이벤트를 게시해야 한다.

#### 이벤트 구독자가 쿼리 모델을 업데이트 한다.

* 동기도 가능하고 **비동기**도 가능하다.

### 결국은 일관성이 유지되는 쿼리 모델 다루기

> Eventually consistent

지연되는 뷰 데이터 동기화를 해결해야한다.

* 낙관적 업데이트 기법!
* 클라이언트에서 publish/subscribe 기법

**결국은 동기화되는 지연시간이 문제**

## 이벤트 주도 아키텍처(EDA)

> 이벤트의 생산, 감지, 소비와 이벤트에 따른 응답 등을 촉진하는 소프트웨어 아키텍처

From **도메인 이벤트**

### 파이프와 필터

`$ cat phone_number.txt | grep 303 | wc -1`

* [Spring Cloud Data Flow](https://cloud.spring.io/spring-cloud-dataflow/)

![필터를 처리하는 이벤트를 보냄으로써 파이프라인이 만들어진다.](http://7u2mak.com1.z0.glb.clouddn.com/pipe-filter.jpg)

`그림 4.8` 필터를 처리하는 이벤트를 보냄으로써 파이프라인이 만들어진다.

[출처] http://zhangyi.farbox.com/post/coding/understand-scala-stack

이는 가상의 예제이자 개념적 부분을 강조했을 뿐이다. 실제 엔터프라이즈에선 큰 문제를 좀 더 작은 단계로 나누기 위해 이 패턴을 사용하며, 좀 더 쉽게 분산 처리를 이해하고 관리하도록 해준다.
또한 여러 시스템이 오직 자신이 할 일(도메인)만을 걱정하게 되게 해주기도 한다.

### 장기 실행 프로세스

1. 컴포지트
2. 애그리게잇 집합
3. 이벤트

역시나 중요한 것은 **결과적 일관성**(Eventual Consistency)

#### Saga Pattern

**Events/Choreography**

![Saga Sequence](https://blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.13.39-PM.png)

**Rollback**

![Saga Rollback](https://blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.36.17-PM.png)

* referecne : [https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/](https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/)

### 이벤트 소싱

거의 완벽한 수준의 변경 추적

![이벤트 소싱](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/_images/event-sourcing-overview.png)

[출처] [https://docs.microsoft.com/ko-kr/azure/architecture/patterns/event-sourcing](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/event-sourcing)

Replay : 이벤트소싱에서 이벤트를 되감아서 특정 버전 상태로 되돌리는 것
* 하지만 최신 버전 Replay는 병목의 원인
* 이를 해결하기 위해서 최신 버전의 상태를 *Snapshot*으로 지정해서 최적화

## 데이터 패브릭과 그리드 기반 분산 컴퓨팅

분산 캐시

ex) [hazelcast](https://hazelcast.org/)

### 데이터 복제

* 캐시 master/slave
* 복제를 통한 장애 극복
    * 이벤트 유실 제거 (이벤트 publish/subscribe도 가능)

### 이벤트 주도 패브릭과 도메인 이벤트

For Domain Event

### 지속적 쿼리

For CQRS

### 분산 처리

For Saga 또는 배치 병렬 처리

## 마무리

* Layerd Architecture > DIP > Hexagonal Architecture
* SOA, REST, 분산 컴퓨팅
* CQRS, Event Sourcing, pipe/filter, Saga

