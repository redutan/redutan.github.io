---
layout: "post"
title: Effective Java 2/E - Chatper 10 병행성 (2)
date: "2016-03-13 22:42"
tags:
  - book
  - effective-java2
---

# Rule 70 - 스레드 안정성에 대해 문서로 남겨라

### 스레드 안정성 수준

1. 변경불가능(immutable) : 불변식 `String`, `Integer`
2. 무조건적 스레드 안정성(uncoditionally thread-safe) : `ConcurrentHashMap`
3. 조건부 스레드 안정성(conditionally thread-safe) : iterator
4. 스레드 안정성 없음 : `ArrayList`, `HashMap`
5. 다중 스레드에 적대적(thread-hostile)

스레드 안정성 어노테이션

`Immutable`, `ThreadSafe`, `NotThreadSafe`

*example 콜렉션 뷰 순회 숙어*
{% highlight java %}
Map<K, V> m = Collections.synchronizedMap(new HashMap<K, V>());
...
Set<K> s = m.ketSet();  // 동기화 블록 안에 있을 필요가 없음
...
synchronized(m) {   // s가 아니라 m에 동기화
    for (K key : s)
        key.f();
}
{% endhighlight %}

*example lock 객체 숙어*
{% highlight java %}
// DoS 공격을 피하기 위한 private lock 객체 숙어
private final Object lock = new Object();

public void foo() {
    synchronized(lock) {
        ...
    }
}
{% endhighlight %}

## 결론

**어떤 방식(말로 풀건, 스레드 안정성 어노테이션등)을 이용해서 모든 클래스는 자신의 스레드 안정성을 분명히
밝혀야한다.** - synchronized 키워드는 문서적으로 전혀 의미없다.

무조건적 스레드 안정성을 가진다면 _private lock 객체 숙어_ 를 사용을 고려하라.

# Rule 71 - 초기화 지연은 신중하게 하라

초기화 지연(lazy initialization) : 필드 초기호를 실제로 값이 쓰일 때까지 미루는 것

**대부분의 경우 초기화 지연을 하는 것 보단 일반 초기화를 하는 것이 낫다.**

### 초기화 숙어들

**일반적인 초기화**
{% highlight java %}
private final FieldType field = computeFieldValue();
{% endhighlight %}

일반적인 방법

**지연 초기화 - synchronized**
{% highlight java %}
// 동기화된 접근자를 사용한 객체 필드 초기화 지연 방법
private FieldType field;

synchronized FieldType getField() {
    if (feild == null)
        filed = computeFieldValue();
    return field;
}
{% endhighlight %}

안전하긴 하지만 항상 동기화가 걸리기 때문에 성능이 나쁨

**지연초기화 - 초기화 지연 담당클래스 숙어**
{% highlight java %}
// 정적 필드에 대한 초기화 지연 담당 클래스 숙어
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}
static FieldType getField() { return FieldHolder.field; }
{% endhighlight %}

안전하면서 성능도 챙기는 좋은 숙어

_단점으로는 한번 재초기화를 하기 힘들다._

**지연초기화 - Double check**
{% highlight java %}
// 이중 검사 패턴을 통해 객체 필드 초기화를 지연시키는 숙어
private volatile FieldType field;

FieldType getField() {
    FieldType result = field;
    if (result == null) {   // 첫 번째 검사 (락 없음)
        synchronized(this) {
            result = field;
            if (result == null) // 두 번째 검사 (락 있음)
                field = result = computeFieldValue();
        }
    }
    return result;
}
{% endhighlight %}

부득이하게 담당클래스를 이용할 수 없거나 재초기화가 필요한 경우 사용

한 번은 동기화 전, 한 번은 동기화 후에 검사해(그래서 double)서 단 한 번만 락이 걸리게 한다.

## 결론

**대부분의 초기화 지연을 시키지 말아야한다.**

- 객체 필드는 Double check 숙어
- 정적 필드는 초기화 지연 담당클래스 숙어

# Rule 72 - 스레드 스케줄러에 의존하지 마라

정확성을 보장하거나 성능을 높이기 위해 스레드 스케줄러 에 의존하는 프로그램은 이식성이 떨어진다.
_즉 플랫폼(OS) 환경에 따라서 스레드 우선순위가 달라질 수 있기 때문에 실행 결과가 달라질 수 있다._

### 멀티 스레드 관리 방안

- **스레드 풀을 적절하게** : 실행 가능 스레드의 평균적인 수가 프로세서 수보다 너무 많아지지 않도록 하는 것
- **태스크를 적당히 작게** : 너무 작으면 context switching 이슈가 생김
- **서로 독립적으로**
- **바쁘게 대기하지 않는다** : 공유객체에 너무 자주 접근하지 않는다.

스레드 우선순위에 대한 이슈를 해결하기 위해서 `Thread.yield()`나 **스레드 우선순위를 바꾸는 기법** 을
사용하지 말자. 별 효능이 없다.

*프로그램의 구조를 바꿔서 실행 가능한 스레드의 수를 줄이는 것이 좋은 해결책이다.*

만약 병행성 테스트가 목적이라면 차라리 `Thread.sleep(1)`를 사용하라. - `Thread.sleep(0)`은 바로
return 하므로 효과가 없다

## 결론

**플랫폼이 제공하는 스레드 스케줄러에 의존하지 말고 적절한 멀티스레드 관리 방안을 마련하라.**

마찬가지로 우선순위를 조작하는 메서드에도 의존하지 마라 - 그저 힌트 수준이지 강제성이 없다.

# Rule 73 - 스레드 그룹은 피하라

스레드 그룹은 거의 폐기수순이다. - _스레드 안정성에도 취약하다_

쓸모 있는 기능이 무점검 예외를 던졌을 때 가져오는 유일한 수단인 `ThreadGroup.uncaughtException`
이 있었는데 이 마저도 `Thread.setUncaughtExceptionHandler`가 제공되었으니 이것을 사용 하라.

# 결론

가능한 절대로 사용하지 말고 대신 `ThreadPoolExecutor`를 사용 하라.
