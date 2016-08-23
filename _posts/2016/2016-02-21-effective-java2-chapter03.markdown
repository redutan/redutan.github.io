---
layout: post
title: Effective Java 2/E - Chatper 03 모든 객체의 공통 메서드
date: '2016-02-21 22:06'
tags:
  - book
  - effective-java2
---

# Rule 08 - `equals`를 재정의할 때는 일반규약을 따르라

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

- **반사성(reflexive)** : `x.equals(x) = true`
- **대칭성(symmetric)** : `x.equals(y) = true -> y.equals(x) = true`
- **추이성(transitive)** : `x.equals(y) = true && y.equals(z) = true` -> `x.equals(z) = true`
- **일관성(consistent)** : `x.equals(y)` 을 여러 번 해도 -> all true
- **Null에 대한 비동치성(Non-nullity)** : `x.equals(null) = false`

하지만 애석하게도 **객체 생성 가능(instantiable) 클래스를 상속하여 새로운 속성을 추가하면서 `equals` 규약을 어기지 않을 방법은 없다.** *상속을 이용할 경우 동치관계는 깨짐*
그 외에도 여러가지 우회법이 있지만 다 문제가 생긴다 (ex: `instanceof` 대신 `getClass`를 이용하는 경우)

**일반적으로 대칭성(symmetric)을 만족시키지 못한 문제가 생긴다.** : `자식.equals(부모) = true` 이나 `부모.equals(자식) = false` 가 나올 확률이 큼

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

# Rule 09 - `equals`를 재정의할 때는 반드시 `hashCode`도 재정의하라

### `hashCode` 일반규약

- `hashCode`를 여러 번 호출해도 값은 동일하다. **멱등성**
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

# Rule 10 - `toString`은 항상 재정의하라.

- **`toString`을 잘만들어 놓으면 클래스를 좀 더 쾌적하게 사용할 수 있다.**
- **가능하다면 `toString` 메서드는 객체 내의 중요정보를 전부 담아 반환해야한다.**
- **`toString`이 반환하는 문자열의 형식을 명시하건 그렇지 않건 간에, 어떤 의도인지 문서에 분명하게 남겨야한다.**
- **`toString`이 반환하는 문자열에 포함되는 정보들은 전부 프로그래밍을 통해서 가져올 수 있도록(programmatic access)하라.**
    - `toString`을 파싱하지 않고 각 속성을 조회할 수 있는 getter를 제공하라는 것이다.
    - 역으로 말하면 `toString`을 통해서 반환된 정보를 파싱해서 사용하는 것은 매우 위험한 행위이다. `toString` 문자열 형식은 언제든 변할 수 있기 때문이다.

**`toString`를 재정의 하는 가장 큰 이유는 객체를 문자열로 모두 표현해서 한 번에 확인할 수 있다는 것이다** - *Debugging에 큰 도움이 된다.*

