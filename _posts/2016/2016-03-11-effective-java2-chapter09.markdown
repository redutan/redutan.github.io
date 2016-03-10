---
layout: post
title: Effective Java 2/E - Chatper 09 예외 (1)
date: '2016-03-11 00:22'
tags:
  - book
  - effective-java2
---

# Rule 57 - 예외는 예외적 상황에만 사용하라

*예외를 남용하면?*

1. 성능 저하
2. 가독성 저하

**예외는 예외적인 상황에서 사용해야 한다. 제어 흐름에 이용하면 안 된다.**

*example `hasNext`가 메서드가 없었다면*

{% highlight java %}
// 이렇게 하면 곤란
try {
    Iterator<Foo> i = collection.iterator();
    while(true) {
        Foo foo = i.next();
        ...
    }
} catch (NoSuchElementExcepton e) {
}
{% endhighlight %}

위와 같인 제어흐름 일부를 일부러 예외를 사용하도록 강요하면 안된다.

**정상적인 제어를 위함 처리 방안**

1. 상태 검사 메서드 : `Iterator#hasNext()`
    2. 대부분의 상황에서 추천
    3. 가독성이 좋으며, 디버깅에 유리
2. 특이값(distinguished value) 제공 : `null`, `-1` 또는 enum상수
    3. 멀티쓰레드 환경에서 강추 - 상태 메서드가 가변적이므로

# Rule 58 - 복구 가능 상태에는 점검지정 예외를 사용하고, 프로그래밍 오류에는 실행지점 예외를 이용하라

### Throwable

**점검지정 예외(checked exception)**

client가 복구할 것으로 여겨지는 상황에 던지는 예외. 즉 복구한 권한을 준다는 것

**실행시점 예외(runtime exception)**

프로그래밍 오류를 표현할 때는 실행시점 예외를 사용한다.

일반적으로 선행조건 위반(precondition violation)을 나타낸다. 예를들면 인자유효성 검증 오류

**오류(error)**

**프로그램을 더 이상 실행할 수 없는 상황** 자원(메모리)이 부족한 상황에 발생할 수 있음.

JVM에 기반하는 프로세스 자체가 죽어버림. Exception들은 해당 Thread 가 죽을 뿐 프로세스는 살아있음.
즉 프로그램은 계속 유지가능

### 예외도 객체다

필요에 따라서 catch구문에서 적절하게 상태나 메서드를 구현해서 처리할 수 있도록 한다.

*example*

통장 잔고가 부족해서 이쳬가 실패할 경우(`NotEnoughBalanceException`)
잔고가 얼마인지 알려주는 getter(`getBalance()`) 제공

# Rule 59 - 불필요한 점검지정 예외 사용은 피하라

*너무 남발하면 사용하기 불편한 API가 될 것이다.*

# Rule 60 - 표준 예외를 사용하라

*코드 재사용성은 예외에도 적용되어야 한다.*

### 표준예외 장점

1. 쉬우며, 친숙함
2. 가독성이 좋음
3. 클래스 로딩 수가 줄어듬 (성능향상)

**자주 사용하는 표준 예외**

| 예외 | 용례 |
|-----|-----|
| `IllegalArgumentException` | null이 아닌 인자의 값이 잘못되었을 때 |
| `IllegalStateException` | 객체 상태가 메서드 호출을 처리하기에 적절치 않을 때 |
| `NullPointerExceptoin` | null 값을 받으면 안되는 인자에 null이 전달되었을 때 |
| `IndexOutOfBoundsException` | 인자로 주어진 첨자가 허용범위를 벗어났을 때 |
| `ConcurrentModificationException` | 병렬적으로 사용이 금지된 객체에 대한 병렬 접근이 탐지되었을 때 |
| `UnsupportedOperationException` | 객체가 해당 메서드를 지원하지 않을 때 |

*그 외의 예외*

- `ArthmeticException` : 수학적계산오류
- `NumberFormatException` : 숫자형식오류
