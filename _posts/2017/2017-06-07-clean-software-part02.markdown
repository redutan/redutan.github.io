---
layout: post
title: 클린소프트웨어 Part 2. 애자일 설계
date: '2017-06-07 21:09'
tags:
  - books
  - clean-software
  - SOLID
  - SRP
  - OCP
---

# 서론

지금 당장을 위해서 설계와 구현을 동시에 진행한다.

### 잘못된 설계의 증상

* 경직성(Rigidity) : 설계를 변경하기 어려움
* 취약성(Fragility) : 설계가 망가지기 쉬움
* 부동성(Immobility) : 설계를 재사용하기 어려움
* 점착성(Viscosity) : 제대로 동작하기 어려움
* 불필요한 복잡성(Needless Complexity) : 과도한 설계
* 불필요한 반복(Needless Repetition) : 마우스 남용
* 불투명성(Opacity) : 혼란스러운 표현

### 원칙

[SOLID][e29eaf06]

  [e29eaf06]: https://ko.wikipedia.org/wiki/SOLID "SOLID"

### 악취와 원칙

정말 소중한 원칙이지만, **원칙에 대한 맹종은 불필요한 복잡성이란 설계의 악취로 이어진다**

# Chapter 7. 애자일 설계란 무엇인가?

> 설계는 우선적으로 소스 코드에 의해 문서화되며, 소스 코드를 표현하는 다이어그램은 설계에서 부수적인 것일 뿐, 설계 그 자체는 아니다
> - 잭 리브스(Jack Reeves)

* 설계 ≠ UML
* 설계 = 코드

## 소프트웨어에서 어떤 것이 잘못되는가?

재설계(renewal)보다는 지속적으로 발전시켜야 한다.

## 설계의 악취: 부패하고 있는 소프트웨어의 냄새

위 [잘못된 설계의 증상][a668b17f] 참고

  [a668b17f]: #section-1 "잘못된 설계의 증상"

### 애자일 팀은 소프트웨어가 부패하도록 내버려두지 않는다

지속적 리팩토링

## 애자일 개발자는 해야 할 일을 어떻게 알았는가?

1. 그들은 다음과 같은 애자일 실천방법으로 문제를 찾아냈다. - 유연하지 않음
2. 그들은 설계 원칙을 적용해 문제를 진단했다. - OCP, DIP
3. 그리고 적절한 디자인 패턴을 적용해 문제를 해결했다. - 전략패턴

**위와 같이 소프트웨어 개발의 세 측면 사이에서 일어나는 상호작용이 바로 설계 작업이다.**

## 가능한 좋은 상태로 설계 유지하기

* 다음은 없다. 나쁜 냄새가 나면 바로 개선한다.
* 깨진 유리창이 하나라도 생기면 슬럼화 되는 것은 금방이다.

## 결론

* **애자일의 설계는 과정이지, 결과가 아니다.**
* 시스템을 좋은 상태로 유지하려는 노력이다.

# 단일 책임 원칙(SRP)

**응집도**

## 단일 책임 원칙(SRP)

> 한 클래스는 단 한가지의 변경 이유만을 가져야 한다.

![하나 이상의 책임](https://www.plantuml.com/plantuml/img/Iyv9B2vMSCxFBIWjIIp9pCzBp75FpSzDBIcgT2meoCbC1Wjo9OEL1QKcboJcfUUaAbHpAG21TafHOhc69eITM9IQgA5ffP2INvgKayfL2zNZ7ke9OnKb5cG03Sn1DfYGpGgwkdPA4AEL4FPpOJCBh1JY8cAKWbs6y96k7OWF0000)

*하나 이상의 책임*

![분리된 책임](https://www.plantuml.com/plantuml/img/Iyv9B2vMSCxFBIWjIIp9pCzBp75FpSzDBIcgT2meoCbC1Wjo9OEL1QKcboJcfUUaAbHpAG21TafHOhc69bSjL1wgCpCPGs5YKMgYXgQLGaf-QL9EAa93g2UCLPHOa06qBGVPLaBEKj3LjSDYAHV2UjsSrBGIx8gmsGWsa0Wb87SZMM87uWC0)

*분리된 책임*

**SRP를 지키지 않으면**

* 책임이 섞임으로 불필요한 컴파일 발생
* 부수효과 발생할 확률이 높아짐

### 책임이란 무엇인가?

책임(responsibility) : `변경을 위한 이유`

**변경의 축은 변경이 실제로 일어날 때만 변경의 축이다.** 아무 증상도 없는데 이 문제에 SRP나 다른 원칙을 적용하는 것은 현명하지 못하다. - 오버엔지니어링?

## 결론

책임이라는 정의가 어플리케이션이 현 상황에 따라서 달라지기 때문에 항상 적용하기 어려운 것 같다.
중요한 것은 **요구사항의 변경에 맞게 적절하게 책임을 분배** 하는 것이라고 생각한다.

# 개방 폐쇄 원칙(OCP)

> 모든 시스템은 생명주기 동안에 변화한다. 이것은 개발 중인 시스템이 첫 번째 버전보다 오래 남길 원한다면 반드시 염두에 두어야 할 사실이다.
> - 이바르 야콥슨(Ivar Jacobson)

## 개방 폐쇄 원칙(OCP)

> 소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.

* **경직성** 악취를 해결할 수 있음
* OCP가 잘 적용된다면, 이미 제대로 동작하고 있던 원래 코드를 변경하는 것이 아니라 새로운 코드를 덧붙임으로써 나중에 그런 변경을 할 수 있게 된다.

## 상세 설명

1. 확장에 대해 열려 있다.
2. 수정에 대해 닫혀 있다.

## 해결책은 추상화다

일반화(Generalization) 을 통한 추상화 : 인터페이스 or 추상클래스

![Client는 개방 폐쇄 원칙에 어긋난다](https://www.plantuml.com/plantuml/img/Iyv9B2vMSCx9JCqhuKe6Su9JYyfIYxWWOWgwTZ010000)

*Client는 개방 폐쇄 원칙에 어긋난다.*

![전략 패턴: Client는 개방 폐쇄 원칙을 따른다](https://www.plantuml.com/plantuml/img/Iyv9B2vMSCx9JCqhuShCAqajIajCJeKAUCBuNCbWPS6fHMMfHLmGIGLTEmnb40KAUgK5UZMOiW00)

*전략 패턴: Client는 개방 폐쇄 원칙을 따른다.*

**왜 AbstractServer이 아니라 ClientInterface일까?**

* **추상 클래스(인터페이스)는 자신을 구현하는 클래스보다 클라이언트에 더 밀접하게 관련되어 있기 때문이다.**

## 예상과 '자연스러운' 구조

> 모든 상황에서 자연스러운 모델은 없다!

* **폐쇄는 완벽할 수 없기 때문에, 전략적이어야 한다.** 설계자는 자신의 설계에서 닫혀 있는 변경의 종류를 선택해야 한다.
  * 폐쇄는 추상화에 기반한다.
* 경험으로 얻은 통찰력이 필요하다.
* 가장 중요한 것은 미리 설계하지 말고, **변경이 일어날 때까지 기다리는 것** 이다.

## 결론

* OCP는 객체 지향 설계의 심장이다.
* **어설픈 추상화를 피하는 일은 추상화 자체만큼이나 중요하다.**
