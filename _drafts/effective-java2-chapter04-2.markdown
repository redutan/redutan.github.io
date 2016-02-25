---
layout: "post"
title: "Effective Java 2/E - Chatper 04 클래스와 인터페이스 (2)"
date: "2016-02-26 01:35"
tags:
  - book
  - effective-java2
---

# Role 18 - 추상 클래스 대신 인터페이스를 사용하라.

자바에서 지원하는 2가지 일반화 방법

- **인터페이스** : 행위만 존재 (추상)
- **추상클래스** : 일부 구현 가능 (추상 + 구체)

**인터페이스는 믹스인을 정의할 때도 좋다.**
즉, 주 자료형(primary type) 이외에 추가로 구현할 자료형(행위)으로 선택적 기능을 제공한는 사실을 선언할 수 있다.
물론 주 자료형은 클래스(or 추상 클래스)이거나 또 다른 인터페이스 일 것이다.

*example : `Comparable` 비교를 위한 인터페이스*

계층적인 것은 클래스 기반으로 어울리며, 비 계층 적인 것은 인터페이스가 어울리다.
*물론 인터페이스는 계층적인 표현에도 어울린다.*

### 추상 구현체
**추상 골격 구현(abstract skeleta implementation) 클래스를 중요 인터페이스마다 두면, 인터페이스의 장점과 추상 클래스의 장점을 결합할 수 있다.**

- naming rule : *`AbstractInterface`*

*example 추상클래스를 통한 구상클래스*

{% highlight java %}
// 골격 구현 위에서 만들어진 완전한 List 구현
static List<Integer> intArrayAsList(final int [] a) {
    if (a == null) {
        throw new NullPointerException();
    }
    return new AbstractList<Integer>() {
        public Integer get(int i) {
            return a[i];    // 자동객체화 - Auto Boxing (규칙 5)
        }

        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;     // 자동비객체화 - Auto unboxing
            return oldVal;  // 자동객체화
        }

        public int size() {
            return a.length;
        }
    }
}
{% endhighlight %}

int배열을 `Integer`배열로 만드는 Adapter 겸 정적 팩터리 예제

**중요한 것은 골격 구현체가 있으면 인터페이스 전체가 아니라 일부를 구현할 수 있게 도움이 된다.**
*물론 골격 구현도 계승(상속)을 기반으로 하기 때문에 골격구현 관련해서 문서화를 통해서 구현 가이드를 제공해야한다.*

골격 구현체를 뛰어 넘어서 기본 간단 구현체를 제공하는 방법도 있다.

- *example : `AbstractMap.SimpleEntry`*

## 결론

**적절하게 인터페이스와 추상클래스를 일반화의 도구로 사용해야한다. 그리고 각각의 장단점도 이해애햐한다.**

- 인터페이스 : 유연
- 추상클래스 : 확장

추상타입은 한 번 API에 포함되면 수정 비용이 매우 크거가 수정할 수 없으므로 설계에 주의한다. 하지만
**더 유연한 인터페이스를 사용하고 그 인터페이스의 구현 편의를 돕는 추상 골격 클래스나 기본 간단 구현체로 지원하자.**

# Role 19 - 인터페이스는 자료형을 정의할 때만 사용하라

**인터페이스는 타입이다.**

### 인터페이스 안티패턴

*상수 인터페이스* : 인터페이스 내 상수만 정의된 것

- 인터페이스는 행위에 대한 표현을 나타낼 뿐(그런 타입)이지 상수를 표현하기에는 적합하지 않다.
- 개선안 : 상수 유틸리티 클래스 : *하지만 이것도 적절한 방법은 아니다. 개인적으로 `enum`을 추천한다.*

# Role 20 - 태그 달린 클래스 대신 클래스 계층을 활용하라.

**SRP 원칙을 상기하자**

**클래스의 여러개의 역할이나 기능을 담지 말고 하나의 기능을 담도록 하자.**

### 태그 달린 클래스 - 안티패턴

*example*
{% highlight java %}
// 태그 달린 클래스
class Figure {
    enum Shape {RECTANGLE, CIRCLE};

    // 어떤 모양인지 나타내는 태그 필드
    final Shape shape;

    // 태그가 RECTANGLE일 때만 사용되는 필드들
    double length;
    double width;

    // 태그가 CIRCLE일 때만 사용되는 필드들
    double radius;

    // 원을 만드는 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형을 만드는 생성자
    Figure(double length, double width) {
        shape = CIRCLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RETANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError();
        }
    }
}
{% endhighlight $}

**태그 기반 클래스는 너저분한데다 오류 발생가능성이 높고, 효율적이지도 않다. 그거 클래스 계층을 모아서 분기로 처리한 것 뿐이다.**

### 태그 클래스를 클래스 계층 으로 변경 - 추천

*example*
{% highlight java %}
// 태그기반 클래스를 클래스 계층으로 변환한 결과
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    double area() { return length * width; }
}
{% endhighlight $}

**장점**

- 단순하고 명료
- 완벽하게 역할 분리 (SRP 원칙 만족)
- 의미없이 반복적인 상투적인(boilerplate) 코드가 없어짐
- 확장이 용이함

*example 클래스의 다형성 기반 확장*
{% highlight java %}
// 정사각형으로 확장
class Share extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
{% endhighlight %}

## 결론

**태그 기반 클래스 사용은 피해야한다.**

# Role 21 - 전략을 표현하고 싶을 때는 함수 객체를 사용하라.
