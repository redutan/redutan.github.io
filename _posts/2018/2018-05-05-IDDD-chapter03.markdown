---
layout: "post"
title: "IDDD 3장. 컨텍스트 맵"
date: "2018-05-05 02:08"
tags:
    - IDDD
    - DDD
    - BOOK
---

바운디드 컨텍스트 간 관계를 그리는 것으로써 **해결책 공간**에 초점을 맞추가 있다.

## 컨텍스트 맵이 필수적인 이유

![추상적 도메인의 컨텍스트 맵.](https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/graphics/03fig01.jpg)

`그림 3.1` 추상적 도메인의 컨텍스트 맵

[출처] https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/ch03lev1sec1.html

* U = 업스트림
* D = 다운스트림

> *컨텍스트 맵*은 상호 교류하는 시스템의 목록을 제공하고, 팀 내 의사소통의 촉매 역할을 한다.

### 프로젝트와 조직 관계

**통합 패턴들**

* 파트너십(Partnership) : 두 컨텐스트가 한 트랜젝션으로 묶임 - 2 phase commit?
* 공유 커널(Shared kernel) : 상호 의존하는 공유 모델을 관리 - *안티 패턴이라고 봄*
* 고객-공급자(Customer-Supplier Development) : 업스트림(서버:공급자), 다운스트림(클라이언트:고객)로 단방향 의존 표현
* 순응주의자(Conformist) : 업스트림(서버) is King
* **부패 방지 계층(Anticorruption Layer)** : 변환을 통해서 다운스트림 컨텍스트 내 순수함을 지킴 (Adapter + Translator)
* **오픈 호스트 서비스(Open Host Service)** : REST/API, RPC, Socket
* **발행된 언어(Published Language)** : Json, XML, Byte
* 분리된 방법(Seprate Ways) : 의존 없음
* 큰 진흙공(Big ball of mud) : 똥덩어리

### 세 가지 컨텍스트를 매핑하기

*before*

![Big ball of mud](https://buildplease.com/img/ballofmud.png)

* 문제점 공간
* *분리된 핵심*을 이용

*after*

![Separated](https://buildplease.com/img/integrations.png)

* 해결책 공간

![Context Map Example](https://www.codeproject.com/KB/architecture/1158628/6.jpg)

* 대부분 *순응주의자*를 많이 사용하는 것 같다.
* 상호 의존이 걸리는 *partnership*도 많은 것 같다.
* 개인적으로는 *고객-공급자*가 좋다.
* 일반적으로 **핵심 도메인은 다운스트림**이다.
* 업스트림에서 다운스트림으로 모델 복제는 *Evil*이다.

[출처] https://www.codeproject.com/Articles/1158628/Domain-Driven-Design-What-You-Need-to-Know-About-

![](http://josecuellar.net/wp-content/uploads/contextmap3.png)

* 통합 전에 ACL를 통해서 변활할 시 최소한의 속성만 있으면 된다.

**식별자와 액세스 컨텍스트의 통합**

![식별자 접근 부패방지 Sequence Diagram](http://www.plantuml.com/plantuml/png/XP51gi8m48RtEKMMkl02BgH5H5qemii5ncGU3hIJaamBzVH6IYabO1RP_6I-R_YdYW91-hPHO8K64DGtR9yO_aQsh-2PtXXK7kdTOVw8OI2BUgzR89RqfZnkjihX3sfcd41gZKsUgqCMah6s5cEyUw5_iY3akNRG2ORaZWjuqS-2Ca6L7McHYp6FOqF8aepdaobFBJMP01mR4F_TLlmKhYigqh9giXFqdKka39vrN26xTFGF)

* 중요한 것은 애자일 프로젝트 컨택스트에 식별자 도메인이 침투하지 못하게 하는 것

**협업 컨텍스트와 통합**

결과적 일관성이 중요하다. - 이벤트 기반 아키텍쳐

그렇다면 상태관리를 해야한다.

```java
enum DiscussionAvailability {
    ADD_ON_NOT_ENABLED, NOT_REQUESTED, REQUESTED, READY;
}

@Value
class Discussion {
    DiscussionAvailiability availability;
    DiscussionDescriptor descriptor;
    ...
}

class Product extends Entity {
    private Discussion discussion;
    ...
}
```

* Product에서 Discussion을 사용할 시 `READY`인 경우에만 가능

![프로젝트 관리 컨텍스트와 협업 컨텍스트 통합](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuIf8JCvEJ4zLU3jZuflfhMzshtZRslkcQydRhXqArLmA2iavYSN52cxvHQMvGQd5-QL5oQbmKUV4dDIybCGYk4Gj5qJ28oGam3adCpMl12hWd9-JavHVb5YIcP_dc99OK99Q19LnoInEBYqkpiyBAKhCAyv9BCal0jjRaW-L0UhGGB5UqmeXem2tCZWv8pMbD2SpBnt388IK1ip52BCGaborYB2Oql9wuPmt2-O2G-7LbeRNozPWX0l2zVaGfmId5fLb8WKEmc2OJ2qNfd85NLqxhA63q4v89HOn1Q1Ip835eJGdDQq4g1vX8I4DkdR84OnWWcLGeWfD4ZF1E02vm4G80000)

* 협업.Calendar -> 프로젝트관리.Scheduling
* 협업.Forum -> 프로젝트관리.Discussion

## 마무리

* OHS, PL, ACL
