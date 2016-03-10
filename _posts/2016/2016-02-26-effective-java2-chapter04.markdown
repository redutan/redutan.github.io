---
layout: post
title: Effective Java 2/E - Chatper 04 클래스와 인터페이스 (1)
date: '2016-02-26 01:36'
tags:
  - book
  - effective-java2
---

# Rule 13 - 클래스와 맴버의 접근 권한은 최소화하라

중요한 것은 외부로 제공되는 인터페이스이지, 내부 구현이 아니다.

- 정보은닉
- 캡슐화

**정보은닉**

- 의존성을 낮춘다.
- 재사용성을 높인다.
- 개발과정의 위험성을 낮춘다.
- **단순함(복잡하지 않음)**

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

**객체 필드(instance field)는 절대로 public 으로 선언하면 안된다.**

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

# Rule 14 - public 클래스 안에는 public 필드를 두지 말고 접근자메서드(Setter)를 사용하라.

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

# Rule 15 - 변경가능성을 최소화하라

**불변객체(값객체)를 사용하라.**

### 장점

- 설계하기 쉽다.
- 구현하기 쉽다.
- 사용하기 쉽다.
- 오류가능성이 적다.
- 안전하다. - threadsafe

### 불변객체 5규칙

1. **setter를 제공하지 않는다.**
2. **계승(상속)할 수 없도록 한다.**
    1. `final class`
    2. `private 생성자`
3. **모든 필드를 final로 선언한다.**
4. **모든 필드를 private로 선언한다.**
5. **변경 가능 컴포넌트에 대한 독점적 접근권을 보장한다.**
    6. 생성자, 접근자, readObject(규칙 76)안에서는 방어복사로 설정한다.

