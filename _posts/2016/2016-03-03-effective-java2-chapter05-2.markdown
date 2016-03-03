---
layout: post
title: Effective Java 2/E - Chatper 05 제네릭 (2)
date: '2016-03-03 23:50'
tags:
  - book
  - effective-java2
---

# Role 28 - 한정적 와일드카드를 써서 API 유연성을 높여라

### 전제

*`List<Number>`와 `List<Integer>` 간에는 부모-자식 관계가 성리될 수 없다. 즉 캐스팅이 불가능하다.*
하지만 논리적(상식적)으로 두 Collection 간 putAll과 같은 메소드는 호출 가능해야 할 것 같다.

위와 같은 문제를 해결하고자 **한정적 와일드카드 자료형(bounded wildcard type)** 이 존재한다.

### 알아보자

*example 한정적 와일드카드 자료형*
{% highlight java %}
// E 객체 생산자 역할을 하는 인자에 대한 와일드 카드 자료형
public void putAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
{% endhighlight %}

{% highlight java %}
// E의 소비자 구실을 하는 인자에 대한 와일드카드 자료형
public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
{% endhighlight %}

**PECS (Produce - Extends, Consumer - Super)**

위 `Stack` 예제에서 `pushAll`의 인자 src는 스택이 사용할 E형의 객체를 만드는 생산자 이므로 src의 자료형은 `Iterable<? extends E>`가 되어야 한다.
`popAll`의 dst는 Stack에 보관된 객체를 소비하므로 dst의 자료형은 `Collection<? super E>`가 되어야한다.

*example*
{% highlight java %}
public static <T extends Comparable<? super T>> T max(List <? extends T> list)
{% endhighlight %}

`Compareable`의 경우 부모의 `compare`를 사용할 수 있으므로

> 나프탈린과 와들러는 이를 Get(Consumer) and Put(Produce) Principle이라고 표현한다.

**반환값에는 와일드카드 자료형을 쓰면 안된다.**

좀 더 유연한 코드를 만들 수 있도록 도와주기는 커녕, 클라이언트 코드 안에도 와일드카드 자료형을 명시해야 하기 때문이다.

**명시적 형인자(explicit type parameter)**

컴파일러가 자료형을 명확하게 유추하지 못할 경우 사용

*before*
{% highlight java %}
Set<Integer> integers = ...;
Set<Double> doubles = ...;
Set<Number> numbers = Union.union(integers, doubles);
{% endhighlight %}

*after*
{% highlight java %}
// 명시적 형인자 .<Number> 적용
Set<Number> numbers = Union.<Number>union(integers, doubles);
{% endhighlight %}

**비한정 와일드카드 자료형 포착 helper**

*before*
{% highlight java %}
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
    // 하지만 컴파일 오류 발생. <?>이기 때문에 null이외에는 값을 넣을 수가 없음
    // 해결하려면 명시적으로 타입을 지정해주거나 포착(capture)할 수 있게 해줘야함
}
{% endhighlight %}

*after*
{% highlight java %}
public static void swap(List<? list, int i, int j) {
    swapHelper(list, j, i);
}

// 와일드카드 자료형을 포착하기 위한 private 도움 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
{% endhighlight %}

## 결론

- **API에는 와일드카드 자료형을 사용해서 유연성을 높여라 - 필수다**
- **PECS**
- `Comparable`, `Comparator`는 모두 소비자이다 - `<? super T>`

# Role 29 - 형 안전 다형성 컨테이너를 쓰면 어떨지 따져보라

*example*
{% highlight java %}
// 형 안전 다형성(heterogeneous)컨테이너 패턴
public class Favorites {
    Map<Class<?>, Object> favroites = new HashMap<Class<?>, Object>();

    public <T> void putFavorite(Class<T> type, T instance) {
        if (type == null)
            throw new NullPointerException("Type is null");
        // before 타입안정성 사라짐
        // favorites.put(type, instance);
        // after 보완코드
        favorites.put(type, type.cast(instance));
    }

    public <T> T getFavorite(Class<T> type) {
        // 타입안정성 복구 - 동적 형변환
        return type.cast(favorites.get(type));
    }
}
{% endhighlight %}

**형 안전 다형성 컨테이너 패턴의 문제점**

1. `Class`(raw type)를 인자로 넘기면 형이 지정되지 않았기 때문에 형 안정성이 깨질 수 있음
    2. `favorites.put(type, type.cast(instance));` 로 커버
2. 실체화 불가능 자료형에는 쓰일 수 없음. `List<String>.class`는 존재할 수 없음
    3. 실질적으로 완벽한 솔루션은 없음
    4. 외부 라이브러리에 `ParameterizedType`과 같은 헬퍼가 있긴 하나 라이브러리 별로 종속적임

### `asSubClass`

실행시점에 특정한 Class객체를 인자로 주어진 하위클래스의 Class객체로 형변환 해준다.

- 성공하면 Class 객체 반환
- 형변환이 불가능하면 `ClassCastException` 발생

## 결론

**컨테이너 대신 키를 제네릭으로 만들면 형인자 개수가 고정되는 제약없는 형 안전 다형성 컨테이를 만들 수 있다.**
**그런경우 Class 객체(자료형 토큰)를 키로 쓰면되며, `Column<T>`와 같이 직접 구현할 수도 있다.**
