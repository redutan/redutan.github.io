---
layout: post
title: DDDQ 4장 깊은 통찰을 향한 리팩터링
date: '2015-10-20 10:10'
tags:
  - ddd
---

> **DDDQ(Domain Driven Design Quickly) - 도메인 주도 설계란 무엇인가?**
>
> 라는 책의 간단 요약 정리

### 지속적인 리팩터링

설계와 구현사이 피드백을 통한 코드 고도화

**코드 리펙터링**

- 애플리케이션의 기능에 변화를 주지 않고 코드를 더 좋게 만들기 위해 재설계하는 절차
- 많은 방법론과 패턴이 존재한다.

**DDD 리펙터링**

프로젝트 진행 중에 도메인에 대한 새로운 통찰이 생기고 어떤 것들은 점점 명확해지며, 둘간의 관계가 발견되기도 한다.
이러한 것은 모두 리펙터링을 통해 설계에 반영되어야 한다.

- 위와 같은 **깊은 통찰** 을 위한 리펙터링은 *패턴이 존재하지 않는다.*
- *모델링* : 비즈니스 명세서를 읽고 명사와 동사를 찾는 것이다. 명사는 클래스로 동사는 메소드로 변환된다.
- 하지만 위와 같은 기본적인 *모델링* 은 아주 심한 단순화이며, 여기에서 점점 깊은 통찰을 가지도록 **개선(refactor)** 해야 한다.
- 유연한 설계가 바탕이 되어야한다. 유연하지 못하면 **개선** 에 따른 비용이 커진다
- 가능한 작은 단위로 나누어서 진행해야 한다.

**정교한 도메인 모델은 도메인 전문가와 개발자들이 밀접하게 엮인 조직이 반복적으로 리팩터링을 수행하지 않는다면 만들어질 수 없다.**

### 핵심 개념 드러내기

리펙터링은 작은 단계로 나누어 진행되며, 그 결과 작은 개선의 연속으로 나타난다. 그런데 소규모의 변경이 큰 차이를 초래하는 경우가 있다.
바로 이것이 **도약(Breakthrough)** 이다.

어떤 암시적 개념이 핵심개념이며, 이것을 명시적 개념으로 모델링하게되면 도약의 기회가 생긴다.

*암시적개념 : 도메인 전문가와 이야기할 때 외부로 노출되지 않은 것*

암시적 개념을 발견하는 방법

- 언어를 주의 깊게 듣기
- 설계 영역에서 분명하지 않는 부분을 주의 깊게 들여다 보기
- 지식체계(모델링 관계 설정 등)를 만들 때 모순에 부딪칠 경우
- 해당 도메인의 문헌을 활용 - 기존예시

##### (암시적인 것을) 명시적인 개념으로 만들어 낼 때 유용한 추가개념

**제약조건(constraint)**

- 불변식(invariant)?를 표현하는 간단한 방법

*예제*

as-is

{% highlight java %}
public class Bookshelf {
  private int capacity = 20;
  private Collection content;
  public void add(Book book) {
    if(content.size() + 1 < = capacity) {
      content.add(book);
    } else {
      throw new IllegalOperationException(
        "The bookshelf has reached its limit.");
      )
    }
  }
}
{% endhighlight %}

제약사항 메소드 분리를 통한 리팩토링 후

{% highlight java %}
public class Bookshelf {
  private int capacity = 20;
  private Collection content;
  public void add(Book book) {
    if(isSpaceAvailable()) {
      content.add(book);
    } else {
      throw new IllegalOperationException(
        "The bookshelf has reached its limit.");
      )
    }
  }
  // 제약사항 메소드 분리
  private boolean isSpaceAvailable() {
    return content.size() < capacity;
  }
}
{% endhighlight %}

**처리(process)**

- 일반적으로 처리는 절차적인 코드로 표현된다.
- OOP를 사용하므로 절차적 접근은 허용하지 않는다.
- 고로, 처리를 구현하는 최고의 방법은 서비스(Service)를 이용하는 것

**명세(specification)**

- 객체가 특정 기준을 만족하는지 여부를 확인
- [명세 패턴][3beb455c]을 참조
  - 추가 블로그 자료 : http://vandbt.tistory.com/4


  [3beb455c]: https://en.wikipedia.org/wiki/Specification_pattern "명세 패턴"
