---
layout: post
title: Effective Java 2/E - Chatper 05 제네릭 (1)
date: '2016-03-03 00:41'
tags:
  - book
  - effective-java2
---

> 간단/명료 하게 작성하도록 하자.

# Rule 23 - 새 코드에는 무인자 제네릭 자료형을 사용하지 마라

타입 선언부에 형인자(Type parameter)가 포함되어 있으면 제네릭 타입이라 부른다.

마찬가지로 메소드 선언부에 Type parameter이 포함되어 있으면 제네릭 메서드라 부른다.

**용어정리**

- Type parameter : 형인자 - `< >`
    - Formal type parameter : 형식 형인자 - `<T>`
    - Actual type parameter : 실 형인자 - `<String>`
- Parameterized type : 형인자 자료형 - `List<T>`
- Raw type : 무인자 자료형 - `List`
- Unbounded wildcard type : 비한정적 와일드카드 자료형 - `List<?>`
- Bounded wildcard type : 한정적 와일드카드 자료형 - `List<? extends String>`
- Bound type parameter : 한정적 형인자 - `<E extends String>`

### 목적

*형인자 선언을 통해서 컴파일 타임에 Type safety(형 안정성)을 확보하기 위함*

**고로 무인자 자료형(Raw type)을 쓰면 형 안정성이 사라지고, 제네릭의 장점 중 하나인 표현력(expressiveness) 측면에서 손해를 보게 된다.**

### `List`, `List<Object>`, `List<?>` 구분

- `List` : 모든 타입 추가 가능 *안전하지 않음*
- `List<Object>` : 모든 타입 추가 가능
    - 하지만 `List<String>`와 호환되지 않음 - 캐스팅 불가능
- `List<?>` : 비한정적 와일드카드 자료형(unbounded wildcard type)
    - null 이외에 어떤 원소도 넣을 수 없음(어떤 자료형이 있는 건데 어떤 자료형이 담긴지는 알 수가 없음)
- `List<T extends String>` : 한정적 와일드카드 자료형(bounded wildcard type)
    - String 포함 String 상속 받는 타입 호환
    - `List<String>`와 호환됨 - 캐스팅 가능

### `instanceof` 사용법

{% highlight java %}
// instanceof 연산자에는 무인자 자료형을 써도 OK
if (o instanceof Set) {         // 무인자 자료형
    Set<?> m = (Set<?>) o;      // 와일드카드 자료형
}
{% endhighlight %}

# Rule 24 - 무점검 경고(unchecked warning)를 제거하라.

*example : 무점검 경고*
{% highlight java %}
Set<Lark> exaltation = new HashSet();

/* 컴파일경고
    ???.java:4: warning: [unchecked] unchecked conversion
    found : HashSet, required: Set<Lark>
    Set<Lark> exaltation = new HashSet();
*/
{% endhighlight %}

**모든 무점검 경고는, 절대로 무시하지 말아야한다. 가능한 없애야한다.**

- Type safety
- No `ClassCastException`

*단, 제거할 없는 경고 메세지는 형 안정성이 확실할 때문 `@SupressWarnings("unchecked")` 어노테이션을 사용해 억제하기 바란다.*
가능하면 **작은 범위에 적용하라.**
그리고 해당 경고를 억제한 이유를 주석으로 표현하라.

# Rule 25 - 배열 대신 리스트를 써라

**배열과 리스트(Collection)의 차이**

- 배열 : 공변 자료형(covariant)
    - `Sub extends Super`이면 `Super[]` -> `Sub[]`으로 형변환 가능
- 리스트 : 불변 자료형(invariant)
    - `Sub extends Super`이면 `List<Super>` -> `List<Sub>`으로 형변환 **불가능**
    - **하지만** `List<Super>` -> `List<? extends Sub>`으로 형변환 가능

*하지만 취약한 것은 배열이다.* 역시 불변식이 좋음

### 리스트의 장점

**1. 컴파일 시 타입안정성 확보**

{% highlight java %}
// 실행 시 예외 발생 (컴파일 성공)
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in";  // ArrayStoreException 발생

// 컴파일 되지 않음
List<Object> ol = new ArrayList<Long>();    // 자료형 불일치
ol.add("I don't fit in");
{% endhighlight %}

**2. 배열은 실체화(reification) 자료형**

