---
layout: "post"
title: "IDDD 5장. 엔터티"
date: "2018-05-10 01:23"
tags:
    - IDDD
    - DDD
    - BOOK
---

> 개발자는 도메인보다 **데이터**에 초점을 맞추려는 경향이 있다.

속성만 있고 행위는 없는 데이터홀더

## 엔터티를 사용하는 이유

`Object + Identity = Entity` + Life Cycle(Mutable:가변성)

비즈니스 관계자는 보통 근사한 데이터베이스 테이블 편집기를 개발하는 데만 너무 많은 노력을 들인다.
올바른 도구를 선택하지 않았다면 공들여 다룬 CRUD 기반 솔루션의 비용은 너무 커진다.
CRUD가 타당한 선택일 때가 Django, RoR등과 같은 언어 및 프레임워크가 비로소 합리적일 수 있는 순간이다.

**하지만 대부분의 엔터프라이즈 애플리케이션에서는 Entity가 좋은 선택이다.**

## 고유 식별자

### 사용자가 식별자를 제공한다.

자연키

* 변경될 가능성이 크고, 사용자를 신뢰하기 힘들다.

### 애플리케이션이 식별자를 생성한다.

UUID, Random, Timestamp NanoSeconds, ...

**읽을 수 있는 식별자**

`APM-P-08-14-2012-F36AB21C` : 2012년 08월 14일에 생성된 애자일프로젝트관리(APM)의 Product(P)

```java
@Entity
class Product {
    private ProductId productId;

    public Date creationDate() {
        return productId.creationDate();
    }
}
```

* 개인적으로 위 패턴을 선호하지 않는다.
* 대리키를 사용하고 표현이 필요한 코드는 따로 Unique 값으로 하는 것이 좋은 것 같다.

### 영속성 메커니즘이 식별자를 생성한다.

