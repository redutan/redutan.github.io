---
layout: post
title: 클린코드 정리와 의문
date: '2016-07-01 17:33'
tags:
  - book
  - clean-code
---

# 정리

## 1. 깨끗한코드

**DSL은 무엇인가?**

특정 문제 도메인, 특정 문제 표현 기법, 특정 문제 해결 기법에 사용할 목적으로 만든 프로그래밍 언어나 명세 언어를 의미한다.
해당 분야 전문가들의 의사소통을 돕기 위해, 모호함을 없애며 표현력을 높인 특징이 있다.

 - [https://martinfowler.com/bliki/DomainSpecificLanguage.html](https://martinfowler.com/bliki/DomainSpecificLanguage.html)
 - [http://aeternum.egloos.com/tag/도메인특화언어](http://aeternum.egloos.com/tag/도메인특화언어)

**르블랑의 법칙** : (나쁜 코드를 해결할) 나중은 결코 오지 않는다.

![생산성 대비 시간 그래프](https://www.butterfly.com.au/sites/default/files/images/blog/cleancode/productivity-vs-time.png)

*출처 : https://www.butterfly.com.au/blog/website-development/clean-high-quality-code-a-guide-on-how-to-become-a-better-programmer*

나쁜코드의 책임은 **우리에게** 있다.

**명언**

- **명쾌한 추상화** !!!!
- **한 가지를 제대로 한다** : SRP
- **코드는 문학적** : Readability
- **중복이 없다** : DRY

코드를 작성하는 시간보다 읽는 시간이 훨씬 많다 : 고로 잘 읽을 줄 알아야한다.

*보이스카우트 규칙*

> 캠프장은 처음 왔을 때보다 더 깨끗하게 해놓고 떠나라.

**연습해 연습!!**

## 2. 의미 있는 이름

### 의도를 분명히 밝혀라

*before*

{% highlight java %}
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<>();
    for (int[] x : theList)
        if (x[0] == 4)
            list1.add(x);
    return list1;
}
{% endhighlight %}

*after*

{% highlight java %}
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<>();
    for (Cell cell : gameBoard)
        if (cell.isFlagged())
            flaggedCells.add(cell);
    return flaggedCells;
}
{% endhighlight %}

### 그릇된 정보를 피하라

accountList -> accounts, accountGroup, brunchOfAccounts

### 의미 있게 구분하라

**NO!!!**

- o1, o2, o3
- clazz, klass, NameString, theName

{% highlight java %}
getActiveAccount();
getActiveAccountDetail();
getActiveAccountInfo();
{% endhighlight %}

### 발음하기 쉬운 이름을 사용하라

`genymdhms` : `generateDateYearMonthDayHourMinuteSecond`

*before*

{% highlight java %}
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
}
{% endhighlight %}

*after*

{% highlight java %}
class Customer {
  private Date generationTimestamp;
  private Date modificationTimestamp;
  private final String recordId = "102";
}
{% endhighlight %}

### 검색하기 쉬운 이름을 사용하라

### 인코딩을 피하라

Anti *헝가리안 표기법*
Anti *맴버변수 접두어* : `m_???`
Anti *IFactory* + *Factory* : `Factory` + `DefaultFactory`

- 동적언어는 어떻게 하지?

### 자신의 기억력을 자랑하지 마라

i, j, k 정도는 넘어가주자

### 클래스 이름

- 명사
- Manager, Processor, Data, Info는 피하자???

### 메서드 이름

- 동사
- 정적팩토리 메서드 좋음

### 기발한 이름은 피하라

### 한 개념에 한 단어를 사용하라

`fetch` or `retrieve` or `get`

**일관성 있는 어휘를 사용하자.**

### 말장난을 하지 마라

VS *한 개념에 한 단어를 사용하라*

**개념이 달라진 부분에서는 일관된 단어가 아니라 다른 단어를 사용해야한다.**

### 해법 영역에서 가져온 이름을 사용하라

For Concrete Class

`SimpleAdder`, `Accumulator`

### 문제 영역에서 가져온 이름을 사용하라

For Interface

`Adder`

### 의미 있는 맥락을 추가하라

- 맥락이 이어진다면 prefix를 추가해보자
  - `addrFirstName`, `addrLastName`, `addrState`
- prefix보단 타입(Class)으로 구분하는 것이 더 낫다
  - `class Address : firstName, lastName, state`

### 불필요한 맥락을 없애라

`GSDAccount, GSDAddress` VS `Account, Address`

*인코딩을 없애라* 가 여기에 포함되는 것 같음

## 3. 함수

### 작게 만들어라!

