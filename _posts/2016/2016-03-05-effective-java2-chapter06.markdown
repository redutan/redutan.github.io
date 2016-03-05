---
layout: post
title: Effective Java 2/E - Chatper 06 Enum과 Annotation (1)
date: '2016-03-05 13:41'
tags:
  - book
  - effective-java2
---

# Role 30 - int 상수 대신 enum을 사용하라

*example*
{% highlight java %}
public enum Apple { FUGI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
{% endhighlight %}

enum 자료형은 컴파일 시점 형 안정성(compile-time type safety)을 제공한다.

> 상수를 추가하거나 순서를 변경해도 클라이언트는 다시 컴파일할 필요가 없다. 상수를 제공하는 필드가 enum
자료형과 클라이언트 사이에서 격리 계층(layer of insulation) 구실을 하기 때문이다. int enum
패턴과는 달리, 상수의 값이 클라이언트 코드와 함께 컴파일되는 일은 생기지 않는다는 것이다.

*reference를 기반으로 해서 분리가 된다고 이해했다.*

### 풍부한 기능을 가진 enum

- 임의의 메서드나 필드 추가 가능
- 인터페이스 구현 가능
- 비교(Comparable), 직렬화(Serializable), Object 메서드 재정의 가능

**즉 일반객체처럼 상태와 행위를 가질 수 있으며 이것을 풍부하게 표현할 수 있다. 거기에다가 불변성을 유지한다.**

### 행위 다형성을 가진 enum

**행위의 다형성 기반 구현이 가능하다.**

*example*
{% highlight java %}
public enum Operation {
    PLUS   {double apply(double x, double y) {return x + y;}},
    MINUS  {double apply(double x, double y) {return x - y;}},
    TIMES  {double apply(double x, double y) {return x * y;}},
    DIVIDE {double apply(double x, double y) {return x / y;}};

    abstract double apply(double x, double y);
}
{% endhighlight %}

### 정책 enum 패턴

**중복되는 행위(정책)이 필요한 경우 아래처럼 하위 정책 enum에 위임해서 다형성과 재사용성을 확보할 수 있다.**

*example 주중, 주말 수당 계산*
{% highlight java %}
enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    ...
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;
    PayroleDay(PayType payType) { this.payType = payType; }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }
    // 정책 enum 자료형
    private enum PayType {
        WEEKDAY {
            double overtimePay(double hours, double payRate) {
                return hours <= HOURS_PER_SHIFT ? 0 :
                    (hours - HOURS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            double overtimePay(double hours, double payRate) {
                return hours * payRate / 2;
            }
        };
        private static final int HOURS_PER_SHIFT = 8;

        abstract double overtimePay(double hrs, double payRate);

        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
{% endhighlight %}

## 결론

- **int상수 같은 것을 자제하고 enum을 사용하라.**
    - 가동성, 안정, 강력
- **속성과 행위를 추가해서 더 풍부하게 할 수 있다.**
- **분기(if, switch) 대신 상수 별 메서드를 구현하라.**
- **상수 별로 공통 기능이 요구되면 정책 enum 패턴 사용을 고려하라.**

# Role 31 - `ordinal` 대신 객체 필드를 사용하라.

**enum 상수의 순서(ordinal)가 유지보수 하면서 변경될 수 있으므로 안전하지 않다.**
`oridinal()`는 거의 사용할 일이 없다.

> 대부분의 프로그래머는 이 메서드를 사용할 일이 없을 것이다. `EnumSet`이나 `EnumMap`처럼
> 일반적인 용도의 enum 기반 자료구조에서 사용할 목적으로 설계한 메서드다.

**enum 상수에 연계되는 값은 객체 필드에 저장하자.**

# Role 32 - 비트 필드(bit field) 대신 `EnumSet`을 사용하라

*example 비트필드*
{% highlight java %}
// 비트 필드 열거형 상수 - 안티패턴
public class Text {
    public static final int STYLE_BOLD = 1 << 0;            // 1
    public static final int STYLE_ITALIC = 1 << 1;          // 2
    public static final int STYLE_UNDERLINE = 1 << 2;       // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3;   // 8

    // 이 메서드의 인자는 STYLE_상수를 비트별(bitwise) OR 한 값이거나 0.
    public void applyStyles(int styles) { ... }
}
{% endhighlight %}

*example client*

`text.applyStyles(STYLE_BOLD | STYLE_ITALIC);`

하지만 위 비트필드는 int enum 패턴과 똑같은 단점을 가지고 있다.

- 가독성이 떨어지고, 이해하기 힘듬

### 솔루션 `EnumSet`

**enum 자료형의 값으로 구성된 집합을 효율적으로 표현** : `Enum` + `Set`

*example `EnumSet`*
{% highlight java %}
// EnumSet - 비트필드를 대신할 현대적 기술
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // 어떤 Set객체도 인자로 전달(유연성 확보)할 수 있으나, EnumSet이 분명 최선
    public void applyStyles(Set<Style> sytles) { ... }
}
{% endhighlight %}

*example client*

`text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));`

## 결론

**열거형 자료 집합을 사용할 시 `EnumSet`를 이용하자.**

*단점은 불변타입이 아니라는 것인데 `Collections.unmodifiableSet`로 포장하거나 Guava와 같은
라이브러리를 이용하면 된다.*

# Role 33 - `ordinal`을 배열 첨자로 사용하는 대신 `EnumMap`을 이용하라

ordinal 값을 배열 첨자로 사용하는 것은 적절하지 않다. 값이 변할 수 있을 뿐만 아니라, 타입 안정성도 보장 받지 못한다.
**대신 `EnumMap`를 써라**

만약 구현해야하는 것이 다차원이라면 `EnumMap<?, EnumMap<?>>`과 같이 표현하면된다.

# Role 34 - 확장 가능한 enum을 만들어야 한다면 인터페이스를 이용하라

enum 자료형은 상속하기에는 적합하지 않을 뿐만 아니라, 계승도 불가능하다.

**하지만 인터페이스 구현을 통한 확장은 가능하다.**

하지만 상속을 통한 코드 공유가 불가능하므로 코드 중복이 발생할 수 있다.
**이런 경우에는 `helper class`나 `static helper method`를 통해서 중복을 제거할 수 있다.**

### 클라이언트

**한정적 자료형 토큰**
{% highlight java %}
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(
        Class<T> opSet, double x, double y) {
    for (Operation op : opSet.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, op.apply(x, y));
    }
}
{% endhighlight %}

**한정적 와일드카드 자료형**
{% highlight java %}
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(
        Collection< ? extends Operation> ops, double x, double y) {
    for (Operation op : ops) {
        System.out.printf("%f %s %f = %f%n", x, op, op.apply(x, y));
    }
}
{% endhighlight %}

1. 이렇게 하면 `EnumSet`나 `EnumMap`을 사용할 수 없기 때문에 여러 자료형을 정의하는 유연성이 확보가 안됨
2. 위와 같은 유연성이 필요없다면 **한정적 와일드카드 자료형** 방식이 더 나음

## 결론

**계승 가능 enum 자료형은 만들 수 없지만, 인터페이스를 만들고 그 인터페이스를 구현하는 기본 자료형을 만들면
계승 가능 enum 자료형을 흉내 낼 수 있다.**