기본적으로 toString을 재정의하는 것이 귀찮다면 Lombok([https://projectlombok.org/features/ToString.html](https://projectlombok.org/features/ToString.html))을 사용하는 것을 추천한다.

# Rule 11 - `clone`을 재정의할 때는 신중하라

`Cloneable` `clone`를 허용(구현된)한다는 사실을 알리려고 고안된 믹스인(mixin) 인터페이스이다. - *어떤 선택적 기능을 제공한다는 사실을 선언하는 인터페이스(여기서는 복제)*
위에 설명했다시피 특이하게도 구현할 메소드는 존재하지 않는다. 그저 **`protected`로 선언된 `Object`의 `clone` 메서드가 어떻게 동작할지 정한다.**

> `Cloneable`의 경우 인터페이스를 굉장히 괴상하게 이용한 사례로, 따라하면 곤란하다.

### `clone`(복사) 메서드 일반규약

- `x.clone() != x`
- `x.clone().getClass() == x.getClass()`
- `x.clone.equals(x)`
- 어떤 생성자도 호출하지 않는다.
- **`super.clone()`를 반드시 호출한다.**

**`Cloneable`인터페이스의 책임 : `public clone` 메서드를 제공한다.**

{% highlight java %}
@Override
// Object 대신 PhoneNumber을 써도 된다 - 아래 공변반환형 참고
public PhoneNumber clone() {
    try {
        // super.clone() 라는 라이브러가 제공하는 기능을 적극 사용한다.
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); //수행될 리 없음.
    }
}
{% endhighlight %}

> 공변반환형(convariant return type) : 재정의 메서드의 반환값 자료형은 재정의 되는 메서드의 반환값 자료형의 하위클래스가 될 수 있다.

### `clone`의 shallow copy와 deep copy

> 객체의 복사 [https://en.wikipedia.org/wiki/Object_copying](https://en.wikipedia.org/wiki/Object_copying)

**실질적으로 `clone`는 deep copy를 지원해야한다.**
shallow copy(단순하게 `super.clone()`)가 될 경우 일부 상태(속성)를 공유하게 되므로 문제가 생긴다.
*A라는 객체를 복사해서 B라는 객체를 만들었는데 A의 특정속성 (call by reference 기반 속성)을 수정하면 B도 수정되는 이슈*

**즉, 복사본의 불변식(invariant)의 깨진다.**

복잡한 `clone`를 재정의 하는 것 보다는 **복사 생성자(copy constructor)나 복사 팩터리(copy factory)를 제공하는 것** 이 더 낫다.
- 복사 생성자 : `public Yum(Yum yum);`
- 복사 팩터리 : `public static Yum newInstance(Yum yum);`

> 또한 불변객체의 경우 실질적으로 복제를 허용하는 것 논리적으로 오류가 생긴다 : 복사본과 원본을 논리적으로 구별할 수 없음. - 불변객체는 일반적으로 값객체들인데 어짜피 논리적으로 같아도 괜찮지 않는가? Deep Copy면 괜찮을 거 같음.

### 복사 생성자, 복사 팩터리의 장점

1. `clone`를 재정의 하지 않아도 됨
2. `final`필드(불변객체)와 충돌없음
3. 인터페이스 활용가능 : `TreeSet.clone()` VS `new TreeSet(Set set)`

## 결론

** `Cloneable`을 계승하는 인터페이스는 만들지 않는다.
** 계승을 목적으로 설계하는 클래스는 Cloneble([규칙 17](#))을 구현하지 말아야한다.
** 계승 목적으로 클래스를 설계할 때는 올바르게 동작하는 `protected clone` 메서드를 제공하지 않으면 하위 클래스에서 `Cloneable`을 구현할 수 없다.
** 즉, clone를 쓰는 경우는 배열을 복사할 때 빼고 사용할 일이 거의 없을 것이다. - **너무나도 단점이 많다.**

# Rule 12 - `Compareable` 구현을 고려하라.

{% highlight java %}
public interface Comparable<T> {
    /*
    `this`가 인자(t)보다 작으면 음수
    `this`가 인자(t)와 같으면 0
    `this`가 인자(t)보다 크면 양수
    */
    int compareTo(T t);
}
{% endhighlight %}

**`Comparable` 인터페이스**
- 말 그대로 비교를 통한 객체간 검색, 정렬, 최대/최소 계산
- `Object` 메서드에 대한 재정의가 아니지만 그래도 중요한 이야기다.
- 사용예시
    - `TreeSet`, `TreeMap` - `final int compare(Object k1, Object k2)`
    - `Arrays`, `Collections` - `public static <T extends Comparable<? super T>> void sort(List<T> list)`

**알파벳 순서나 값의 크기, 또는 시간적 선후관계처럼 명확한 자연적 순서를 따르는 값 클래스를 구현할 때는 `Comparable` 인터페이스를 구현한 것을 반디시 고려해 봐야 한다.**

### `compareTo` 구현 일반규약
- **반사성**
- **대칭성** : 모든 x와 y에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`
    - 만약 `sgn(x.compareTo(y))`이 예외를 발생시킨다면 `sgn(y.compareTo(x))`도 예외가 발생해야한다.
- **추이성** : `(x.compareTo(y)) > 0 && y.compareTo(z) > 0)` -> `x.compareTo(z) > 0`
- `x.compareTo(y) == 0`이면 `sgn(x.xompareTo(z)) == sgn(y.compareTo(z))`
- **동치성(강력한권고)** : `(x.compareTo(y) == 0) == (x.equals(y))`
    - *만약 위 조건이 만족하지 않을 경우 반드시 javadoc에 명시해야한다.*

### `compareTo`의 문제점
- `equals` 재정의와 마찬가지로 계승을 이용하면 이슈가 발생한다. 고로 구성을 사용해서 해결하는 것을 추천

### Example code
{% highlight java %}
public int compareTo(PhoneNumber pn) {
    // 지역번호비교
    if (areaCode < pn.areaCode)
        return -1;
    if (areaCode > pn.areaCode)
        return 1;
    // 지역번호가 같으니 국번비교
    if (prefix < pn.prefix)
        return -1;
    if (prefix > pn.prefix)
        return 1;
    // 지역번호와 국번이 같으므로 회선번호 비교
    if (lineNumber < pn.lineNumber)
        return -1;
    if (lineNumber > pn.lineNumber)
        return 1;

    return 0;   // 모든 필드가 일치
}
{% endhighlight %}