쉽게 읽히고 이해할 수 있어야 한다.

### 한 가지만 해라!

**함수는 한 가지를 해야 한다. 그 한 가지를 잘해야 한다. 그 한 가지만을 해야한다.**

### 함수 당 추상화 수준은 하나라!

그리고 위에서 아래로 내려가기

### Switch 문

- 다형성을 이용하자.
- 한번은 참아준다

### 서술적인 이름을 사용하라!

### 함수 인수

3개 이하로 유지하라?!

- CQS
- 플래그 인수는 과연 악인가?
- 인자 수가 4개 이상이면 악인가?

**인자 객체 추천!!!** : 개념이 묶인다면

*동사와 키워드* : `write(name)`

### 부수 효과를 일으키지 마라!

- 일시적인 결합 : 실행 순서와 동시성에 따른 부수효과
  - A 메서드 호출 전 B 메서드를 호출해야 정상작동 한다.
  - 보고서는 한 번에 오직 하나만 실행 가능
  - 버튼 클릭을 처리하면 먼저 화면이 갱신되어야 한다.
- 순서 종속성 : 메서드들을 정해진 순서대로 실행해야 정상 작동

*출력 인수* 는 상태가 변하는 경우 반환하는 것이 좋다.

### 명령과 조회를 분리하라 (CQS)

*before*

{% highlight java %}
if (set("username", "unclebob"))...
{% endhighlight %}

*after*

{% highlight java %}
if (attributeExists("username")) {
    setAttribute("username", "unclebob");
}
{% endhighlight %}

*과연 항상 분리하는 것이 옳은가?*

### 오류 코드보다 예외를 사용하라!

절차적보다는 명령형으로

*try-catch 뽑아내기* : 항상 지켜야하는가?

- 오류처리도 한가지 작업이므로 메서드를 분리한다.
- 오류반환코드 의존성을 제거한다.

### 반복하지 마라! (DRY)

### 구조적 프로그래밍

- no goto, break, continue
- one input one output

### 함수를 어떻게 짜죠?

글짓기와 비슷하다.

### 결론

- 길이가 짧고
- 이름이 좋고
- 체계가 잡힌
- **가장 중요한 것은 도메인 문제 해결**

## 시스템

**'처음부터 올바르게' 시스템을 만들 수 있다는 믿음은 미신이다.**

## 창발성

**단순한 설계 규칙**

1. 모든 테스트를 실행한다.
2. 중복을 없앤다.
3. 프로그래머 의도를 표현한다.
4. 클래스와 메서드 수를 최소로 줄인다. (중복제거)

_2~4는 리팩터링을 통해 개선 가능_

**중복은 추가작업, 추가 위험, 불필요한 복잡도를 뜻한다.**

**좋은 표현 규칙**

1. 좋은 이름
2. 함수와 클래스 크기를 가능한 줄인다. (for SRP)
3. 표준 명칭을 사용한다.
4. 단위 테스트 - 예제 문서

_하지만 가장 중요한 것은 위 규칙을 실천하고자 하는 노력_

## 동시성

**동시성과 깔끔한 코드는 양립하기 어렵다. 아주 어렵다.**

**동시성은 결합(coupling)을 없애는 전략이다. 즉, 무엇(what)과 언제(when)를 분리하는 전략이다.**

**동시성 방어 원칙**

1. 동시성 코드는 다른 코드와 분리하라
2. 자료를 캡슐화하라. 공유 자료를 최대한 줄여라
3. 자료 사본을 사용하라
4. 쓰레드는 가능한 독립적으로 구현하라

## 동시성 II

프로그램에서 스레드 관련 코드를 분리하면 조율과 실험이 가능하므로 통찰력이 높아져 최적의 전략을 찾기 쉬워진다.

_[Multicore SDK][af165db1] 라는 제품을 통해서 동시성 테스트가 조금 더 쉬워진다_

  [af165db1]: https://www.ibm.com/developerworks/community/groups/service/html/communityview?communityUuid=9a29d9f0-11b1-4d29-9359-a6fd9678a2e8 "Multicore SDK"

[서버-클라이언트 샘플코드][756c1078]

  [756c1078]: https://github.com/redutan/simple-server-client-clieancode/ "서버-클라이언트 샘플코드"

# 의문

## 창발성

**p.216**

클래스와 메서드 수를 최소로 줄인다.

> 단순한 설계 규칙이라고 했는데, 클래스와 메서드의 크기를 줄이는 것이 더 낫다고 생각한다. 아마 중복되는 클래스나 메서드를 줄이라는 의미인 것으로 해석된다.
