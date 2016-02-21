---
layout: "post"
title: "Effective Java 2/E - Chatper 03 모든 객체의 공통 메서드"
date: "2016-02-21 23:23"
tags:
  - book
  - effective-java2
---

# Role 13 - 클래스와 맴버의 접근 권한은 최소화하라

중요한 것은 외부로 제공되는 인터페이스이지, 내부 구현이 아니다.
- 정보은닉
- 캡슐화

**정보은닉**
- 의존성을 낮춘다.
- 재사용성을 높인다.
- 개발과정의 위험성을 낮춘다.
- **단순함(복잡하지 않음)**

> 하지만 성능은 보장하지 않는다.

정보은닉을 위한 대원칙 : **각 클래스와 맴버는 가능한 접근불가능하도록 만든다**
- 소프트웨어의 정상적인 동작을 보증하는 한도 내에서 가장 낮은 접근 권한을 설정

Java의 접근제어 매커니즘 : 권한수정자(접근자)
- public : 제한없음
- protected : 같은 패키지와 하위클래스
- package(default) : 같은 패키지
- private : 클래스내부

## 유의사항

**protected의 경우 public과 마찬가지로 외부 공개이므로 수정에 비용이 매우 크다.**
고로 protected도 가능하면 사용을 자제해야한다. private와 package는 외부 노출이 안된다.

** 객체 필드(instance field)는 절대로 public 으로 선언하면 안된다.**
- 불변식 강제가 안됨
- ThreadSafe하지 않음

#### `public static final 배열필드`를 두거나, 배열 필드를 반환하는 접근자(accesor)를 정의하면 안된다.

- 클라이언트가 배열 내용을 변경할 수 있어서 캡슐화가 깨진다.

*example*
{% highlight java %}
// 보안 문제를 초래할 수 있는 코드
public static final Thing[] VALUES = { ... };
{% endhighlight %}

**솔루션**

1. 변경불가 객체로 반환
{% highlight java %}
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Array.asList(PRIVATE_VALUES));
{% endhighlight %}
2. 방어복사 반환
{% highlight java %}
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
{% endhighlight %}

## 결론
- **접근 권한은 가능한 낮추라.**
- **최소한의 public API를 설계한다.**
  - 다른 모든 클래스, 인터페이스, 맴버는 API에서 제외하라.
- **`public static final` 필드를 제외한 어떤 필드도 public으로 선언하지 마라.**
- **`public static final` 필드가 참조하는 객체는 불변객체로 만들라.**

# Role 14 - public 클래스 안에는 public 필드를 두지 말고 접근자메서드(Setter)를 사용하라.

{% highlight java %}
// 이런 저급한 클래스는 절대로 public으로 선언하지 말 것
class Point {
    public double x;
    public double y;
}
{% endhighlight %}

### 문제점
- 캡슐화의 이점을 누릴 수가 없음
- API에 필드가 포함되어 있어서 수정 비용이 매우 큼
- 불변식을 강제할 수 없음 - `private final`

### 솔루션
- getter와 setter(가변객체라면)로 제공한다.
- 기본적으로 필드는 캡슐화 하자.

{% highlight java %}
// getter과 setter를 이용한 데이터 캡슐화
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
{% endhighlight %}

**public, protected로 선언된 클래스에는 getter를 제공하라.**

**하지만, package 클래스, private 중첩 클래스는 데이터 필드를 public으로 제공할 수 있다.**
- 어짜피 클래스 자체가 외부로 공개되어 있지 않기 때문에 클래스를 기반으로 충분히 캡슐화의 이점을 누릴 수 있다.

## 결론
**가능하면 public 클래스의 필드는 외부로 공개하지 않는다.**
**package 클래스, private 중첩 클래스의 필드는 외부로 공개하는 것이 나을 수도 있다.**

# Role 15 - 변경가능성을 최소화하라
