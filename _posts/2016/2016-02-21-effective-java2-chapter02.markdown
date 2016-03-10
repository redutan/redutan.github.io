---
layout: "post"
title: "Effective Java 2/E - Chatper 02 객체의 생성과 삭제"
date: "2016-02-21 17:41"
tags:
    - book
    - effective-java2
---

# Rule 01 - 사용자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해보라

**장점**

- 첫번째 장점은, 생성자와는 달리 정적 팩터리 메서드에는 이름(name)이 있다.
    - 이름을 기반으로 가독성 확보 가능
    - (생성자와는 다르게) 시그네쳐에 의존적이지 않고 다양하게 제공 생성메소드 제공
- 두번째 장점은, 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요는 없다는 것이다.
    - ex) `Boolean.valueOf(boolean)`
- 세번째 장점은, 생성자와는 달리 반환값 타입의 하위 타입 객체를 반환할 수 있다는 것이다.
    - 중요한 건 구현클래스가 아니라 인터페이스만 반환하면 된다.
    - 구상체가 아니라 인터페이스에 의존하기 때문에 더 유연해진다.
- 네번째 장점은, 형인자 자료형(parameterized type) 객체를 만들 때 편하다는 점이다.
    - 하지만 JDK 1.7 이상 부터는 생성자에서도  Parameterized type 유추가 가능해졌다
    - ex) `Map<String, List<String>> map = new HashMap<>();`

**정적 팩터리 메서드만 있는 클래스를 만들면 생기는 단점**

- public이나 protected로 선언된 생성자가 없으므로 하위클래스를 만들 수 없다.
- 정적 팩터리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다.
    - 하지만 개선안이 있다. 정적 메서드의 명칭을 관례에 따른다
        - `valueOf` : 형변환
        - `of` : `valueOf`  줄임
        - `getInstance` : 객체 반환 (같은 객체일 수도 있음)
        - `newInstance` : 항상 다른 객체 반환
        - `getType` : `getInstance`와 같으나, 해당 클래스의 서브타입이 아니라 다른 타입을 반환할 경우 사용한다. 여기에서 Type는 그 다른 타입을 말한다.
        - `newType` : `newInstance`와 같으나, 해당 클래스의 서브타입이 아니라 다른 타입을 반환할 경우 사용한다. 여기에서 Type는 그 다른 타입을 말한다.

{% highlight java %}
/**
 * 서비스 인터페이스
 */
public interface Service {
    // 서비스에 대한 고유한 메서드가 이 자리에 온다
}

/**
 * 서비스 제공자 인터페이스
 */
interface Provider {
    Service newservice();
}

/**
 * 서비스 등록과 접근에 사용되는 객체 생성 불가능 클래스
 */
class Services {

    // 객체 생성 방지 (규칙 4)
    private Services() { }

    // 서비스 이름과 서비스 간 대응관계 보관
    private static final Map<String, Provider> providers = new ConcurrentHashMap<>();

    public static final String DEFAULT_PROVIDER_NAME = "<dev>";

    /**
     * 제공자 등록 API
     * @param p
     */
    public static void registerDefaultProvider(Provider p) {
        registerProvider(DEFAULT_PROVIDER_NAME, p);
    }

    private static void registerProvider(String name, Provider p) {
        providers.put(name, p);
    }

    /**
     * 서비스 접근 API
     * @return
     */
    public static Service newInstance() {
        return newInstance(DEFAULT_PROVIDER_NAME);
    }

    private static Service newInstance(String name) {
        Provider p = providers.get(name);
        if (p == null) {
            throw new IllegalArgumentException(
                    "No provider registered with name : " + name);
        }
        return p.newservice();
    }
}
{% endhighlight %}

> 정적 팩터리 메서드와 public 생성자는 용도가 다르며, 그 차이와 장단점을 이해하는 것이 중요하다. 정적 팩터리 메서드가 효과적인 경우가 많으니, 정적 팩터리 메서드를 고려해 보지도 않고 무조건 public 생성자를 만드는 것은 삼가기 바란다.

# Rule 02 - 생성자 인자가 많을 때는 `Builder` 패턴 적용을 고려하라.

**생성자 인자가 많을 경우** 점층적 생성자 패턴(Telescoping constructor pattern)은 잘 동작하지만 인자 수가 늘어나면 클라이언트 코드를 작성하기가 어려워지고, 무엇보다 읽기 어려운 코드가 되고 만다.

**생성자 인자가 많을 경우**  자바빈 패턴을 사용할 경우 단점

- 1회의 함수 호출로 객체 생성을 끝낼 수 없으므로, 객체 일관성(consistency)이 일시적으로 깨질 수 있다. - 방어할려면 모든 setter에 유효성 체크를 걸어야한다.
- 불변(immutable)객체를 만들 수 없다.
- 스레드 안정성(Thread-safety)을 제공하기 위해 할 일도 더 많아진다.

