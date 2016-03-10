---
layout: post
title: Effective Java 2/E - Chatper 04 클래스와 인터페이스 (2)
date: '2016-02-28 14:18'
tags:
  - book
  - effective-java2
---

# Rule 18 - 추상 클래스 대신 인터페이스를 사용하라.

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

# Rule 19 - 인터페이스는 자료형을 정의할 때만 사용하라

**인터페이스는 타입이다.**

### 인터페이스 안티패턴

*상수 인터페이스* : 인터페이스 내 상수만 정의된 것

- 인터페이스는 행위에 대한 표현을 나타낼 뿐(그런 타입)이지 상수를 표현하기에는 적합하지 않다.
- 개선안 : 상수 유틸리티 클래스 : *하지만 이것도 적절한 방법은 아니다. 개인적으로 `enum`을 추천한다.*

# Rule 20 - 태그 달린 클래스 대신 클래스 계층을 활용하라.

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
{% endhighlight %}

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
{% endhighlight %}

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

# Rule 21 - 전략을 표현하고 싶을 때는 함수 객체를 사용하라.

**함수객체**

- 함수 성격의 하나뿐인 메서드를 가진 객체
- 대부분 상태가 없음 (stateless)

*example*
{% highlight java %}
class StringLengthComparator {
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}
{% endhighlight %}

*example : 함수 객체 전략 인터페이스*
{% highlight java %}
// 전략 인터페이스
public interface Comparator<T> {
    public int compare(T t1, T t2);
}

// 구상클래스
class StringLengthComparator implements Comparator<String> {
    ...
}
{% endhighlight %}

만약 전략 인터페이스를 익명 클래스로 사용하면 의미없는 객체가 반복될 수 있으므로 **정적인 필드(`private static final`)를 고려해보자**

*example : 익명 클래스*
{% highlight java %}
Array.sort(stringArray, new Comparator<String>() {
   public int compare(String s1, String s2) {
       return s1.length() - s2.length();
   }
});
{% endhighlight %}

*example : 전략 정적 클래스*
{% highlight java %}
// 실행 가능한 전략들을 외부에 공개하는 클래스
class Host {
    private static class StrLenCmp implements Comparator<String>, Serializable {
        public int compare(String s1, String s2) {
            return s1.length() - s2.length();
        }
    }

    // 이 비교자는 직렬화가 가능
    public static final Comparator<String> STRING_LENGTH_COMPARATOR =
            new StrLenCmp();
    // 다른 전략들...
}
{% endhighlight %}

## 결론

- **함수 객체의 주된 용도는 전략 패턴(Strategy pattern)을 구현하는 것**
- **전략을 표현하는 인터페이스를 선언하고, 실행 가능 전략 클래스를 구현한다.**
- **만약 전략 클래스가 반복적으로 사용된다면 `private static` 맴버 클래스로 전략을 표현한 다음, `public static final` 필드를 통해서 외부에 공개하는 것이 바람직하다.**
    - *그냥 바로 `public static final` 필드를 통해서 구현 + 외부 공개까지 한 번에 하는 것이 더 낫지 않은가?*

# Rule 22 - 맴버 클래스는 가능하면 `static`으로 선언하라.

중첩 클래스(nested class)의 4가지 종류

### 정적 맴버 클래스 (static member class)

**클래스 안에 선언된 일반 클래스**

*example* : `Calculator.Operation`

### 비-정적 맴버 클래스 (nonstatic member class)

**outer class의 객체를 참조할 수 있음 - *[1]this한정구문(qualified this)구문*을 통해서**

*example*
{% highlight java %}
// 비-정적 맴버 클래스의 전형적인 용례
public class MySet<E> extends AbstractSet<E> {
    // ...
    public Interator<E> iterator() {
        return new MyItefator();
    }

    private class MyIterator implements iterator<E> {
        // ...
    }
}
{% endhighlight %}

*exampe* : `Map.keySet`, `Map.entrySet`

**바깥 클래스 객체에 접근할 필요가 없으면 정적 맴버 클래스로 만들자**

비정적 맴버 클래스가 되면 해당 객체는 내부적으로 바깥 객체에 대한 참조를 유지하게 되므로, 각종 시간 공간 요구량이 늘어나게 되면 GC가 힘들어진다.

### 익명 클래스 (anonymous class)

- 이름이 없다
- 바깥 클래스의 맴버도 아니다.
- 사용하는 순간에 선언하고 객체를 만든다.
- 코드 어디에서나 사용할 수 있다.
- 비-정적(nonstatic context)안에서 사용될 때는 바깥 객체를 갖는다.
- 정적 문맥(static context) 안에서 사용된다 해도 static 맴버를 가진 수는 없다.
- 여러 인터페이스를 구현하는 익명 클래스는 선언할 수 없다.
- **표현식 중간에 등장하므로, 10줄 이하로 짧게 작성되어야 가독성이 좋아진다.**
- **함수 객체를 정의할 때 특히 많이 쓰인다.**

### 지역 클래스 (inner class)

- 지역 변수로 클래스를 선언한 것이다.

*[1]this한정구문(qualified this)*

this한정구문은 바깥 객체를 참조하기 위해서 this 앞에 바깥 객체의 자료형의 이름을 붙이는 것을 말한다.
{% highlight java %}
class Envelope {
    void x() {
        System.out.println("Hello");
    }
    class Enclosure {
        void x() {
            Envelope.this.x();  // 한정됨
        }
    }
}
{% endhighlight %}

## 결론

1. 맴버클래스 : 중첩 클래스를 메서드 밖에서 사용할 수 있어야 하거나, 메서드 안에 놓기에는 너무 길 경우
    2. 비-정적 맴버 클래스 : 바깥 객체에 대한 참조를 가져야하는 경우
    3. 정적 맴버 클래스 : 바깥 객체에 대한 참조가 필요없는 경우
4. 익명 클래스 : 중첩 클래스가 특정한 메서드에 속해야 하고, 오직 한 곳에서만 객체를 생성하며, 해당 중첩 클래스의 특성을 규정하는 자료형이 이미 있다면(인터페이스, 추상클래스) 사용
5. 지역 클래스 : 익명 클래스를 이용하지 않을 경우