[JPA @GenerationType](https://docs.oracle.com/javaee/7/api/javax/persistence/GenerationType.html)
* AUTO, IDENTITY, SEQUENCE, TABLE

### 또 하나의 바운디드 컨텍스트가 식별자를 할당한다.

SKIP : 비추천

### 식별자 생성의 시점이 문제가 될 때

**식별자 생성 시점**
1. `Repository`에 **저장 후** 식별자가 생성
2. `Repository`에 **저장 전** 식별자가 생성

**저장 후 식별자 생성 시 부작용**
1. **도메인 이벤트**를 발행하는 경우 식별자가 있어야지 정확히 발행 가능 : *문제 없음 - 아래 코드 참조*
2. `Set`에 추가할 시 다른 식별자와 같아질 수 있음 : *문제 없음 - 충돌가능성 없음*

```java
@Entity
class Product {
    @Id @GeneratedValue
    Long id;
    String name;

    Product(String name) {
        ...
        this.registerEvent(new ProductCreatedEvent(this))
    }
}
```

* 위 코드 처럼 이벤트를 발행해도 구독 시점에는 `id`속성이 할당되어 있어서 문제없이 이벤트 처리됨

> 저자는 빠른 식별자 할당을 선호한다고 했지만, 현재 기술 상으로 **지연 할당도 전혀 문제 없다.**

### 대리 식별자

[JPA @GenerationType](https://docs.oracle.com/javaee/7/api/javax/persistence/GenerationType.html)
* AUTO, IDENTITY, SEQUENCE, TABLE

== 영속성 메커니즘이 식별자

### 식별자 안정성

수정 불가(값 객체), 엔터티 수명주기 동안 안정적으로 유지

* 식별자 `Setter`를 제거한다.
* 식별자 `Setter`가 존재하면 유효성 체크를 강화한다. > `하지만 대리키를 사용해서 Setter가 존재하지 않는 것이 나은 것 같음`

**나는???**
* 왠만하면 단일 대리키를 사용한다. = 식별자 생성 지연
* 식별자는 `Getter`만 사용한다.
* 다른 접근이 요구되면 Unique 키로 보조한다.

## 엔터티 발견과 그들의 내부적인 특성

데이터 중심 모델 > Getter + Setter > 무기력한 도메인 모델

**엔터티 발견**

유비쿼터스 언어로 도메인 전문가와 대화하면서 나오는 개념들 

하지만
* 명사(이름) : 엔터티
* 동사(행위) : 메서드

위처럼 단순하게 뽑아낸다고 생각하면 **실수하는 것이다.**

결국에는 도메인에 대한 토론을 통해 **지식 탐구**와 **정제**를 거쳐야지 깊은 도메인 모델을 얻을 수 있다.

### 엔터티와 속성을 알아내기

*Example : User*

* 사용자는 테넌시와 관련이 있고 테넌시의 제어를 받는다.
* 시스템의 사용자는 반드시 인증돼야 한다. : Need 검색
* 사용자는 이름과 연락처를 비롯한 개인정보를 갖고 있다.
* 사용자의 개인정보는 사용자나 관라자에 의해 변경될 수 있다. : 값 객체?
* 사용자의 보안 인증(비밀번호)은 변경될 수 있다.

![앞선 발견에 따른 Tenant와 User라는 두 개의 엔터티](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKhEIImkLWX9pKlCAr6miN7DAyaigRIpWug75gSM8OiwfEQb07K10000)

`그림 5.5` 앞선 발견에 따른 `Tenant`와 `User`라는 두 개의 엔터티

*추가 스팩*

* 테넌트는 초대를 통한 많은 사용자의 등록을 허용한다.
* 테넌트는 활성화될 수 있고, 비활성화될 수도 있다.
* 시스템의 사용자는 반드시 인증돼야 하지만, 테넌트가 활성화되니 경우에만 인증이 가능하다.

![엔터티가 발견해서 이름 지은 후에는 이를 고유하게 식별하고 찾을 수 있도록 해주는 속성을 알아내자](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKhEIImkLWX9pKlCA_5CKR2n2KlCAKsriqEH00gxvfLabbJQsIbKSoaev2Ncfbef19SKPUQbSzL2bOOMfnQXAom5YY4h1WeL0DMMvnUb8EdFooz9LKZABod9pxLIUBspvVM6XjUREjxEsF6wQuh2cwbJWAotCwUydZ3dTVSIBeHJTNMXpaCH0xk3oo4rBmNeFG00)

`그림 5.6` 엔터티가 발견해서 이름 지은 후에는 이를 고유하게 식별하고 찾을 수 있도록 해주는 속성을 알아내자

*확인 스팩*

* 테넌트 : 식별자와 엑세스 서비스와 그 밖에 다른 온라인 서비스의 명명된 조직적 구독자, 초대를 통해 사용자를 등록
* 사용자 : 테넌티 내에 보안 주체. 고유 사용자명과 암호화된 비밀번호를 가진다.
    * `SecurityPricipal` : 인증 주체 값 객체 - 추후에 알아보자
* 암호화 서비스 : 암호화, 복호화

### 필수 행동 파헤치기

*Tenant의 스팩*

* 테넌트는 활성화되거나 비활성화될 수 있다.

```java
@Entity
class Tenant {
    void activate() {
        // TODO 구현
    }
    void deactivate() {
        // TODO 구현
    }
}
```

![테넌트에 필수적인 행동 추가](http://www.plantuml.com/plantuml/svg/LSw_2i903CVn_PuYemvz0Ib77HoS_RE7NgY1QuRaLq74T_T4GJf-VXc-6GBiMEQQnieHT1PZmx5Gtr-vBfBpwj3cWq7no9cUYSXubXsTu6fJ8u_GEqCssuOYAshiF_p2PTA0-2N4s_1A_sxdEjtG_O9f42ljlJS0)

`그림 5.7` 테넌트에 필수적인 행동 추가

* `activate()` : 테넌트 활성화
* `deactivate()` : 테넌트 비활성화
* `isActive()` : 사용자는 *테넌트가 활성화된 경우*에만 인증이 가능하다.
    * 인증을 위해 `AuthenticationService`가 필요
* `registerUser()` : 테넌트는 초대를 통해서 *사용자 등록*을 허용한다.

*User의 스팩*

* 사용자는 이름과 연락처를 비롯한 개인정보를 갖고 있다. : `Person` Entity 분리
* 사용자의 개인정보는 사용자나 관리자에 의해 변경될 수 있다. : `change?()`
* 사용자의 보안 인증(비밀번호)은 변경될 수 있다. : `changePassword()`

![User 상세 모델링](http://www.plantuml.com/plantuml/svg/RP2zIWH148JpUOeEDTWNE8xXG0mk4S6VlDrjpuNTRfdfHKGE30n44uCL50nz037IPnhl7Umx1svXpOQlcggQcaN5e5tRkBB16E6O65dd5KodfzXqv7qMJY85W_kijLvx3pSEe3F6sD84ZZJKl31qQRTN4ge1AY-G5tIOXPtTBQ8GXR4vC8j_y9wmOgbpFfVGejR2ThHqB4fm9ghIJY1ztwMFs_HvlthvIWyz_3ptzbgzzkdfBJs-v-v_ZeFscOyJHjzUTnl0xJn5iPd4RNOfVCvmEQemCdOVgcjZDoEkRFjV)

`그림 5.8` User 상세 모델링

### 역할과 책임

**어떤 설계를 하더라도, 유비쿼터스 언어가 기술적 선호보다 항상 우위에 서도록 하자. DDD에서는 비즈니스 도메인 모델이 가장 중요하다.**

### 생성

고정자(Invariant:불변식)을 지킨다. > 엔터티의 전체 수명주기에 걸쳐 트랜잭션적 일관성이 유지돼야 하는 상태

복잡한 엔터티 생성을 위해서 `Factory`를 사용한다.

### 유효성 검사

* 특성 : 불변?
* 속성 : 가변?

DBC(Design By Contract) : 계약의 의한 설계

* Pre condition, Post condition, Invariant로 설명
* 하지만 Java 등의 언어에서는 보호절로써 처리함 : 방어적 프로그램

**유효성 체크의 책임 DB가 아니라 도메인이 져야한다.**

* 유효성 검사의 책임이 너무 커지면 도메인 모듈(패키지)내 `Validator` 클래스 분리한다.
* with 명세 패턴, 전략 패턴

*유효성 체크 책임 위치*

* 엔터티
* 엔터티에서 분리된 Validator로 위임
* 도메인 서비스

## 변화 추적

가장 실용적인 변경 추적 : 도메인 이벤트 + 이벤트 저장소 = 이벤트 소싱!

* [Spring Data(Hibernate) envers](https://docs.spring.io/spring-data/envers/docs/2.0.7.RELEASE/reference/html/)도 꽤 쓸만하다.

## 마무리

* 식별자
* 유비쿼터스 언어를 바탕으로 엔터티 모델링
* 엔터티 구성, 유효성 검사, 변경 추적 등 구현
