---
layout: post
title: Effective Java 2/E - Chatper 07 메서드 (1)
date: '2016-03-06 23:59'
tags:
  - book
  - effective-java2
---

# Role 38 - 인자의 유효성을 검사하라

메서드나 생성자를 구현할 때는 받을 수 있는 인자에 제한이 있는지 따져보고, 제한이 있다면 문서
(javadoc : `@throws`)에 남기고, 메서드 앞부분에서 검사하도록 해야 한다.

# Role 39 - 필요하다면 방어적 본사본을 만들라

자바는 상대적으로 안전한 언어이다. 하지만 스스로 노력이 없이는 안전하지 못하게 된다.

**여러분이 만드는 클래스를 사용하는 클라이언트가 불변식(invariant)을 망가뜨리기 위해 최선을 다할
것이라는 가정하에, 방어적으로 프로그래밍 해야한다.**

구현한 클래스가 불변식이라고 해도 인자로 전달한 객체가 가변객체라면 객체 외부에서 변경해서 불변식이 깨진다.

고로 **생성자로 전달되는 변경가능 객체를 반드시 방어적으로 복사** 해서 객체 내부에서 사용해야한다.

*그리고 인자의 유효성 검증은 방어적 복사본에 시행한다.*

*example 생성자*
{% highlight java %}
// 인자를 방어적으로 복사함
public Period(Date dtart, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(
            this.start + " after " + this.end);
}
{% endhighlight %}

**인자로 전달된 객체의 자료형이 계승이 가능하다면(`final` 클래스가 아니라면) 방어적 복사본을 만들 때
`clone`을 사용하지 않도록 해야 한다.**

또한 **getter을 통해서 변경 가능한 객체를 조회해서 수정할 수도 있으므로, getter에서도 방어적 복사본을
반환해야 한다.**

*example getter*
{% highlight java %}
// 수정된 접근자 - 내부 필드의 방어적 복사본 생성
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
{% endhighlight %}

### 방어적 복사가 필요한 클래스

- `Date`
- 각종 배열
- 가변 Collection들

## 결론

**클라이언트에게 받는 인자나 반환하는 객체가 변경가능한 객체라면 방어적 복사본으로 제공해야 한다.**

만약 방어적 복사를 하기가 힘들다면(복사 비용이 크거나 변경이 필요한 경우) 문서로 명시해서 객체를 변경하지
않도록 가이드 해야한다.

# Role 40 - 메서드 시그너처는 신중하게 설계하라

### 1. 메서드 이름은 신중하게 고르라
- 표준작명
- 일반적으로 합의 사항에 부합해야 한다.

### 2.편의 메서드(convenience method)를 제공하는 데 너무 열 올리지 마라.
- 메서드 책임에 충실하게 하라 (pull its weight)

### 3. 인자 리스트(parameter list)를 길게 말들지 마라
- 인자수 4개 이하로
- 자료형이 같은 인자들이 길게 연결된 인자 리스트는 특히 더 위험하다. - 순서를 착각할 수 있음

#### 솔루션 3가지
1. 여러 메서드로 분리
2. (인자로 쓰일)도움 클래스를 만들어 인자를 그룹별로 분류
3. 메서드를 빌더패턴을 적용한 클래스로 분리

### 4. 인자의 자료형으로는 클래스보다 인터페이스가 좋다

`HashMap` 보다는 `Map`. 그러면 각종 구현체를 쓸 수 있어서 유연해짐

### 5. 인자 자료형으로 boolean을 쓰는 것보다는, 원소가 2개인 enum을 쓰는 것이 낫다.
- 가독성 향상
- 추후 확장 가능

# Role 41 - 오버로딩할 때는 주의하라

**오버로딩된 메서드는 정적으로 선택되지만, 재정의(override)된 메서드는 동적으로 선택된다.**

재정의는 실행시점의 자료형을 바탕으로 실행되는 반면, 오버로딩은 컴파일 시점 자료형(실행 시간 자료형은
영향을 주지 못함)을 바탕으로 실행된다.

*example 오버로딩을 실행시점 가능하게 수정*
{% highlight java %}
public static String classify(Collection<?> c) {
    return c instanceof Set ? "Set" :
        c instanceof List ? "List" : "Unknown Collection";
}
{% endhighlight %}

**고로, 오버로딩을 사용할 때는 혼란스럽지 않게 사용할수 있도록 주의해야한다.**

### 전략

**1. 혼란을 피하는 안전하고 보수적인 전략은, 같은 수의 인자를 갖는 두 개의 오버로딩 메서드를 API에
포함시키지 않는 것이다.**

메서드의 인자들의 성격이 **확실히 다르다(radically different)** 를 만족하면 된다.

- 즉 상호간 cast가 불가능한 것
- Wrapping class와 primitive type은 만족하지 못함

*example 인자의 성격이 다르지 않는 경우 - 즉 안전하지 않음*

- `List<Integer>` 인터페이스의 `remove(Integer)`, `remove(int)` 메서드
- `String` 클래스의 `valueOf(char[])`, `valueOf(Object)` 정적 메서드
