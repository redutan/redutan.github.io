---
layout: post
title: 클린소프트웨어 Part 2. 애자일 설계(2)
date: '2017-06-08 12:13'
tags:
  - books
  - clean-software
---

# 리스코프 치환 원칙(LSP)

## 리스코프 치환 원칙(LSP)

> 서브타입(subtype)은 그것의 기반 타입(base type)으로 치환 가능해야 한다.

일반적인 **상속 (의 행위)** 관한 원칙

*LSP 위반은 잠재적은 OCP 위반이다.*

## IS-A

* 고양이는 동물이다.
* Cat is an Animal

### 문제

![영속 집합 계층구조](https://www.plantuml.com/plantuml/img/IqmgBYbAJ2vHICv9B2vM24ujuOAm0bABYZEBIrBpIh29-ITbfIR3X4CoCejI0XABIYfHDQ7m57HB2tHhx1Gm9RHqYpBJCqfqxN0QX4DSGQ-qGCyEqrK0)

*영속 집합 계층구조*

하지만 여기에서 Add 메서드가 특정 타입인 경우 `PersistentObject`에서 파생된 것이 아닌 경우에는 LSP를 위배하게 된다.

### LSP를 따르는 해결책

**LSP가 깨지는 메서드는 서브 클래스로 분리하고 문제 없는 메서드는 기반 클래스로 분리한다.**

![LSP를 따르는 해결책](https://www.plantuml.com/plantuml/img/IqmgBYbAJ2vHICv9B2vMy4tDJKejSixFAqdCp4ijKgZcKW02xQ3KtFooL8qGpJeWW0Xvvddc0GMuQhaIKOq8JYs1QNDCIO4eWSW4f1OLPnQNfEQL4AF6FoahDRa4AXoIaLcK4f1OL5A9OWWNo23TqWBT6ZjqftEXsaQK8YtTeipqZ19TEvpsuH1Nq2ijqBF3T3m0)

*LSP를 따르는 해결책*

## 파생(상속) 대신 공통 인자 추출(인터페이스 분리) 하기

* 최상위에 인터페이스를 하나 선언 (Shape)
  * 클라이언트는 해당 인터페이스에 의존하면 됨
* 인터페이스를 구현하는 추상클래스나 클래스를 생성 (Rectangle)
* 그리고 해당 클래스를 **구성** 해서 인터페이스를 구현하면 됨 (Square)

![더 나은 방법](https://www.plantuml.com/plantuml/img/oymhIIrAIqnELGZEI2n8LQZcKW02xVJK4iUYr4GDJQvQhkISnE9YXQ3Kv9B4lFGSk9BYr9Bmp9II3A0Q6DyZDJCzemH9Kt1XQM8HJXs9nN13mNeWBh3HqqDOXYG6COiBuGuRtPpKj19Tc0G0)

> 상속(LSP)이 제대로 가능한 경우는 템플릿 메소드 패턴을 이용한 경우인 것 같다. 그 외의 경우라면 그냥 구성하는 편이 나은 것 같다 - 필자생각

## 휴리스틱과 규정

**LSP 위반의 단서를 보여주는 휴리스틱**

1. 기반 클래스에서 어떻게든 기능성을 제거한 파생 클래스에 대해 적용해야 한다.
2. 기반 클래스보다 덜한 동작을 하는 파생 클래스는 보통 그 기반 클래스와 치환이 불가능하므로 LSP 위반

### 파생 클래스에서의 퇴화 함수

{% highlight java %}
class Base {
  public void f() {
    // 구현 코드
    ...
  }
  class Derived extends Base {
    @Override
    public void f() {
      // 퇴화 시킴
    }
  }
}
{% endhighlight %}

### 파생 클래스에서의 예외 발생

기대하지 않은 예외가 발생하면 위반 가능성이 크다

## 결론

기반 타입으로 표현된 모듈을 수정 없이도 확장 가능하게 만드는, **서브 타입의 (특히 행위) 치환 가능성을 말한다.**
