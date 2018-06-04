---
layout: "post"
title: "IDDD 10장. 애그리게잇"
date: "2018-06-03 19:26"
tags:
    - IDDD
    - DDD
    - BOOK
---

**일관성 경계** 내에서 엔터티와 값 객체의 묶음

* **일관성 경계**의 기준은 같은 트랜잭션인가로 검증된다.
* 애그리게잇 내의 불변식(invariant)?

## 스크럼 핵심 도메인에서 애그리게잇 사용하기

*기능 목록*

* 제품은 백로그 아이템과 릴리스, 스프린트를 포함한다.
* 새로운 제품 백로그 아이템을 계획했다.
* 새로운 제품 릴리스를 계획했다.
* 새로운 제품 스프린트 일정을 수립했다.
* 계획된 백로그 아이템에 관한 릴리스 일정을 수립할 수 있다.
* 일정이 잡힌 백로그 아이템은 스프린트로 커밋할 수 있다.

### 첫 번째 시도: 큰 클러스터의 애그리게잇

> 제품이 ~를 포함한다.

컴포지션 VS 객체 그래프
포함 VS (상호) 연결

*도메인 로직*

* 백로그 항목을 스프린트로 커밋하면, 이를 시스템에서 제거하도록 허용해선 안 된다.
* 스프린트가 백로그 항목을 커밋하면, 이를 시스템에서 제거하도록 허용해선 안 된다.
* 릴리스가 백로그 항목의 일정을 수립하면, 이를 시스템에서 제거하도록 허용해선 안 된다.
* 백로그 항목의 릴리스 일정을 수립하면, 이를 시스템에서 제거하도록 허용해선 안 된다.

```java
class Product {
    private Set<BacklogItem> backlogItems;
    private String description;
    private String name;
    private ProductId productId;
    private Set<Release> releases;
    private Set<Spring> sprints;
    private TenantId tenantId;
}
```

![아주 큰 애그리게잇으로 모델링된 Product](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKhEIImkLWWeoayfJIvHiB5nJ4ylIarCBqbL2ChFBx6pWofmIapEpibFzon9pGKgSiqhoIofX4i6fUQa9XQdOae45nHbvfKWYyCiKZ9KKj3IrRLJK3BGqzDIGZOVbngODRZaeRPnEQJcfG1z1W00)

*아주 큰 애그리게잇으로 모델링된 Product*

