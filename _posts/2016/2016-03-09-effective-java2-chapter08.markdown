---
layout: post
title: Effective Java 2/E - Chatper 08 일반적인 프로그래밍 원칙들 (1)
date: '2016-03-09 01:37'
tags:
  - book
  - effective-java2
---

# Role 45 - 지역 변수의 유효범위를 최소화하라

### 지역변수는 처음 사용할 때 선언하라

### 거의 모든 지역변수는 초기값이 포함되어야한다.

만약 초기화 하기에 정보가 충분하지 않으면 그때까지 선언을 미뤄야한다.

*try-catch블록은 예외적일 수 있다.*

### Loop 시

**while 문 보다는 for 문을 쓰는 것이 좋다.**

{% highlight java %}
// 컬렉션을 순회할 때는 이 숙어대로 하는 것이 바람직
for (Element e : c) {
    doSomething(e);
}
{% endhighlight %}

위 코드를 보면 `e`라는 객체가 for 블록 안에 있으므로 유효범위가 좁다.

*하지만 `while`를 사용하면 `while` 블록 밖에 필요한 객체(`Iterator`)가 선언되므로 유효범위가 더
커져서 문제가 생길 수 있음*

{% highlight java %}
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
    doSometing(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while(i.hasNext()) {                // 버그 i2를 써야하는데 i를 씀
    doSometingElse(i2.next());
}
{% endhighlight %}

**하지만 위와 같은 상황에서 `for` 구문은 compile 시 오류를 탐지할 수 있다.**

### 메서드(코드블록)의 크기를 줄이고 특정한 기능에 집중하라

각 기능(책임) 별로 쪼개서(메서드 분리, 클래스 분리 등) 구현한다.

# Role 46 - for 문보다는 for-each 문을 사용하라

### for-each 문을 사용할 수 없는 경우

1. 삭제를 통한 필터링(filtering)
    2. `Iterator`의 `remove`메서드를 호출해야해서
2. 변환(transforming)
    3. 일부 원소 또는 전체 원소의 값을 수정하기 위해서 반복자(`Iterator`) 배열 첨자(`i`)가 필요함
3. 병렬순회(parallel iteration)
    4. 여러 컬렉션을 병렬적으로 순회하는 경우

# Role 47 - 어떤 라이브러리가 있는지 파악하고, 적절히 활용하라

### 장점

1. 선배 개발자의 경험을 활용한다.
2. 핵심 로직에 집중할 수 있다.
3. 성능 개선

### 라이브러리를 잘 쓸려면?

- 릴리즈 노트를 확인하고 어떤 기능이 추가됬는지 알아두자.
- 기본적으로 제공하는 라이브러리는 파악하자.
    - `java.lang.*`
    - `java.util.*`
    - `java.io.*`


## 결론

**바퀴를 다시 발명하지 말라(don't reinvent the whell)**

# Role 48 - 정확한 답이 필요하다면 float와 double은 피하라

이진 부동 소수점 연산(binary floating-point arithmetic)

**float와 double은 특히 돈과 관계된 계산에는 적합하지 않다.**

*example 계산실패*
{% highlight java %}
System.out.println(1.03 - .42);
// 0.6100000000000001 출력

System.out.println(1.00 - 9 * .10)
// 0.0999999999999998 출력
{% endhighlight %}

### 솔루션 1

**돈 계산을 할 때는 `BigDecimal`**

*하지만 `BigDecimal`을 쓰는 방법에는 2가지 문제가 있다.*

1. 사용이 불편
2. 느리다

### 솔루션 2

**`int` 또는 `long`을 사용**

달러 기준으로 달러가 아니라 센트단위로 하거나

원화 기준으로 원 소수점 이하를 소수점 이상으로 한다.

*예를들면 1,000원이라면 가상으로 100,000으로 100을 곱해서 처리*

## 결론

**통화 관련 처리를 할 때는 아래 자료형을 사용하라**

- **성능이 떨어지더라도 세밀한 계산이 필요하면 `BigDecimal`**
- **성능이 중요하다면**
    - 십진수 9자리 이하는 `int`
    - 십진수 18자리 이하는 `long`
    - 그 이상엔 `BigDecimal`

# Role 49 - 객체화 된 기본 자료형 대신 기본 자료형을 이용하라

### 기본설명

- 기본자료형(primitive type) : int, double, boolean 등
- 참조자료형(reference type) : String, List 등 일반적인 객체

**객체화된 기본 자료형(boxed primitive type)** : Integer, Double, Boolean 등

### autoboxing, auto-unboxing

- autoboxing : int -> Integer
- auto-unboxing : Integer -> int

### primitive type, boxed primitive type 차이

| 항목 | primitive type | boxed primitive type | 비고 |
|-----|----------------|----------------------|-----|
| identity(==) | 없음 | 있음 | 값 or 메모리 참조값 |
| null | 불가능 | 가능 | 객체는 null 가능 |
| 성능 & 리소스 | 좋음 | 상대적으로 나쁨 | |  

*example null*
{% highlight java %}
public class Unbelievale {
    static Integer i;
    public static void main(String[] args) {
        if (i == 42)    // 여기에서 NullPointerException이 발생
            System.out.println("Unbelievable");
    }
}
{% endhighlight %}

*i 에 비교값이 int이기 때문에 자동으로 int로 auto-unboxing(`Integer.intValue()?`) 될려다가
NullPointerException이 발생*

### 성능이슈

**기본자료형(Integer)과 객체화된 기본 자료형(int)을 한 연산 안에 엮어 놓으면 객체화된 기본 자료형은
자동으로 기본자료형으로 변환된다**(auto-unboxing)

**객체화된 기본자료형은 불변객체(immutable object)이므로 항상 String처럼 항상 방어적 복사가 실행
되고 객체화와 비객체화가 반복되므로 루프 구현이 있으면 성능이 매우 심하게 저하된다.**

예를 들면 Long으로 루프를 돌면서 합계를 구하는 것 **이런 경우 long(기본자료형)을 이용하면 된다.**

### 객체 자료형을 사용해야 하는 경우

- 컬랙션의 요소
- 제네릭
- 리플레션을 통한 호출

*객체만 가능한 경우임*

## 결론

- **가능하다면 기본자료형을 사용하자.**
- **객체 자료형의 값비교는 `equals`, 기본 자료형의 값비교는 `==`**
- **autoboxing, auto-unboxing는 성능이슈가 크다**
- **auto-unboxing 시 `NullPointerException`를 주의하라**

# Role 50 - 다른 자료형이 적절하다면 문자열 사용은 피하라

문자열은 텍스트 표현과 처리에 적합한 자료형이다.

### 안티패턴

- 문자열은 값 자료형(value type)을 대신하기에는 부족하다.
    - Integer나 int와 같은 기본자료형
- 문자열은 enum 자료형을 대신하기에는 부족하다.
- 문자열은 집합타입(aggregate type)을 대신하기에는 부족하다.
    - `String compoundKey = className + "#" + i.next();`
- 문자열은 권한(capability)을 표현하기엔 부족하다.
    - 값 자료형을 대신하기에는 부족하다와 유사
    - 문자열기반이 아니라 키 객체 기반으로 표현하자.

## 결론

**더 좋은 자료형이 있는데(아니면 만들던지) 굳이 문자열로 표현하는 것을 피하라**

- 다루기도 어려움
- 유연성이 떨어짐
- 느림
- **오류발생 가능성이 큼**