**결론은 빌더패턴 (Builder pattern)**

`Ada`나 `Python`언어는 선택적 인자에 이름을 붙일 수 있도록 허용하는데, 그거과 비슷한 코드를 작성할 수 있기 때문이다.

`Class.newInstance`는 컴파일 시점에 예외 검사가 가능해야 한다는 규칙을 깨트린다.

- `Class.newInstance`는 항상 무인자 생성자를 호출하는데, 해당 생성자가 존재하지 않기 때문에 문제가 생긴다

{% highlight java %}
/**
 * p.19 규칙 02 빌더패턴 고려하라
 * 빌더패턴!!
 */
public class NutritionFacts3 {
    private final int servingSize;      // (mL)                 *Required
    private final int servings;         // (per container)      *Required
    private final int calories;         //                      Optional
    private final int fat;              // (g)                  Optional
    private final int sodium;           // (mg)                 Optional
    private final int carbohydrate;     // (g)                  Optional

    public static class Builder {

        // 필수인자
        private final int servingSize;
        private final int servings;
        // 선택인자
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val; return this;
        }

        public Builder fat(int val) {
            fat = val; return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val; return this;
        }

        public Builder sodium(int val) {
            sodium = val; return this;
        }

        public NutritionFacts3 build() {
            return new NutritionFacts3(this);
        }
    }

    private NutritionFacts3(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        carbohydrate = builder.carbohydrate;
        sodium = builder.sodium;
    }

    public static void main(String[] args) {
        NutritionFacts3 cocaCola = new Builder(240, 8)
                .calories(100)
                .sodium(35)
                .carbohydrate(27).build();
    }
}
{% endhighlight %}

**빌더 패턴은 인자가 많은 생성자나 정적 팩터리가 필요한 클래스를 설계할 때, 특히 대부분의 인자가 선택적 인자인 상황에 유용하다.**

