---
layout: "post"
title: "클린소프트웨어 Part 5. 기상 관측기 사례 연구"
date: "2017-07-12 14:22"
tags:
  - books
  - clean-software
  - composite
  - observer
---

# 컴포지트 패턴

부분과 전체의 계층을 표현하기 위해 객체들을 모아 트리 구조로 구성. 사용자로 하여금 **개별 객체와 복합 객체를 동일하게 다룰 수 있도록** 하는 패턴

## Class Diagram

![컴포지트 패턴 클래스 다이어그램](https://dooray.com/plantuml/img/oymhIIrAIqnELGZEI2n8LQZcKW02xPIYn78DJQvQhkISnE9Y1UVCekISL8NCt8ASrDpKl99Yk6gOYk32qiGYl2gSytCByeipIr8X4bXKWcrEJ4dH00jeeha4JRzkKMPwHeckdOAIWPwUbXB442u0)

* `Shape`는 `Component` 이다.
* `Circle`, `Square`의 경우에는 더 이상 복합이 불가능하므로 `Leaf` 이다.
* `CompositeShape`가 `Composite`(복합체) 이다. - **위임자**

### 참고

* [https://en.wikipedia.org/wiki/Composite_pattern](https://en.wikipedia.org/wiki/Composite_pattern)

## Source

{% highlight java %}
interface Shape {
    void draw();
}

class CompositeShapte implements Shape {
    private List<Shape> shapes = new ArrayList<>();
    public void (Shape s) {
        shapes.add(s);
    }
    public void draw() {
        shapes.forEach(Shape::draw);
    }
}
{% endhighlight %}

## 결론

* 1:N 관계를 1:1 관계로 단순화 시킨다.
* **다수성**을 가지는가? (1:N 관계를 1:1관계로 변경할 수 있는가?)

# 옵저버 패턴

## 그 전에

### Worst Practice

1. 패턴을 먼저 생각하고 그것을 해결책으로 사용한다.
2. 패턴은 구조적인 문제해결 방법인데, 그것을 문제에 대입한다?? : **주객전도**

### Best Practice

1. 문제에 맞는 적절하고 단순한 리팩토링
2. 그러다 보면 특정 패턴과 유사해진다.
3. 그러면 본격적으로 해당 패턴으로 리팩토링 한다.

## 옵저버 패턴이란?

객체 사이에 **1:N 관계**를 정의하고 어떤 객체의 상태가 변할 때 그 객체에 **묵시적인 의존성(추상적인 결합도)**을 가진 다른 객체들이 그 **변화를 통지 받고 자동으로 갱신**될 수 있게 만듦

* Publish-Subscribe

![옵저버 패턴 클래스 다이어그램](https://dooray.com/plantuml/img/ROyn3i8m34Ntd2BAr0wz0MBX00mzWPEuK8GI5Bi8X7ftggPk8U3jVzdF_uCbJk1OkoiGwNQm5vpKI-bfW1dSkJfQmdJ7LC-cnpzcYDntpwfMeygWGmmk8QC0yS4OVFc0iceP66VZX5bK6KkR75KV65C73hNyNYZ3pSiEVDUH5Ek1n2W8SP5Ra0-cH3QoHKOt-nMXoggxHfkbcFb9eDu0)

*옵저버 패턴 클래스다이어그램*

![옵저버 패턴 시퀀스 다이어그램](https://dooray.com/plantuml/img/IomjoSyhpKrABU9ATCxFIovABKaDBatAIaqkKR3HLO2B-ISLfnQLfHOfM2aKfvO4boIMf6feSYKcbsIM0PcOwh18GOtbIad5fmsV8s1YLWfv-IMPQPLONG2p1aENhXrMxvGMf13s3951auu86j9y2N8Rq4yPgKN4kH2xMkpkn6ak3jVYC1kRX_15AKmE0000)

*옵저버 패턴 시퀀스다이어그램*

1. 주체(`subject`)의 상태 변경이 발생 : `subject.setState()`
    2. 여기서는 감시자(`observer`)가 호출했지만, 다른 객체가 주체의 상태 변경을 호출할 수 있음
2. 주체는 변경을 통보함 : `this.notify()`
3. (감시자들을 순회하면서) 감시자에게 변경을 통보 : `observer.update()`
4. 감시자는 주체에게서 변경된 상태를 얻어옴 : `subject.getState()`
    5. 그리고 주체의 상태를 감시자 자신의 상태와 일치시킴 : `observerState = subject.getState();

## 장점

* 객체 간 협력을 위해서 상호 양방향 연관관계가 필요한 상황(주로 순환참조가 발생할 시)에서 단방향 참조로 변경하면서 결합도를 낮출 수 있음
* 객체 간 일관성을 유지하면서 (동기화가 하지 않으면서) 결합도를 낮출 수 있음

## 대표적 예시

* [MVC](https://ko.wikipedia.org/wiki/모델-뷰-컨트롤러)의 Model과 View

## Pull vs Push

## Pull-model

감시자가 주체에게서 데이터를 끌어오는(pull) 방식

* 주체가 감시자를 전혀 몰라도 됨
* 감시자가 무엇이 변했는지 항상 확인해야 하므로 비효율적일 수 있음

## Push-model

![옵저버 패턴 Push-model](https://dooray.com/plantuml/img/RO-n3i8m34JtV4MKgHtw1mW65ZQ6Ve6Rkb24KXIx5GZnxpHDN4GWsvsSTtVG47kmhEqLY7GzDXUSrLFf-G4ps7DnR0ZzXBvSp1R_c6xWldkivg5tNAgNYj3zuAn7He7ZdT6rUHX5LJCmBiO9eoXY18dS1V8SWBu3Yreo4sQyjU4eRmBVEwNI-bawPRGXoK-hn0zCwM_aYeokjYj2vRitbJPffVbfeDu0)

주체가 감시자에게 자신의 변경에 대한 상세한 정보를 밀어내는(push) 방식

* 어떤 정보가 변하는지 알 수 있음
* 주체가 감시자의 요청이 무엇인지 알아야 함
* Pull 모델에 비해 재사용성이 떨어지며 주체의 어떤 정보가 변하는 것에 대한 의존성이 생김
    * 어떤 정보가 변하는 것 : `subject.notify`, `observer.update` 메소드의 파라미터

## 옵저버와 OOD

* OCP : 주체 변경 없이(closed) 새로운 관찰자 추가(open) 가능
* LSP
    * `Subject`와 `ConcreteSubject`
    * `Observer`와 `ConcreteObserver`
* DIP : `ConcreteSubject` -> `Observer`
    * `ISSUE` 하지만 `ConcreteObserver` 은 `ConcreteSubject`에 의존한다.
* ISP : `ISSUE` 이건 좀 어거지 아닌가?
