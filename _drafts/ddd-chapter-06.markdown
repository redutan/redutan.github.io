---
layout: post
title: "DDD 06장 - 도메인 객체의 생명주기"
date: "2015-11-25 21:39"
tags: ddd
---

![도메인객체_라이프사이클](/images/2015/11/ddd-lifecycle.png)

*그림 1 | 도메인 객체 생명주기*

<br/>

**도메인 객체 관리의 문제**

1. 생명주기 동안의 무결성 유지하기
2. 생명주기 관리의 복잡성으로 모델이 난해해지는 것을 방지하기

#### 해결을 위한 3가지 패턴

**1. Aggregate(집합체)** : Domain 캡슐화. 접근 Root

**2. Factory** : Domain 객체(또는 Aggregate) 생성 캡슐화

**3. Repository** : 인프라스트럭쳐 캡슐화 및 객체지향적 접근 제공

<br/>

## Aggregate(집합체)

객체들이 밀접한 연관관계를 맺는 **객체 집합** 에는 불변식이 적용돼야 한다. - 일관성 있는 객체의 상태를 유지하기 위해서.

> 불변식 : 데이터가 변경될 때마다 유지돼야 하는 일관성 규칙

모델 내의 참조에 대한 캡슐화의 추상화가 필요하고 그것이 바로
**Aggregate**(데이터 변경의 단위로 다루는 연관 객체의 묶음)이다.

- 루트(Root) 와 경계(Boundary)가 있다.

![지역 및 전역 식별성과 객체 참조](/images/2015/11/ddd-aggregate-1.jpg)

*그림 2 | 지역 및 전역 식별성과 객체참조*

> 참조 : http://www.moshesty.com/category/ddd/

**Aggregate 구현 규칙**

- 루트 Entity는 전역 식별성을 지니며, 궁극적으로 불변식을 검사할 책임이 있다.
- 각 루트 Entity는 전역 식별성을 지닌다. 경계 안의 Entity는 지역 식별성을 지니며, 이러한 지역 식별성은 해당 Aggregate 안에서만 유일하다.
- Aggregate 경계 밖에서는 루트 Entity를 제외한 Aggregate 내부의 구성요소를 참조할 수 없다.
- 데이터베이스 질의(Query)를 이용하면 Aggregate의 루트만 직접적으로 획득할 수 있다.
- Aggregate 안의 객체는 다른 Aggregate의 루트만 참조할 수 있다.
- 삭제 연산은 Aggregate 경계 안의 모든 요소를 한 번에 제거해야한다.
- Aggregate 경계 안의 어떤 객체를 변경하더라도 전체 Aggregate의 불변식은 모두 지켜져야 한다.

다시 정리하면

**Entity와 Value Object를 Aggregate로 모으고 각각에 대해 경계를 정의하라.
한 Entity를 골라 Aggregate의 루트로 만들고 Aggregate 경계 내부의 객체에 대해서는 루트를 거쳐 접근할 수 있게 하라.
Aggregate 밖의 객체는 루트만 참조할 수 있게 하라.
내부 구성요소에 대한 일시적인 참조는 단일 연산에서만 사용할 목적에 한해 외부로 전달될 수 있다.
루트를 경유하지 않고 Aggregate의 내부를 변경할 수 없다.
이런 식으로 Aggregate의 각 요소를 배치하면 Aggregate 안의 객체와 전체로서의 Aggregate의 상태를 변경할 때 모든 불변식을 효과적으로 이행할 수 있다.**

## Factory
