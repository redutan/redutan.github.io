---
layout: post
title: Effective Java 2/E - Chatper 07 메서드 (2)
date: '2016-03-08 00:27'
tags:
  - book
  - effective-java2
---

# Rule 42 - varargs는 신중히 사용하라

가변 인자 메서드(variable arity method)

*example*
{% highlight java %}
// varargs의 간단한 사용 예
static int sum(int... args) {
    int sum = 0;
    for (int arg : args) {
        sum += arg;
    }
    return sum;
}
{% endhighlight %}

*example 배열을 출력하는 올바른 방법*
{% highlight java %}
System.out.println(Arrays.toString(myArray));
{% endhighlight %}

**마지막 인자가 배열이라고 해서 무조건 뜯어고칠 생각을 버려라. varargs는 정말로 임의 개수의 인자를
처리할 수 있는 메서드를 만들어야 할 때만 사용하라.**

*타입이 다르면 빌더패턴이 더 나을 듯*

# Rule 43 - null 대신 빈 배열이나 컬렉션을 반환하라

**null대신 빈배열이나 컬렉션을 반환하라**

반환할 시에는 길이가 0인 불변객체로 반환하면 성능저하도 없고 재사용할 수 있다.

*example Array*
{% highlight java %}
// 컬렉션에서 배열을 만들어 반환하는 올바른 방법
private final List<Cheese> cheesesInStock = ...;

private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

/**
 * @return 재고가 남은 모든 치즈 목록을 배열로 만들어 반환
 */
public Cheese[] getCheeses() {
    return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
{% endhighlight %}

*example List*
{% highlight java %}
// 컬렉션 복사본을 반환하는 올바른 방법
public List<Cheese> getCheeseList() {
    if (cheeseInStock.isEmpty()) {
        return Collections.emptyList(); // 언제나 같은 리스트 반환
    } else {
        return new ArrayList<Cheese>(CheesesInStock);
    }
}
{% endhighlight %}

# Rule 44 - 모든 API 요소에 문서화 주석을 달라

**좋은 API 문서를 만들려면 API에 포함된 모든 클래스, 인터페이스, 생성자, 메서드, 그리고 필드 선언에
문서화 주석을 달아야한다**

public, protected 요소들이라고 생각하면된다.

### 메서드

**메서드가 무엇을 하는지를 설명** 해야지 메서드가 어떻게 그 일을 하는지를 설명해서는 안된다.

- 단, 계승을 위한 클래스는 어떻게 하는지 설명이 있어야함

**태그**

- `@param` 인자
- `@return` 반환값을 설명
- `@throws` 어떤 경우(if) 예외가 발생하는가

*example*
{% highlight java %}
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param index index of the element to return; must be
 *        non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= size()})
 */
E get(int index);
{% endhighlight %}

*임의의 HTML태그를 사용할 수 있다.*

*html이 깨지는 것이 걱정되면 `<code>`나 `<tt>`태그를 쓰는 것보단 `{@code}`태그를 쓰는 편이
낫다* 여러줄인 경우에는 `<pre>{@code }</pre>` 태그 안에 넣으면 된다.

*다른 대안으로는 `{@literal}`태그가 있는데 `{@code}`태그와 다른 점은 고정폭폰트가 아니라는 점이다*

**1. 선행조건과 후행조건**

- 선행조건 : 클라이언트가 메서드를 호출하려면 반드시 참(true)이 되어야 하는 조건들
    - `@throws` 태그를 통해서 무점검 예외로 기술
    - 인자유효성
- 후행조건 : 메서드 실행이 성공적으로 끝난 다음에 만족되어야 하는 조건들

**2. 부작용**

**3. 스레드안정성**

**4. 직렬화**

### 주의사항

요약문에 마침표가 여러 번 포함되면 주의하라. 의도치 않게 잘릴 수 있다.

**제네릭 자료형이나 메서드에 주석을 달 때는 모든 자료형 인자들을 설명해야한다.**
{% highlight java %}
/**
 * An object that maps keys to values (중략)
 *
 * @param <K> the type of keys maintained by this maps
 * @param <V> the type of mapped values
 */
public interface Map<K, V> {
    ....
}
{% endhighlight %}

**enum 자료형에 주석을 달 때는 상수 각각에도 주석을 달아 주어야한다.**
{% highlight java %}
/**
 * 교향악단에 쓰이는 악기 종류.
 */
public enum OrchestraSection {
    /** 플루트, 클라리넷, 오보에 같은 목관악기 */
    WOODWIND,
    /** 프렌치 혼이나 트럼펫 같은 금관악기 */
    BRASS,
    ...
}
{% endhighlight %}

**어노테이션 자료형에 주석을 달 때는 모든 맴버에도 주석을 달아야한다.**
{% highlight java %}
/**
 * 지정된 예외를 반드시 발생시켜야 하는 테스트 메서드임을 명시.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * 어노테이션이 붙은 테스트 메서드가 테스를 통과하기 위해 반드시 발생시켜야 하는 예외.
     * (이 Class 객체가 나타내는 자료형의 하위 자료형이기만 하면 어떤 예외든 상관없다.)
     */
    Class<? extends Throwable> value();
}
{% endhighlight %}

### 추가 팁

**패키지 문서화** : `package-info.java`

**상속** : `{@inheritDoc}`

> 공식문서인 오라클이 제공하는 [How to Write Doc Comments for the Javadoc Tool][9fbaa149] 꼭 읽어보자


[9fbaa149]: http://www.oracle.com/technetwork/articles/java/index-137868.html "How to Write Doc Comments for the Javadoc Tool"
