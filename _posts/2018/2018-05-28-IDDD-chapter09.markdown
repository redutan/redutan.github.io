---
layout: "post"
title: "IDDD 8장. 모듈"
date: "2018-05-28 13:44"
tags:
    - IDDD
    - DDD
    - BOOK
---

* Java: 패키지
* C#: 네임스페이스
* Roby: 모듈

## 모듈로 설계하기

* 모듈 : 도메인 객체의 컨테이너

**규칙**

* 모델링의 개념에 맞춰 모듈을 설계하자
* 유비쿼터스 언어에 맞춰 모듈을 명명하자
* 모델에서 사용하는 일반적인 컴포넌트 타입이나 패턴에 따라서 기계적으로 모듈을 생성하지 말자
* 느슨하게 결합된 모듈을 설계하자.
* 결합이 필요하다면 짝이 되는 모듈 사이에서 비순환적 의존성이 형성되도록 노력하자
* 자식 모듈과 부모 모듈 사이에 규칙은 느슨하게 하자
* 모듈을 모델의 정적인 개념에 따라 만들지 말고, 모듈이 담고 있는 객체에 맞추도록 하자

> 주방의 서랍에 식기류가 포크와 나이프와 스푼별로 잘 정리돼 있음

## 기본 모듈 명명 규칙

Java: `com.saasovation`
C#: `SaaSOvation`

## 모델을 위한 모듈 명명 규칙

**바운디드 컨텍스트**

* `com.saasovation.identityaccess`
* `com.saasovation.collovoration`
* `com.saasovation.agilepm`

**모듈 구성 예시**

* `com.saasovation.identityaccess`
    * `domain.model`

결론은 여러분 팀의 몫이다.

## 애자일 프로젝트 관리 컨텍스트의 모듈

* `com.saasovation.agilepm`
    * `domain`
        * `model`
            * `product`
                * `backlogitem`
                * `release`
                * `sprint`
            * `tenant`
            * `team`

이 팀은 모듈 사이의 결합도 문제보다 *정리*에 중점을 뒀다.

## 다른 계층 속의 모듈

| 계층 | 모듈 | 비고 |
|-----|-----|-----|
| 사용자 인터페이스 | `com.saasovation.agilepm.resources` | `ui` |
| 애플리케이션 | `com.saasovation.agilepm.application` |  |
| 도메인 | `com.saasovation.agilepm.domain` |  |
| 인프라 | `com.saasovation.agilepm.infrastructure` | `adapter` |

## 바운디드 컨텍스트보다 모듈

모듈은 응집력이 낮은 경우 분리하기 위한 컨테이너이고, 바운디드 컨텍스트는 그저 경계일 뿐이다.

어짜피 모듈은 상향식(bottom-up)으로 설계하게된다. 
모듈 내의 컴포넌트들에 따라서 나눠질 수도 합쳐질 수도 있다. 중요한 것은 도메인 로직과 유비쿼터스 언어에 따르는 것이다.

## 마무리

* 유비쿼터스 언어를 표현한다.
* 모듈 설계의 예시!!
* 기계적인 모듈 설계는 창의성을 방해한다.
* 바운디드 컨텍스트 분리보다 모듈 사용이 먼저다.



