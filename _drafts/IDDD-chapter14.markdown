---
layout: "post"
title: "IDDD 14장. 애플리케이션"
date: "2018-06-17 18:46"
tags:
    - IDDD
    - DDD
    - BOOK
---

* 사용자 인퍼페이스와 도메인 모델 사이에 위치
    * 유스케이스 태스크를 조정
* 트랜잭션, 보안 권한 부여 담당

> 애플리케이션을 핵심 도메인 모델과 상호 교류하며 이를 지원하기 위해 잘 조합된 컴포넌트의 집합을 의미하기 위해 사용하고 있다.
> 이는 일반적으로 도메인 모델 그 자체와 사용자 인터페이스, 내부적으로 사용되는 애플리케이션 서비스, 인프라적 컴포넌트를 뜻한다.
> 이 각각의 영역에 어떤 것들이 들어가는지는 애플리케이션마다 다르며, 사용하는 아키텍처가 무엇인지에 따라서도 달라진다.

![특정한 한 가지 아키텍처에 국한되지 않은 주요 애플리케이션 영역. 여전히 각기 다른 영역의 추상화에 의존적인 인프라의 DIP를 강조한다.](/images/2018/06/IDDD-figure-14-1.png)

`그림 14.1` *특정한 한 가지 아키텍처에 국한되지 않은 주요 애플리케이션 영역. 여전히 각기 다른 영역의 추상화에 의존적인 인프라의 DIP를 강조한다.*

`ui.Controller` `-구현->` `infra.TomcatController` `-위임->` `application.Service` `-구현->` `infra.TransactionalService` `-위임->` `domain.Repository` `-구현->` `infra.JpaRepository` `-의존->` `infra.Mysql`

* 필자는 `UI` -> `Appcliation` -> `Infra` 순서로 설명해 나간다. (Top-Down 방식)

## 사용자 인터페이스

UI 종류

1. Web 1.0 렌더링 되는 HTML
2. Web 2.0 DHTML + Ajax
3. 데스크톱 애플리케이션 + HTTP 통신

### 도메인 객체의 렌더링

![사용자 인터페이스는 다수의 애그리게잇 인스턴스의 속성을 렌더링할 필요가 있지만 한 번에 하나의 인스턴스를 수정하도록 요청한다](/images/2018/06/IDDD-figure-14-2.png)

`그림 14.2` 조회 시에는 복수 의 애그리게잇. 명령 시에는 한 개의 애그리게잇

### 애그리게잇 인스턴스로부터 데이터 전송 객체를 렌더링하기

DTO + Assembler

> 개인적으로 가장 선호하는 방식

### 애그리게잇 내부 상태를 발행하기 위해 중재자를 사용하자

중재자 패턴

> 개인적으로 비추 도메인 모델이 (약하긴 하지만) UI에 의존하게 됨

### 도메인 페이로드 객체로부터 애그리게잇 인스턴스를 렌더링하라

도메인 객체를 내부에 담고 있는 DPO(Domain Payload Object)로 렌더링 하는 방식

* 지연로딩 이슈가 있기 때문에 OSIV와 같은 기술과 함께 쓰길 권장한다.

### 애그리게잇 인스턴스의 상태 표현

View Model? Presentation Model?

> DTO와 차이점을 모르겠다. 상태를 표현한다는 것으로 보아 불변이 가능한 정도 차이인 것인가?

### 유스케이스 최적 리파지토리 쿼리

특정 유스케이스에 특화된 쿼리 메소드 제공.

CQRS와 구분하자.

### 다수의 개별 클라이언트 처리하기

이건 SKIP 하자. 최근의 MVC 프레임워크에서 지원하므로 Application의 책임이 아니라고 본다.

### 변환 어댑터와 사용자 편집의 처리

Adapter + Command (Or Presentation Model)

주로 명령(편집, 수정) 시에 사용하는 방식

## 애플리케이션 서비스

* **애플리케이션 서비스를 얇게 유지(파사드)**
* **도메인 모델로 위임**

### 애플리케이션 서비스 예제

원시타입의 나열 VS **인자 (커맨드) 객체**