*example*
{% highlight java %}
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    // 대응되는 수정자가 없는 접근자들
    public double realPart() { return re; }
    public double imageinaryPart { return im; }

    public Complex add(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex subtract(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    @Overrice
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if(!(o instanceof Complex))
            return false;
         Complex c = (Complex) o;

         return Double.compare(re, c.re) == 0 &&
                Double.compare(im, c.im);
    }

    @Override
    public int hashCode() {
        int result = 17 + hashDouble(re);
        result = 31 * result + hashDouble(im);
        return result;
    }

    private static int hashDouble(double val) {
        long longBits = Double.doubleToLongBits(val);
        return (int) (longBits ^ (longBits >>> 32));
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
{% endhighlight %}

### 장점상세

- **단순하다.**
- **스레드에 안전하다. 동기화도 필요없다.**
- **자유롭게 공유할 수 있다.**
- **내부도 공유할 수 있다.**
    - 객체 내부의 일부 상태값을 공유함
- **다른 객체의 구성요소로 훌륭하다.**

### 단점

- 항상 값마다 별도의 객체를 생성해야한다.
    - 가능한 기본타입(primitive)로 제공한다.
    - 가변 헬퍼 클래스 제공 - ex) `String`(불변)과 `StringBuilder`(가변)
- 방어복사 기법이 요구됨

### 유의사항

불변객체는 *정적 팩터리 생성자* 를 이용하면 좋은 점이 많다

- 상속불가
- 객체 자체 캐싱기능 추가 가능

일부 non-final 객체를 내부에 설정해서 응용해도 된다 (캐시 등)

- 모든 필드가 final 일 필요는 없다.

**직렬화(Serialization) 구현 시 주의해야한다**

- [규칙76에서 자세히 설명](#)

## 결론

- **변경 가능한 클래스로 만들 타당한 이유가 없다면, 반드시 변경 불가능 클래스로 만들어야 한다.**
- **변경 불가능한 클래스로 만들 수 없다면, 변경 가능성을 최대한 제한하라.**
- **특별한 이유가 없다면 모든 필드는 final로 선언하라.**
- **초기화 메서드(재초기화포함)를 제공하지 마라.**

# Rule 16 - 계승(상속)하는 대신 구성하라 (중요)

**계승은 캡슐화 원칙을 위반한다.**

- 상위클래스에 의존적이기 때문에 만약 상위클래스가 변화하면 하위클래스가 망가질 수 있다.
- *상위클래스의 하위클래스가 강결합 상태이고, 즉 상위 클래스가 캡슐화가 되지 않는 다는 것*

### 간단한 해결방법 (구성)

- 계승하지 않고 객체에 private 필드 하나를 두는 것

*example* 포장클래스를 통한 구성
{% highlight java %}
// ForwardingSet은 Set을 포장한 Wrapper class
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}
{% endhighlight %}

**계승은 하위 클래스가 상위 클래스의 하위 자료형(subtype)이 확실한 경우에만 바람직하다**
즉 `IS-A` 관계가 성립할 때만 계승해야한다.

- `IS-A` 의 관계는 클래스와 객체의 관계이고 `IS KIND OF`가 더 적절한 거 같다.

### 계승을 사용한 안좋은 예

- `Properties`와 `HashMap` : `Properties#getProperty`와 `HashMap#get` 이 상호 접근을 허용한다.

## 결론

가능하면 **계승** 을 하지 않고 **구성** 을 사용하자. *포장 클래스 구현에 인터페이스가 있다면 더욱 그렇다.*

# Rule 17 - 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라

### 규칙 세부사항

1. **재정의 가능 메서드를 내부적으로 어떻게 사용하는지(self-use) 반드시 문서에 남기라는 것이다.**
    2. 재정의 가능 메서드 : public, protected의 일반(non final)메서드
    3. 어떤 메서드가 어떤 다른 메서드나 클래스에 의존하고 있고, 어떻게 작동되는지 상세하게 서술한다.
2. **클래스 내부 동작에 개입할 수 있는 훅(hooks)을 신중하게 고른 protected 메서드 형태로 제공해야한다.**
3. **계승을 위해 설계한 클래스를 테스트할 유일한 방법은 하위클래스를 직접 만들어 보는 것이다.**
    4. 적절하게 테스트케이스를 만들어서 다양한 케이스로 테스트를 해보는 것 뿐이다. : 3개는 만들어보자.
5. *생성자* 는 직접적이건 간접적이건 **재정의 가능 메서드를 호출해서는 안된다는 것이다.**

*example*

{% highlight java %}
public class Super {
    // 생성자가 재정의 가능 메서드를 호출하는 잘못된 사례
    public Super() {
        overrideMe();
    }
    public void overrideMe() {
    }
}

public final class Sub extends Super {
    private final Date date; // 생성자가 초기화하는 final필드

    Sub() {
        // super(); 을 자동으로 호출한다.
        date = new Date();
    }

    // 상위 클래스 생성자가 호출하게 되는 재정의 메서드
    @Override
    public void overrideMe() {
        System.out.println(date);
    }

    public static void main(String[] args) {
        Sub su = new Sub();
        sub.overrideMe();
        // 과연 생각한대로 date가 정상으로 출력되는가??
    }
}
{% endhighlight %}


`Cloneable`나 `Serializable`와 같은 인터페이스를 구현한다면 계승을 사용하지 않는 것이 낫다.

- 너무 과도한 책임이 졔약이 생긴다.

## 결론

**계승을 위한 클래스를 설계하면 클래스에 상당한 제약이 가해진다.**
**계승에 맞도록 철저하게 설계하고, 문서화하지 않은 클래스에 대한 하위클래스를 만들지 않는 것이다.**
*final* 로 클래스를 만들던지, public 생성자를 제공하지 않으면 된다.
**계승을 할 수없게 막았다면 구성을 통해 확장하면 된다.**

### 계승을 위한 안전한 구현

*클래스 내부적으로 재정의 가능 메서드를 사용하는 경우(self-use)를 완전히 제거하는 방법*

1. 재정의 가능 메서드의 내부 코드를 private로 선언된 도움 메서드 안으로 옮긴다.
2. 각각의 재정의 가능 메서드가 해당 메서드를 호출하게 한다.
3. 재정의 가능 메서드를 호출하는 내부 코드는 전부 해당 private 도움 메서드 호출로 바꾼다.
