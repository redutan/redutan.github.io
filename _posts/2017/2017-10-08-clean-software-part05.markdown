---
layout: post
title: 클린소프트웨어 Part 5. 기상 관측기 사례 연구
date: '2017-10-08 15:59'
tags:
  - books
  - clean-software
---

# 컴포지트 패턴

부분과 전체의 계층을 표현하기 위해 객체들을 모아 트리 구조로 구성. 사용자로 하여금 **개별 객체와 복합 객체를 동일하게 다룰 수 있도록** 하는 패턴

## Class Diagram

![컴포지트 패턴 클래스 다이어그램](/images/2017/10/composite-pattern-class-diagram.png)

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

![옵저버 패턴 클래스 다이어그램](/images/2017/10/observer-pattern-class-diagram.png)

*옵저버 패턴 클래스다이어그램*

![옵저버 패턴 시퀀스 다이어그램](/images/2017/10/observer-pattern-sequence-diagram.png)

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

![옵저버 패턴 Push-model](/images/2017/10/observer-pattern-class-diagram-push-model.png)

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

# 추상서버 패턴

![추상 서버 패턴](/images/2017/10/abstract-server-pattern-class-diagram.png)

`Switchable` 이 추상서버가 된다.

## 인터페이스는 누가 소유하는가?

* 클라이언트가 소유해야한다.
* 인터페이스는 구상체가 아니라 클라이언트를 위한 추상화이다.
* ISP를 상기하자

# 어댑터 패턴

* 만약 Light가 서드파티라면 상속할 수 없다.
* 서드파티 등의 외부 의존을 줄이고 인터페이스 호환을 위해서 어댑터가 필요하다.
    * 우리가 제어할 수 없는 라이브러리를 어댑터로 감싸자

![어댑터 패턴](/images/2017/10/adapter-pattern-class-diagram.png)

## 공짜 점심은 없다.

* `ISSUE` 어댑터 사용 비용이 크다?
    * 과연?
* 추상서버로 대부분 해결가능하면 필요하면 어댑터를 사용하자.

### 클래스 형태의 어댑터

![어댑터 패턴 - 클래스 타입](/images/2017/10/adapter-pattern-class-diagram-class-type.png)

* `LightAdapter`와 `Light`가 상속으로 인해 **강한 결합**이 생긴다.
* 개인적으로 **객체 형태의(합성 기반) 어댑터를** 추천한다.

# 브리지 패턴

![브릿지 패턴](/images/2017/10/bridge-pattern-class-diagram.png)

* 추상부(Abstraction)와 구현부(Implementor) 를 분리
* 관심사 분리가 더 깔끔해 진다.
* 그 만큼 코드량이 많아지고 복잡해진다.

# 프록시 패턴

![프록시 패턴](/images/2017/10/proxy-pattern-class-diagram.png)

*프록시 패턴 클래스 다이어그램*

![프록시 패턴 시퀀스 다이어그램](/images/2017/10/proxy-pattern-sequence-diagram.png)

*프록시 패턴 시쿼스 다이어그램*

## 관계 프록시 적용하기

![ORM proxy](http://storage.googleapis.com/xebia-blog/1/2008/03/image001.gif)

*출처 : [Advanced Hibernate: Proxy Pitfalls](http://blog.xebia.com/advanced-hibernate-proxy-pitfalls)*

> ORM 프레임워크들이 실제로 이러한 방식으로 구현되어 있다.

* 연관관계가 맺어 있는 Entity를 항상 모두 조회할 순 없다.
* 위와 같이 연관관계에 있는 Entity(boss)를 프록시를 통해서 지연로딩할 수 있게 한다.

## 프록시 요약

**프록시의 가장 큰 장점은 도메인과 기술(인프라)사이의 관심사 분리이다.**

# 천국으로의 계단 패턴

* 너무 상속을 기반으로 재사용 하는 것 같다.
* 이런 경우에는 LSP를 위반할 가능성이 크기 때문에 주의를 요한다.