간단히 말해서 객체를 생성하면서 초기화 까지 가능한 자료형이라는 뜻. 하지만 `List`는 안됨

*example*

`String strs = new String[] {"str1", "str2"}`

**이해안됨 - 실체 불가능 자료형**

`E`, `List<E>`, `List<String>`와 같은 자료형은 실체화 불가능(non-refiable) 자료형으로 알려져 있다.
쉽게 말하자만, 프로그램이 실행될 때 해당 자료형을 표현하는 정보의 양이 컴파일 시점에 필요한 정보의 양보다 적은 자료형이 **실체화 불가능 자료형** 이다.

> 실행시점(runtime)에 해당 자료형의 모든 정보를 이용할 수 없는 자료형(a type this is not completely available at runtime)이라는 뜻이라는데 명확하게 이해가 안된다.

## 결론

| 구분 | Type | 실체화 | 형안정성 |
|-----|------|------|--------|
| 배열 | 공변자료형 | 가능 | 실행타임에 보장 |
| 제네릭 | 불변자료형 | 불가능 | 컴파일타임에 보장 |

**배열과 제네릭을 뒤섞어 쓰다가 컴파일 오류나 경고메시지를 만나게되면, 배열을 리스트로 바꿔야겠다는 생각이 본능적으로 들어야 한다.**

# Rule 26 - 가능하면 제네릭 자료형으로 만들 것

*example*
{% highlight java %}
// 제네릭 사용해 작성한 최초 Stack 클래스 - 컴파일 되지 않는다.
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
        // 컴파일오류
        // Stack.java:8: generic array creation
        //     elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void pusht(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size = 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;  // 만기 참조 제거
        return result;
    }
    ...
}
{% endhighlight %}

### 배열을 사용하는 제네릭 자료형을 구현할 때 실체화 불가능한 자료형(E)으로 배열을 생성할 수 없는 경우 해결책

**1. 제네릭 배열을 만들 수 없다는 조건을 우회하는 것이다.**

배열 객체 생성 시 `Object` 배열을 만들어서 제네릭 배열 자료형으로 형변환(cast)하는 방법

*example*
{% highlight java %}
// elements 배열에는 push(E)를 통해 전달된 E 형의 객체만 저장된다.
// 이 정도면 형 안정성은 보장할 수 있지만, 배열의 실행시간 자료형은 E[]가 아니라 항상 Object[] 이다.
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
{% endhighlight %}

**2. 배열의 자료형을 E[]에서 Object[]로 바꾸는 것이다.**

*example*
{% highlight java %}
private Object[] elements;
...
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
...
public E pop() {
    if (size = 0)
        throw new EmptyStackException();
    // 자료형이 E인 원소만 push하므로, 아래의 형변환은 안전하다.
    @SuppressWarnings("unchecked") E result = elements[--size];
    elements[size] = null;  // 만기 참조 제거
    return result;
}
{% endhighlight %}

**제네릭 배열 해결책 정리**

1. 무점검 형변환(unchecked cast) 경고 억제의 위험성은 스칼라(scala) 자료형 보다 배열 자료형이 더 크기 때문에, 두번째 해법이 더 낫다고 볼 수 있다.
2. 하지만 현실적으로 배열을 사용하는 코드가 여러군데라면 배열 생성 시 첫번째 방법으로는 E[]으로 한번만 형변환을 하면되지만, 두번째 방법으로는 코드 여기저기에서 E로 형변환을 해야하므로
**첫번째 방법이 더 낫다**

## 결론

**가능하면 하위호환성을 유지하면서 제네릭 자료형으로 리팩터링하라.**

# Rule 27 - 가능하면 제네릭 메서드로 만들 것

## 패턴
- 일반적인 사용
- 제네릭 정적 팩터리 메서드 (1.6 이하)
- 제네릭 싱글턴 팩터리 패턴
    - `Collections.emptyList()`
- 재귀적 자료형 한정(recursive type bound)
    - `<T extends Compareable<T>> T max(List<T> list)`

## 결론

- **제네릭 메서드는 클라이언트의 cast 비용을 줄이고 타입안정성도 높다.**
- **시간 날 때 기존 메서드를 제네릭 메서드로 확장해 놓으면 기존 클라이언트 코드를 깨지 않고도 새 사용자에게 더 좋은 API를 제공할 수 있다.**