* *원시타입이 너무 많이 나열되면 그것은 안티패턴이라고 보고 가능하면 인자(Or 커맨드)로 캡슐화 하는 것이 좋아 보인다.*

> 단순 인자이며 명령을 캡슐화 하지 못하는데 커맨드 패턴이라고 부르는 것이 어색해 보인다.

```java
package com.saasovation.identityaccess.application;

class TenantIdentityService {
    @Transactional  // 트랜잭션과 보안 처리
    @PreAuthorize("hasRole('SubscriberRepresentative')")
    public void activeTenant(TenantId tenantId) {
        // 도메인 모델로 위임
        this.nonNullTenant(tenant).activate();
    }

    @Transactional(readOnly = true)
    public Tenant nonNullTenant(TenantId tenantId) {
        Tenant tenant = this.tenant(tenantId);
        if (tenant == null) {
            throw new IllegalArgumentException("Tenant does not exists");
        }
        return tenant;
    }
}
```

### 결합이 분리된 서비스 출력

```java
    @Override
    @Transactional(readOnly = true)
    public void findTenant(TenantId tenantId) {
        Tenant teannt = 
                this.tenantRepository.tenantOfId(tenantId)
        this.tenantIdentityOutputPort().write(tenant);
    }
```

최근 추세는 어쨋든 사용자에게 출력에 대한 책임은 가능한 UI가 하는 것이 좋다고 본다.
이는 애플리케이션의 책임이 아닌 것 같다. 

> 그러한 것은 컴포넌트로 추상화 해서 UI 모듈에 있는 것이 더 어울리는 것 같다. 개인적으로 출력은 반환 값이 있는 것이 더 직관적으로 보인다.

## 여러 바운디드 컨텍스트 묶기

![UI에서 다수의 모델을 구성해야만 할 때가 있다. 이 그림에서는 하나의 애플리케이션 계층을 사용해서 세 개의 모델을 구성한다.](/images/2018/06/IDDD-figure-14-3.png)

`그림 14.3` UI에서 다수의 모델을 구성해야만 할 때가 있다. 이 그림에서는 하나의 애플리케이션 계층을 사용해서 세 개의 모델을 구성한다.

* 애플리케이션 계층이 **유스케이스**를 관리 - UI 계층이 아니다!!
* 제품, 토론, 리뷰 컨텍스트가 분리 VS 하나의 컨텍스트로 통합 > **결론은 모델 정제!!**

개인적으로 이러한 상황에서는 UI에서 각각 쿼리 해서 조합하던가, API-GW에서 조합하는 것이 좋다고 본다.

물론 특정 컨텍스트 내에서 조합해야 한다면 *애플리케이션 서비스*가 가장 어울리는 장소인 것 같다.

## 인프라

![애플리케이션 서비스는 도메인 모델의 리파지토리 인터페이스에 의존적이지만, 인프라의 구현 클래스를 사용한다. 패키지는 넓은 범위의 책임을 캡슐화 한다](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuIf8JCvEJ4zLU3DrmTifFQ-NhNcpf-7Dt2rlMcSeL7CfA2Jd91ONAuIavYNcbNYcfEQLP9PK1gSMbMKcfuBaWK0xCRaaioon91MYI4CJ8bg2uDLorSAjUTtVydhbb3TpTxnUjU9rxmwm6Pbv9Qb5QOd9gL1xWb8ByeipI_ABAa7I2CFyqpnJC0m46lLsIilhkNkGdEkHcPHQb0Tq4ePvcRa5EQcvG6yKOzW5D1ExDtNjCDJYKAcdPuVRRYw7rBmKO8W30000)

`그림 14.4` 애플리케이션 서비스는 도메인 모델의 리파지토리 인터페이스에 의존적이지만, 인프라의 구현 클래스를 사용한다. 패키지는 넓은 범위의 책임을 캡슐화 한다

## 엔터프라이즈 컴포넌트 컨테이너

EJB VS Spring

## 마무리

* 도메인 모델 -> UI 모델
* 사용자 입력 -> 도메인 모델
* 애플리케이션 서비스의 책임
* DIP로 도메인 모델과 기술(인프라) 분리 및 유연성 확보