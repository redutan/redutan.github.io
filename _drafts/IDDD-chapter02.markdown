---
layout: "post"
title: "IDDD 2장. 도메인, 서브도메인, 바운디드 컨텍스트"
date: "2018-04-28 00:38"
tags:
    - IDDD
    - DDD
    - BOOK
---

서브도메인과 바운디드 컨텍스트로 전략적으로 설계해보자.

## 큰그림

### 서브도메인과 바운디드 컨텍스트의 활용

![서브도메인과 바운디드 컨텍스트를 포함한 도메인](https://s3.amazonaws.com/media-p.slid.es/uploads/dragoshont/images/145866/Snap_2013-11-08_at_11.17.34.png)

온라인 쇼핑몰 도메인
* 전자상거래 시스템 : 상품 카탈로그, **주문**, 송장, 배송 서브도메인
* 재고관리 시스템 : 재고관리
* 외부 예측 시스템 : ?

*제품 카탈로그, 주문, 송장, 배송 모델 등을 하나로 엮어서 복잡도가 높아지며 부정적인 결과를 초래*

**소프트웨어의 관심사를 도메인을 바탕으로 분명하게 분리시켜야 한다.**

상호 의존성을 갖기 때문에, 서브도메인으로 나누지 않는다면 변화가 계속됨에 따라 훨씬 **큰 부담**을 지게 된다.

일반적으로 하나의 바운디드 컨텍스트에 하나의 서브도메인이 있는 것이 좋다. : 재고관리는 좋아 보인다.
* 물론 DSL이 포함되어야 한다.
* 같은 언어일지라도 개념은 전혀 다를 수 있다.

### 핵심 도메인에 집중하기

![서브도메인과 바운디드 컨텍스트를 포함하는 추상적인 비즈니스 도메인](/images/2016/04/IDDD-2.2.jpg)

* 핵심도메인 : 주문
* 지원 서브도메인 : 배송, 상품, 송장
* 범용 서브도메인 : 회원(인증과 권한)

> 결국 도메인 전문가와 소통으로 정제하는 것이 중요하다.

## 왜 전략적 설계가 엄청나게 필수적인가

> 협업 개념이 사용자 및 권한과 강한 결합을 가지는 것이 옳은가?

* 협업 개념에서는 회원은 그저 **작성자**일 뿐이다.

![팀이 전략적 설계의 기초를 이해하지 못했고, 이는 공동 모델에서 개념들이 잘못 찢어지도록 했다. 점선 안의 문제가 되는 요소들이다.](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKhEpot8pqlDAr5usx_czVmzI0AlDlMywPxpQaS3igAs1QyNBk0gI4pEJanFLQX6adhJjERDh9Llvar0Dc9RpzkfSxXgaPNDtVDcmLGCWJVpMbylMCh51N65WcvfWIwIYN2de4j0t8Ck2Z5IGJSblpmFLHnX5OOiijLGXoFiHHQ5MueGGhKH8OYm3atA8JKl1HZy0000)

`그림 2.3` 팀이 전략적 설계의 기초를 이해하지 못했고, 이는 공동 모델에서 개념들이 잘못 찢어지도록 했다. 점선 안의 문제가 되는 요소들이다.

협업 도구는 **사용자의 역할에 집중해야지,** 사용자가 누구며 수행권한이 부여된 행동이 무엇인지에 관심이 있어선 안 된다. 하지만 위에서는 뒤섞여 버렸다.

* *포럼*은 단지 토론을 게시하고 싶은 *작성자*만 있으면 된다.
* 사용자와 권한은 협업 컨텍스트에서 독립돼야 한다.

Big ball of mud : 거대한 진흙공 - 빈약한 도메인으로 가득찬 모노리식 애플리케이션

**이런 사용자와 권한 서브도메엔을 범용 서브도메인이라고 한다.**

이윽고 팀이 비협업 개념의 또 다른 집합을 모델링하는 상황이 오면, 핵심 도메인은 더욱 불확실해진다. 
풍부한 협업의 유비쿼터스 언어를 제대로 소스 코드에 반영하지 못한 채, 이를 은영준에 내포하고 있을 뿐인 모델을 만들고 말 수도 있다. 
팀은 비즈니스 도메인과 그에 따른 서브데메인은 물론이고, 반드시 그들이 개발하고 있는 바운디드 컨텍스트도 제대로 이해해야한다.
이를 통해 전략적 설계를 가로 막는 비열한 적인 거대한 진흙공의 더러운 물을 막을 수 있다.

## 현실의 도메인과 서브도메인

### 문제점 공간과 해결책 공간

도메인은 문제점 공간(problem space)과 해결책 공간(solution space)을 모두 갖고 있다.

* 문제점 공간 : 새로운 핵심 도메인을 만들기 위한 전체 도메인의 일부 > *핵심 도메인과 서브 도메인의 조합*
* 해결책 공간 : 해결책을 소프트웨어로 구현 > *바운디드 컨텍스트*

**서브 도메인을 1:1로 바운디드 컨텍스트로 묶는 것은 좋은 목표이다** 하지만 항상 그렇지는 않다.

![구매나 재고관리와 관련된 핵심 도메인과 다른 서브도메인.](https://s3.amazonaws.com/media-p.slid.es/uploads/dragoshont/images/145864/Snap_2013-11-08_at_11.12.05.png)

`그림 2.4` 구매나 재고관리와 관련된 핵심 도메인과 다른 서브도메인. 이 관점은 특정 문제점 공간 분석을 위해 선택된 *서브도메인으로 제한되며 전체 도메인에 적용할 수는 없다.*

* 최적 매입 컨텍스트 = 핵심 도메인
* 구매 컨텍스트 + ERP 구매 모듈 = 구매 지원 서브 도메인
* 재고 관리 컨텍스트 + ERP 재고 모듈 = 재고 관리 지원 서브 도메인
* 매핑 컨텍스트 : 범용 서브 도메인

*최적 매입 컨텍스트를 개발하는 회사 입장에서의 이런 핵심 포인트*를 살펴봤음을 기억하자.
지리적 매핑 서비스는 문제점 공간에서 재고관리 서브도메인의 일부로 간주하지만, 해결책 공간에서 재고관리 컨텍스트가 아니다.
매핑 서비스가 해결책 공간에선 간단한 컴포넌트 기반 API로 제공됐다 하더라도, 이는 다른 바운디드 컨텍스트다.
**재고 관리와 매핑의 유비쿼터스 언어는 상호 배타적이며, 이는 두 요소가 서로 다른 바운디드 컨텍스트라는 의미다.**
재고 관리 컨텍스트가 외부 매핑 컨텍스트의 어떤 부분을 사용하는 상황에서, 데이터는 적어도 최소한의 **변환**을 거쳐야만 적절히 사용될 수 있다.

한편 *구독자를 위해 매핑 서비스를 개발*하고 제공하는 외부 비즈니스 조직의 관점에서 보면 **매핑은 핵심 도메인**이다.

전략적 이니셔티브 : KPI를 달성하기 위한 핵심적인 활동이나 계획

## 바운디드 컨텍스트 이해하기

> 바운디드 컨텍스트는 그 안에 도메인 모델이 존재하는 **명시적 경계**

* 바운디드 컨텍스는 명시적이고 언어적이다

> 컨텍스트가 왕이다. in DDD

**모델의 중앙화, 범용화는 Evil 이다.** 컨텍스트 별로 모델은 다른 의미를 가진다.

![두 바인디드 컨텍스트 속 어카운트 객체는 의미가 서로 완전 다르지만, 각 바운디드 컨텍스트 안에서 고려해야만 그 사실을 알 수 있다.](https://www.codeproject.com/KB/architecture/1158628/2.png)

`그림 2.5` 두 바인디드 컨텍스트 속 어카운트 객체는 의미가 서로 완전 다르지만, 각 바운디드 컨텍스트 안에서 고려해야만 그 사실을 알 수 있다.

| 컨텍스트 | 어카운트 의미 | 예 |
| 은행 | 계좌 | 적금 계좌 |
| 문학 | 이야기 | A Personal Account of Mt. Everest Disaster |

모든 것을 빠짐없이 포괄하는 모델을 생성하려 시도하는 함정에 빠지며, 어디서든 통용되는 유일한 의미를 가진 이름의 개념에 대해 전체 조직이 모두 동의하는 결과를 목표로 삼는다.
이런 모델링 접근법에는 구멍이 있다.
1. 모든 이해관계자로부터 모든 개념이 하나의 순수하고 구분된 글로벌한 의미를 갖는 것에 대해 동의를 얻기란 거의 불가능하다.
2. ~~모든 사람을 함께 모으는 것은 절대 불가능하다.~~ : 이건 좀 아닌 것 같음.

최상의 선택을 위해선 언제나 차이점은 존재한다는 사실을 직시하고 ,바운디드 컨텍스트를 통해 차이점이 명확하며 잘 이해하고 있는 **도메인 모델을 각각 기술해야 한다.**

1 바운디드 컨텍스트 != 1 프로젝트 아티팩트(산출물)

![](http://static.olivergierke.de/lectures/ddd-and-spring/images/bounded-context1.png)

* 주문은 카탈로그 컨텍스트와 주문 컨텍스트에서 다른 모델로 표현된다.
* 고객은 등록, 계정, 배송 컨텍스트 별로 다른 모델로 표현된다.

### 모델 그 이상을 위해

아래 항목들도 바운디드 컨텍스트 경계 안에 있다.

* Application Service : 보안 트랜젝션 관리 : 퍼사트
* UI
* Api Client : API 통합

**안티패턴**

* Smart UI

### 바운디드 컨텍스트의 크기

 = **유비쿼터스 언어를 표현하기 위해 필요한 크기**

> 딱 내가 원했던 만큼의 음표들이 있었습니다. 더도 덜도 아닌
> - 모짜르트

**잘못된 크기의 바운디드 컨텍스트를 만들게 되는 이유?**

1. 아키텍처적 영향을 기준으로 삼는 경우(플랫폼, 프레임워크, 컴포넌트, 인프라 등)
2. 개발자 리소스(또는 팀)를 작업을 분배하기 위해

### 기술적 컴포넌트로 정렬하기

`com.mycompany.optimalpuchasing` : bounded context

* `com.mycompany.optimalpuchasing.prsentation` : ui
* `com.mycompany.optimalpuchasing.application` : application
* `com.mycompany.optimalpuchasing.domain.model` : domain
* `com.mycompany.optimalpuchasing.infrastructure` : infra

## 샘플 컨텍스트

![완전히 서브도메인과 정렬된 바운디드 컨텍스트 샘플의 평가 관점](https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/graphics/02fig07.jpg)
`그림 2.7` 완전히 서브도메인과 정렬된 바운디드 컨텍스트 샘플의 평가 관점

> 출처 : https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/ch02lev1sec5.html

### 협업 컨텍스트

![협업 컨텍스트](http://www.plantuml.com/plantuml/svg/06y0208pKI4rpEskR_L8tD163T1536Up3Wxw3lM78GYo-K9C3reSW2C8euDkghRDj8VT_n2GH8YG_wTWwwvXROr3K8tpf4UtKY6CEiYDWcBtXOobUU4gvHZdtVicv453I441Ai05rbqeFrTJm6khXvGpcl_cvHA7gJwknxhSJI_1DDKqnePg6aveE5RmywnOwdNKZespkdEvdOt3tpNZzx7-WZ-JRVRP0egnVLoaPg0xLZHGQNqViVf0f4mA0g7caeEOqCN5oMka5e44FZLCHqg4FtNNmBZm1g3ixHn7ShB855vKuCrxLx0q0)

*before*
```java
public class Forum extends Entity {
    public Discussion startDiscussion(...) {
        // repository 의존
        User user = userRepository.userFor(this.tenantId(), aUsername);
        // 보안 로직
        if (!user.hasPermissionTo(Permission.Foru.StartDiscussion))
        //...
        // 열차 충돌
        String authorName = user.person().name().asFormattedName();
        //...
        return newDiscussion;
    }
}
```

* **협업 보다는 보안을 염두함** in 협업 컨텍스트
* 전술적 패턴(DDD-Lite)으로는 해결 불가능. **전략적 설계 컨텍스트 맵!!**

*열차사고*

* `usr.person().name().asFormattedName()` 
* [디미터의 법칙](https://en.wikipedia.org/wiki/Law_of_Demeter) 위배

결론적 해결책을 통해 **추가적인 전략적 설계를 사용해, 재사용 가능한 모델을 별도의 바운디드 컨텍스트로 분리하고 적절히 통합할 수 있게 됐다**

*after*
```java
public class ForumService {
    @Transactional
    public Discussion startDiscussion(...) {
        Author author = this.colloboratorSrvice.authorFrom(tenant, anAuthorId);
        Discussion new Discussion = forum.startDiscussionFor(...);
        //...
        return newDiscussion;
    }
}
public class Forum extends Entity {
    public Discussion startDiscussionFor(...) {
        //...
    }
}
```

* `User`와 `Permission`의 의존을 제거하고 모델을 엄격히 협업에만 집중하게 함 `Author`

### 식별자와 엑세스 컨텍스트

사일로 효과 : 연통 배관 : 상호 협력을 하지 않고 중복이 생기고 각각 따로 놀게됨

![식별자 컨텍스트](http://www.plantuml.com/plantuml/png/06ue20CGlxfS_eGLPi5zfFHuLTrCtBw540-3VAr-Km3wVW6zk3OJeuQ_QDTUTYiHAAe3v67gG1PSHySDSjOwACHbt4eGlpy5xb2n9g5rIqUAEPxEVTKWGo1nl-uEkZhTYzORdrHqvKH1MaOY3pLxX5UlM4lfvKkLGZBi7g1GDfqsgFGYDgekUAeAfBvwf_uDIA4QZYb1HfRbcBqLkJ4OXN9_i6wIE97oXRholfAzI2-O73AFTBTHuGZM6Wj2nvd-OjLMWxy5CauIwTsR-wMNKhiBX5x7G3aePycFe0G00)

* 위에서 지적된 것 처럼 식별자 컨텍스트를 분리하고 범용 지원 도메인으로써 활용

### 애자일 프로젝트 관리 컨텍스트

![애자일 프로젝트 관리 컨텍스트](http://www.plantuml.com/plantuml/svg/06_8201GXoQowhI_rXtR4OhCPifp3VuCtYFZprxnqF-INLxMafe6DVj9iyUZKVHx6bxf1mSIGZgu2tiSDPE5lnFRWAnn5FZ6VEuSkog4K37yHsDC9ml9QLAN1pwVjL2-6xxwaHMhULx2I32Geli348AS_mPk0xkbgIreveL3HHacdInonOTLGbfPvqAm82Tsp6xaDG9q-qeE9JRiPF791SvnubXGRhKYZC2gUyzbLxbOAoHnK48IdJBaivjeybCjr8G-Jt12xbR7_H9bnOe2J3WNoawhfXZfZTGC0)

* DDD 전략적 설계를 바탕으로 애자일 프로젝트 컨텍스트를 분리

> 컨텍스트는 각 팀에게 아주 구체적인 의미를 부여한다.

## 마무리

* 도메인, 서브도메인, 바운디드 컨텍스트
* 문제점 공간, 해결책 공간
* 모델을 구분(User vs Author)
* 바운디드 컨텍스트의 크기
* 사스오베이션 팀의 바운디드 컨텍스트 정제