빌더를 구현하는 것이 귀찮다면 Lombok([https://projectlombok.org/features/Builder.html](https://projectlombok.org/features/Builder.html))을 사용하는 것을 추천한다.


# Rule 03 - `private` 생성자나 `enum` 자료형은 싱글턴 패턴을 따르도록 설계하라

> 해당 규칙은 잘 이해가 안됨.
본인이 이해한 내용은 **싱글턴 패턴을 구현 시에는 private 생성자나 enum자료형으로 설계하라** 로 이해함.

클래스를 싱글턴으로 만들면 클라이언트를 테스트하기가 어려워질 수가 있다.

- public final 필드를 이용한 구현
    - `public static final Elvis INSATNCE = new Elvis();`
- 정적 팩터리 메소드(getInstance)를 이용한 구현
    - `public static Elvis getInstance() { return INSTANCE; }`
- Enum 단일 항목을 이용한 구현
    - `public Enum Elvis{ INSTANCE; ...}`

**원소가 하나뿐인 enum 자료형이야말로 싱글턴을 구현하는 가장 좋은 방법이다.**

- 직렬화 이슈 해결
- 간단한 구현
- reflection 방어


# Rule 04 - 객체 생성을 막을 때는 `private` 생성자를 사용하라

- 정적 메서드나 필드만 필요한 경우 (하지만 대부분의 경우 객체지향적이지 않으므로 필요할 때만 사용한다.)
- 일종의 유틸리티 클래스 (ex: `java.lang.Math`, `java.util.Arrays`)

{% highlight java %}
public class Utility {
    private Utility() {
        throw new AssertionError();
    }
}
{% endhighlight %}

Role  05 - 불필요한 객체는 만들지 말라

적절한 예시 1.

{% highlight java %}
String s1 = new String("stringette");   // 메모리 낭비
String s2 = "stringette";               // 문자열풀 재사용

Boolean b1 = new Boolean("true");       // 메모리 낭비
Boolean b2 = Boolean.valueOf("true");   // 정적 팩터리 메서드를 통한 재사용
{% endhighlight %}

적절한 예시 2.

{% highlight java %}
public class Person {
    private final Date birthDate;

    // 생성자 생략

    // 비용이 큰 객체를 재사용하기 위해 객체 속성 선언
    private static final Date BOOM_START;   // 베이비붐 시작
    private static final Date BOOM_END;     // 베이비붐 끝

    // 정적 초기화 블록(static initializer)로 개선
    static {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }

    public boolean isBabyBoomer() {
        return birthDate.compareTo(BOOM_START) >= 0 &&
                birthDate.compareTo(BOOM_END) < 0;
    }
}
{% endhighlight %}

위 코드 개선사항 : Lazy loading

Autoboxing 으로 인한 primitive과 Wrapper 클래스 객체 간 성능이슈

- 필요에 따라서 primitive와 Wrapper 클래스 간 Autoboxing 이슈가 없게 해야한다.

**Wrapper 클래스 객체 대신 primitive타입을 사용하고, 생각지도 못한 자동 객체화가 발생하지 않도록 유의한다.**

Pooling 기법도 객체의 생성 비용이 너무 크지 않다면 사용하지 않는 것이 낫다. 최신의 JVM은 성능이 매우 좋아서 왠만한 생성 비용의 객체는 Pooling 기법을 사용하지 않아도 빠르다.
생성비용이 높은 객체

- DB커넥션
- 각종 I/O 접근 (`File`, `Socket`, `Stream` 등)

다른관점으로 접근하면

- 재사용이 가능하다면 새로운 객체를 만들지 말라
- 새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 말라
- **방어적 복사** 가 요구되는 상황에서 재사용하게 되면 그냥 새로 객체를 생성하는 것보다 더 큰 비용(복잡도)가 발생하니 유의하자.

추가로 **무작정 객체를 재사용할 것이 아니라, 조건에 따라서 새로운 객체를 사용할 때를 구분해야한다.**

# Rule 06 - 유효기간이 지난 객체 참조는 폐기하라

메모리 누수가 어디서 생기는가?

{% highlight java %}
// 메모리 누수(memory leak)가 어디서 생기는가?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;  // 만기 참조 제거
        return result;
    }

    /**
     * 적어도 하나 이상의 원소를 담을 공간을 보장한다.
     * 배열의 길이를 늘려야 할 때마다 대략 두 배씩 늘인다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
{% endhighlight %}

위 elements 배열처럼, **자체적으로 관리하는 메모리가 있는 클래스를 만들 때는 메모리 누수가 발생하지 않도록 주의해야한다.**

그런다고 모든 객체를 항상 null로 처리할 필요는 없다. 예를 들어 지역변수 객체라면 해당 스코프 (메소드 또는 코드블럭)이 끝날 때 해당 지역변수 객체는 GC로 처리된다.
**객체 참조를 null 처리하는 것은 규범(norm - 해야만 하는 것)이라기 보단 예외적인 경우에 사용하는 조치가 되어야한다.** - 예를 들면 위 Stack 처럼 예외적으로 자체 메모리 관리하는 경우

**캐시(cache)도 메모리 누수가 흔희 발생하는 장소다.**

- 객체 참조를 캐시에 넣어두고 고아객체로 만드는 경우가 많다.
- 해결 방안은 `WeakHashMap`와 같은 자료구조를 사용하면 된다. : `WeakHashMap`의 경우 값항목의 수명이 키에 대한 외부 참조에 따라 결정된다.

메모리 누수가 흔희 발견되는 또 한 곳은 `Callback`이다. *이해가 명확히 안 됨*

- `Callback`를 등록하고 사용하는 클라이언트가 명시적으로 `Callback`을 명시적으로 제거하지 않는 경우가 있다.
- 해결 방안은 `Callback` 참조를 WeakReference로 가지는 것이다. (위 `WeakHashMap`을 이용하는 것과 같은 방식이다.)

# Rule 07 - 종료자 사용을 피하라

**종료자(finalizer)는 예측 불가능하며(GC알고리즘대로), 대체로 위험하고, 일반적으로 불필요하다.**

- C++ 의 소멸자(Destructor)와는 다르다 : Java는 GC가 메모리를 관리하고 C++은 사용자가 소멸자 등을 이용해서 직접 제어한다.

**긴급한 작업(time-critical) 작업을 종료자 안에서 처리하면 안 된다.**

- 종료자는 즉시 실행되지 않는다 (GC알고리즘대로) - 종료자의 더딘 실행(tardy finalization)
- 만약 긴급 작업이 요구되면 try-finally 절을 이용한다.

`System.gc` 나 `System.runFinalization` 을 실행하는 것은 일반적으로 절대 호출하면 안된다.


그러면 자원 반환은 어떻게 하지?

**명시적인 종료 메서드(termination method)를 하나 정의해서 사용한다.**

- 예를 들면 Connection#Close
- 그리고 위와 같은 종료메소드는 try-finally 문과 함께 쓰인다.

{% highlight java %}
Foo foo = new Foo();
try {
    // TODO anything
} finally {
    foo.terminate();    // 명시적 종료메소드 호출
}
{% endhighlight %}

종료 보호자 : 상속 받는 자식 클래스 객체가 부모 클래스 객체의 명시적 종료자를 호출하지 않을 경우 강제 호출을 위한 방어기법

{% highlight java %}
/**
 * 종료 보호자 숙어 (Finalizer guardian idiom)
 */
public class Foo {
    // 이 객체는 바깥 객체(Foo)를 종료 시키는 역할만 한다.
    private final Object finalizerGuardian = new Object() {
        @Override
        protected void finalize() throws Throwable {
            // 바깥 객체를 종료시키는 코드
        }
    };
    // ...
}
{% endhighlight %}

> 가능한 종료자를 사용하지 않도록 하고, 자원 반환에 대한 안전장치 (ex: 명시적 종료 메서드)를 구현하자.
