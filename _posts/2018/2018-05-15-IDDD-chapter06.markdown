---
layout: "post"
title: "IDDD 6장. 값 객체"
date: "2018-05-15 01:39"
tags:
    - IDDD
    - DDD
    - BOOK
---

가능한 엔터티 대신 값 객체를 사용해 모델링하도록 노력해야 한다.

> 특성만 신경 쓴다면 이를 값 객체로 분리하라. 특성의 의미를 표현하고 기능도 부여하자. 불변성. 식별자는 필요 없고, 설계의 복잡성을 줄여준다.

## 값의 특징

**개념을 값으로**

* 측정, 수량화, 설명
* 불변성(no side-effect)
* 개념적 전체
* 변경 대신 대치
* 등가성(value equality:동등성)

**유비쿼터스 언어로써 표현하는 값이어야함**

### 측정, 수량화, 설명

날짜와 시간, 나이, 통화, 주소, 도형, ...

### 불변성

* No Setter
    * 하지만 완벽하진 않다. 방어 복사가 필요할 수 있음 (`final`일지라도 불변은 깨질 수 있다.)

### 개념적 전체

* 주소 = 우편번호 + 기본주소 + 상세주소
* 돈 = 500(수량) + 달러(통화)

*Bad Case*

```java
class ThingOfWorth {    // 가치 있는 것
    private String name;
    private BigDecimal amount;
    private String currency;
}
```

*Good Case*

```java
class ThingOfWorth {    // 가치 있는 것
    private ThingName name;         // !!
    private MonetaryValue worth;    // 가치
}
@Value
class MonetraryValue {
    private BigDecimal amount;
    private Currency currency;
}
```

* `ThingName`으로 값 객체로 만들어서 name의 기능을 중앙화 시킬 수 있다. (도메인 로직을 안으로 품을 수 있음)

### 대체성

값은 변경이 불가능하므로 값 참조를 다른 값으로 바꾸는 것을 말함

```java
FullName name = new FullName("Myeongju", "Jung");
// ...
name = new FullName("Jiwon", "M", "Jung");
```

### 값 등가성

`equals(), hasoCode()`

엔터티로 설계된 개념이 식별자가 필요하지 않다면 *값 객체*로 모델링하자.

### 부작용이 없는 행동

> 만약 값객체에 행위가 없다면, 좋은 설계인지 의심하라

* Getter : No side-effect
* Setter : With side-effect

```java
FullName name = new FullName("Myeongju", "Jung");
// ...
name = name.withMiddleInitial("M");
```

```java
FullName withMiddleInitial(String middleName) {
    return new FullName(this.firstName, middleName, this.lastName);
}
```

**값 객체의 매개변수로 값만 전달하자.**

* 만약 엔터티와 같은 가변성을 가지는 인자를 받는다면 *부작용이 발생할 가능성*이 생긴다.
* 아니면 Getter만 제공하는 추상화된 인자를 받는 것도 좋은 방법이다. > Entity에 Getter만 있는 인터페이스를 Mixin

*기본 언어 값 타입에는 도메인에 맞춘 부작용이 없는 함수를 할당할 수 없다*

* 도메인적인 표현이 기본 언어 함수에서 제공할 수 있을리가 없다.

## 미니멀리즘으로 통합하기

![미니멀리즘으로 통합하기](http://www.plantuml.com/plantuml/svg/ROv12i8m44NtEKN8FZSeeQ482eeBqVsO35fC4qac2z7UNSqYjjBTCEy__cyJGQGyE7O7SuCByer5JpqzjBVQ64o9Fnddni7dEYQCl6bM9Q0K6wjbWdDm3X6e3tvYTFKVlkO9N4QbIc2i8PtfEiK_EoBG8ja5Y_6FZQpi4_lrGV16IYvqjnMp2Mo-voLbALy4fNoHr7BMehTvS6y0)

* ACL 내에서는 조회용으로써 일부 특성과 행위가 요구된다.
* 즉 Entity 전체가 필요한(순응자나 공유모델) 것이 아니라, 해당 컨택스트 내에서 어울리는 타입으로써 값 객체가 좋은 선택

## 값으로 표현되는 표준 타입

 표준 타입에는 Enum을 쓰는 것이 좋다

 > 잘 이해가 안된다 ㅠㅠ

## 값 객체의 테스트

* 클라이언트 관점에서 값 객체를 보는 것은 중요하다.
* 불변성을 테스트 해야 한다.
* 테스트를 통해서 도메인적 표현을 확인할 수 있어야 한다.

## 구현

* 불변, 불변, 불변 (No Setter)
* `equals()`, `hashCode()`, `toString()`
* JPA를 위해서 빈 생성자는 필수 (PACKAGE 접근제어로 생성)
* 보호절을 통한 불변식 강제

## 값 객체의 저장

### 데이터 모델 누수의 부정적 영향을 거부하라

도메인 모델을 위해서 데이터 모델을 설계해야한다.

### ORM 구현

오래된 내용이라서 SKIP. 이젠 [Spring Data Jpa](https://projects.spring.io/spring-data-jpa/)를 사용하자.

## 마무리

* 값 객체 특징, 사용, 구현
