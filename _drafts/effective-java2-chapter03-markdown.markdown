---
layout: "post"
title: "Effective Java 2/E - Chatper 03 모든 객체의 공통 메서드"
date: "2016-02-21 20:15"
tags:
    - book
    - Effective-java
---

# Role 08 - `equals`를 재정의할 때는 일반규약을 따르라

**equals 메서드를 재정의 하지 않아도 되는 경우**

- 각각의 객체가 고유하다. (ex: `Thread`)
- 클래스에 “논리적 동일성(logical equality)” 검사 방법이 있건 없건 상관없다. (ex: 유틸성 객체)
- 상의 클래스에서 재정의한 `equals`가 하위 클래스에서 사용하기에도 적당하다. (ex: `AbstractList` - `List`)
- 클래스가 `private` 또는 default로 선언되었고(외부미노출), `equals` 메서드를 호출할 일이 없다.
    - 가능하다면 이런 경우에는 아래와 같이 방어코드로 재정의한다.
    - `public Boolean equals(Object o) { throw new AssertError(); }`

반대로, 위 조건을 만족하지 못한 경우 equals의 재정의가 요구된다.

- 일반적으로 값 클래스(Value Class)는 대체로 그 조건에 부합한다 (ex: `Integer`, `String` 등)
- *하지만 enum이나 Singleton 기반 객체는 equals가 필요없음*

### equals 메서드 재정의 일반 규약

equals 메서드는 **동치 관계(equivalence relation)** 를 만족해야함

- **반사성(reflexive)** : `x.equals(x)` = true
- **대칭성(symmetric)** : `x.equals(y)` = true -> `y.equals(x)` = true
- **추이성(transitive)** : `x.equals(y)` = true && `y.equals(z)` = true` -> `x.equals(z)` = true
- **일관성(consistent)** : `x.equals(y)` 을 여러 번 해도 -> all true
- **Null에 대한 비동치성(Non-nullity)** : `x.equals(null)` -> false

하지만 애석하게도 **객체 생성 가능(instantiable) 클래스를 상속하여 새로운 속성을 추가하면서 `equals` 규약을 어기지 않을 방법은 없다.** *상속을 이용할 경우 동치관계는 깨짐*
그 외에도 여러가지 우회법이 있지만 다 문제가 생긴다 (ex: `instanceof` 대신 `getClass`를 이용하는 경우)

하지만 위 문제를 해결할 수 있는 방법이 존재함. -> **상속 대신 구성(Composit)하라**

{% highlight java %}
// equals 규약을 위반하지 않으면서 속성 추가
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        if (color == null)
            throw new NullPointerException();
        point = new Point(x, y);
        this.color = color;
    }

    /**
     * ColorPoint의 Point뷰 반환
     */
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
{% endhighlight %}

> 아래 내용 이해 안됨 (p.55)

- abstract class Shape{ }
- class Circle extends Shape { int radius; }
- class Rectangle extends Shape { int length, width; }

**위와 같은 구성에서 왜 Shape 객체의 equals 동치성 이슈가 생기지 않는지 이해가 안됨**

**신뢰성이 보장되지 않는 자원(unreliable resource)들을 비교하는 equals를 구현하는 것은 삼가라**

- ex) `URL#equals`

#### Null에 대한 비동치성(Non-nullity)

`public Boolean equals(Object o) { if (o == null) return false; … }`
*굳이 위와 같이 null 여부를 비교할 필요가 없다. 아래와 같이 instanceof 만으로도 false가 반환된다.*
`public Boolean equals(Object o) { if (!(o instance MyType)) return false; … }`

#### 훌륭한 equals메서드 구현지침