![Product와는 별도의 애그리게잇 타입으로 모델린된 연관된 개념들](http://www.plantuml.com/plantuml/svg/TOv12i8m54JtFKKka1kKKDrrfLHm_qs6Z_hJIF9pVmKjhK6tCu-PDnIbh3LAvuLACSUSGlLg-dx7d46iC5DAwjmtC8ONSYQfC8VB3Nu5zkJladXKn6M5gLsP8A22_y3faQ-p_keNG-jMbsvZPUrMeMa-lqtwFki6pA56UG80)

*Product와는 별도의 애그리게잇 타입으로 모델린된 연관된 개념들*

큰 애그리게잇으로 모델링하다보면 변경에 취약해져서 업데이트 상황에서 버전 충돌이 발생할 가능성이 커진다. 위를 예시로 두면 한 명이 backlogItem을 변경하고 다른 한 명이 Spring를 변경할 시 직적접 연관이 없음에도 불구하고 버전 충돌이 발생해서 업데이트가 실패할 확률이 커진다. (애그리게잇의 크기가 커짐에 따라서 충돌 확률도 더 커짐)

### 두 번째 시도: 다수의 애그리게잇

*하나의 큰 애그리게잇 상 Product.java*

```java
class Product {
    public void planBacklogItem(
        String summary, String category, BacklogItemType type, StoryPorints storyPoints) {
        ...
    }
    ...
    public void scheduleRelease(
        String name, String description, Date begins, Date ends) {
        ...
    }
    ...
    public void scheduleSprint(
        String name, String goals, Date begins, Date ends) {
        ...
    }
    ...
}
```

*여러 개로 분리된 애그리게잇 상 Product.java*

```java
class Product {
    public BacklogItem planBacklogItem(
        String summary, String category, BacklogItemType type, StoryPorints storyPoints) {
        ...
    }
    ...
    public Release scheduleRelease(
        String name, String description, Date begins, Date ends) {
        ...
    }
    ...
    public Sprint scheduleSprint(
        String name, String goals, Date begins, Date ends) {
        ...
    }
    ...
}
```

* 일종의 factory 메서드로써 동작한다.

*Product***Srvice 예시*

```java
@Service
class ProductBacklogItemService {
    @Transactional
    public void planProductBacklogItem(
        String tenantId, String productId
        String summary, String category
        String backlogItemType, String storyPoints) {

        Product product = 
            productRepository.producetOfId(
                new TenantId(tenantId),
                new ProductId(productId));
        BacklogItem plannedBacklogItem = 
            product.planBacklogItem(
                summary, category,
                BacklogItemType.valueOf(aBacklogItemType),
                StoryPoints.valueOf(stroyPoints));
        backlogItemRepository.add(plannedBacklogItem);
    }
    ...
}
```

이와 같이 우린 밖으로 빼서 모델링함으로써(Modeling it away) 트랜잭션 실패 문제를 해결했다. 이제 `BacklogItem`, `Release`, `Sprint`등의 인스턴스가 사용자의 요청에 따라 얼마든지 동시적으로 안전하게 생성될 수 있다.

그러나 큰 애그리게잇을 조금 다듬어서 동시성 문제를 해결할 수도 있을지도 모른다. 하이버네이트 매핑에서 `optmistic-lock` 옵셥을 `false`로 설정해 트랜잭션 실패가 도미노 처럼 전달되는 상황을 피할 수 있다.

## 규칙: 진짜 고장자(invariant)를 일관성 경계 안에 모델링하라

**중요한 것은 진짜 고정자를 이해하는 것이다.*

고장자(invariant) : **일관성(트랜잭션 일관성)을 유지해야만 한다는 비즈니스 규칙**

* 트랜잭션 일관성 : 동기적, 원자적
* 결과적 일관성 : 비동기적

**한 트랜잭션에 한 애그리게잇만 인스턴스만 포함** : 이는 너무 가혹한 것 같다.

## 규칙: 작은 애그리게잇으로 설계하라

![큰 클러스터 `Product` 애그리게잇](http://www.plantuml.com/plantuml/svg/VP1DIiOm443tEKNeij3Y0KgfIXU2eFGJzsConDZyI39PzFQs4YNGq4qNynwTl9aYGQ1a3HC6OkIlmSiaY0_3lL81GH7onNiQnomyW5YDLq-4TfTcHvgsVxYWGOXu1hVle1sTvsyGruejFb4cWvUx7hsrcWZbfJL7qXP8U_VirSx2jZllO1Bobuyl54VONtFRTIDlxlg-RShCAi-bDPPZMV6B4lysi-DJJYiFPNb7gTLEm_9nIwrs73QXaycQ7m00)

*이 `Product` 모델에선 다양한 기본 오퍼레이션이 수행될 동안 큰 컬랙션을 여럿 가져오게 된다.*

* 이 큰 클러스터 애그리게잇은 성능이나 확장성이 절대로 좋을 수 없다. 이는 실패로 이어지는 악몽이 될 뿐이다. 거짓 고정자와 컴포지션적 편의성이 설계를 주도했기 때문에 시작부터 문제가 있었으며, 트랜잭션의 종료, 성능, 확장성의 측면에서 안 좋은 영향을 미쳤다.

작은 애그리게잇은? 다른 대상과 일관성을 유지

* 변경이 되면 엔터티
* 대치가 되면 값 객체 : 생각보다 상당히 많은 개념이 값 객체로 대치된다.

> 파생 금융상품 부문에서 약 70% 애그리게잇이 단 하나의 루트 엔터티로 구성된다.

**작은 애그리게잇은**

* 성능이 좋음
* 확장성이 좋음
* 트랜잭션이 성공할 가능성이 크다

### 유스케이스를 전부 믿지는 말라

* 하나의 유스케이스가 여러 트랜잭션을 발생 시킨다면 의심해 보자
    * **이런 경우에서 결과적 일관성을 통해서 문제를 해결할 수 있다.** + 지연 업데이트
* 물론 하나의 유스케이스가 하나의 트랜잭션일 필요는 없다.

## 규칙: ID로 다른 애그리게잇을 참조하라

객체 그래프가 연결되어 있다고 해서 같은 애그리게잇은 아니다. 그저 다른 애그리게잇을 연결했을 뿐이다.

여기도 결국 **결과적 일관성**으로 이어진다.

### 애그리게잇 ID 참조를 통해 서로 함께 동작하도록 해보자

![ID참조모델](http://www.plantuml.com/plantuml/svg/dP11ImCn48Nl-HMXHw757r125VPG42ohU1ztXnYRp9JCH5Z4_su8MwijUl0qoNkF3zxRY4BMag8P8eZOMnZsaVrMCTdr-iRxZ1uKRS-ipjbtOwqeQ97su3mTxuu3QLDBIj1qdGveFcRm8yY-4ZlIeDDC6b6670uQcEhlXKkM7XC42kIhG92mdZUEXHGnVx5scSSow7Qim2U81UtzyoiEwjmSw34Y2FuUU3ZaG7y0Ej6GG0FJ7VkED4zdoNyt-3xmSkdiudgrkbgqUNvwxbJpt9ZhNHh7MgQjVQ9VrZ4RfB6a-0a0)

*ID를 통해 경계 밖과 연결을 추론할 수 있는 `BacklogItem` 애그리게잇*

```java
class BacklogItem {
    private ProductId productId;
}
```

### 모델 탐색

객체 그래프 탐색과는 다르지만 *리파지토르*와 *ID*가 있으면 연관 모델을 탐색할 수 있다. : *단전될 도메인 모델(Disconnected Domain Model)*

```java
@Service
class ProductBacklogItemService {
    @Transactional
    public void assignTeaMemberToTask(
        String aTenantId,
        String aBacklogItemId,
        String aTaskId,
        String aTeamMemberId) {

        BacklogItem backlogItem = 
            backlogItemRepository.findById(
                new TenantId(aTenantId), new BacklogItemId(aBacklogItemId));
        Team ofTeam = 
            teamRepository.findById(
                backlogItem.tenantId(), backlogItem.teamId());
        backlogItem.assignTeamMemberToTask(
            new TeamMemberId(aTeamMemberId), ofTeam, new TaskId(aTaskId));
    }
}
```

### 확장성과 분산

ID 참조를 이용하게 되면 같은 영속화 플랫폼을 사용하지 않고 샤딩과 같은 확장을 통해서 일부 애그리게잇을 손쉽게 확장할 수 있다.
*예를 들면 어떤 애그리게잇은 DB를 사용하고 연관되는 다른 애그리게잇은 NoSql을 사용할 수 있다.*

도메인 이벤트를 통해서 외부 바운디드 컨텍스트로 분산처리를 더 가속화할 수 있다.

역시나 중요한 것은 **결과적 일관성**이다.

## 규칙: 경계의 밖에서 결과적 일관성을 사용하라

결과적 일관성과 지연 시간 = 도메인 이벤트 발행

* 동시성 이슈로 인해서 발행된 이벤트 구독이 실패하면? 메시징 매커니즘을 통해서 Retry! > 이것은 쉽지가 않은 것 같다.

### 누가 해야 하는 일인지 확인하자

> 데이터의 일관성을 보장하는 주체가 유스케이스를 수행하는 사용자의 일인지를 질문해보자.
> 만약 그렇다면, 다른 애그리게잇의 규칙들은 고수하는 가운데 트랜잭션을 통해 일관성을 보장하도록 하자.
> 만약, 다른 사용자나 시스템이 해야 할 일이라면 결과적 일관성을 선택하자.

## 규칙을 어겨야하는 이유

### 첫 번째 이유: 사용자 인터페이스의 편의

### 두 번째 이유: 기술적 매커니즘의 부족

### 세 번째 이유: 글로벌 트랜잭션

개인적으로 안티패턴. 결과적 일관성을 사용하자

### 네 번째 이유: 쿼리 성능

캐싱을 통해서 어느정도 해결할 수 있다.

## 발견을 통해 통찰 얻기

Skip



## 구현

### 고유 ID와 루트 엔터리를 생성하라

### 값 객체 파트를 선호하라

### '데메테르 법칙'과 '묻지 말고 시켜라'를 사용하기

* 데메테르 법칙 : 정보은닉
* 묻지 말고 시켜라 : 정보은닉 + 응집력

### 낙관적 동시성

애그리게잇 루트에 버전을 통한 낙관적 락 기법

`@Version` : JPA를 이용하면 이 선언만으로도 낙관적 락 기법을 사용할 수 있다. 

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private Long id;
    @Version
    private int version;
    private String description;
}
```

### 의존성 주입을 피하라

애그리게잇에 서비스나 리파지토리를 주입하지 마라

## 마무리

* 가능하면 작은 애그리게잇으로 설계하자
* 일관성 경계, 트랜잭션, 고정자가 중요
* 객체 그래프 참조 VS ID 참조
* 경계 외부에서는 **결과적 일관성**
* 응집력있는 구현 : 데메테르 법칙, 묻지 말고 시켜라(command over query??)