1. **== 연산자를 사용하여 equals 인자가 자기 자신인지 검사하라.**
2. **instanceof 연산자를 사용하여 인자의 자료형이 정확한지 검사하라.**
3. **equals의 인자를 정확한 자료형으로 변환하라. : casting**
4. **“중요” 필드 각각이 인자로 주어진 객체의 해당 필드와 일치하는지 검사하라.**
    - 기본자료형은 == 비교 (`float`, `double`는 제외)
    - 객체(참조)는 `equals` 재귀적 호출로 비교
        - 객체는 null인 경우가 많으므로, 아래 2가지 방식으로 비교 : 아래의 경우는 객체가 같은 경우가 많을 경우 추천
        - `filed == null ? o.field == null : field.equals(o.field)`
        - `field == o.field || (field != null && field.equals(o.field))`
    - `float`는 `Float.compare`, `double`는 `Double.compare`로 비교 : NaN, -0.0 때문에 특별취급
    - 배열은 Arrays.equals 로 비교
    - 추가적으로 필드의 비교 순서도 생각하면 좋다 : 불일치할 확률이 크거나 비교 비용이 낮은 필드부터 먼저 한다.
    - “중요” 필드의 의미는 동치성과 관계가 있는 필드만 비교한다는 것이다.  : 중복필드나 논리적 상태와 관계없는 필드(Lock, `Thread` 등 제어필드)는 제외
5. **equals 메서드 구현을 끝냈다면 대칭성, 추이성, 일관성이 만족되는지 검토하라.**
    - 적절한 단위테스트(unit test)를 이용

#### 주의사항 몇가지 더

- **equals를 구현할 때는 hashCode도 재정의하라** : *다음 장*
- **너무 머리 쓰지 마라**
- **equals 메서드의 인자타입을 `Object`에서 다른 것으로 바꾸지마라** : *`override`가 되지 않고 `overloading` 됨*

# Role 09 - `equals`를 재정의할 때는 반드시 `hashCode`도 재정의하라

### `hashCode` 일반규약

- `hashCode`를 여러 번 호출해도 값은 동일하다.
- **`equals` 가 같다고 판정한 두 객체의 hashCode 값은 같아야한다. : hashCode를 재정의 하지 않을 경우 위반**
- `equals` 가 다르다고 판정한 두 객체의 hashCode 값은 꼭 다를 필요는 없다

만약 필요할 때(Hash함수를 이용하는 경우) `hashCode`를 구현하지 않으면
아래와 같이 Hash함수를 사용하는 `Collection`에서 정상적으로 조회가 안 될 수 있다.

{% highlight java %}
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");

String s = m.get(new PhoneNumber(707, 867, 5309));
System.out.println("s = " + s);     // s = null
{% endhighlight %}

기본적으로 `equals`와 `hashCode`를 재정의하는 것이 귀찮다면 Lombok([https://projectlombok.org/features/EqualsAndHashCode.html](https://projectlombok.org/features/EqualsAndHashCode.html))을 사용하는 것을 추천한다.

### `hashCode`구현 예시

{% highlight java %}
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + areaCode;
    result = 31 * result + prefix;
    result = 31 * result + lineNumber;
    return result;
}
{% endhighlight %}

**추가로 성능을 개선하려고 객체의 중요 부분을 해시 코드 계산 과정에서 생략하면 안 된다는 것이다.**

# Role 10 - `toString`은 항상 재정의하라.

- **`toString`을 잘만들어 놓으면 클래스를 좀 더 쾌적하게 사용할 수 있다.**
- **가능하다면 `toString` 메서드는 객체 내의 중요정보를 전부 담아 반환해야한다.**
- **`toString`이 반환하는 문자열의 형식을 명시하건 그렇지 않건 간에, 어떤 의도인지 문서에 분명하게 남겨야한다.**
- **`toString`이 반환하는 문자열에 포함되는 정보들은 전부 프로그래밍을 통해서 가져올 수 있도록(programmatic access)하라.**
    - `toString`을 파싱하지 않고 각 속성을 조회할 수 있는 getter를 제공하라는 것이다.
    - 역으로 말하면 `toString`을 통해서 반환된 정보를 파싱해서 사용하는 것은 매우 위험한 행위이다. `toString` 문자열 형식은 언제든 변할 수 있기 때문이다.

**`toString`를 재정의 하는 가장 큰 이유는 객체를 문자열로 모두 표현해서 한 번에 확인할 수 있다는 것이다** - *Debugging에 큰 도움이 된다.*

기본적으로 toString을 재정의하는 것이 귀찮다면 Lombok([https://projectlombok.org/features/ToString.html](https://projectlombok.org/features/ToString.html))을 사용하는 것을 추천한다.
