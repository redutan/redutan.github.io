---
layout: "post"
title: "리팩토링 정리(2)"
date: "2016-08-13 15:14"
tags:
    - book
    - refectoring
---

# 6장 메서드 정리

## 메서드 추출(Extract Method)

## 메서드 내용 직접 삽입(Inline Method)

_메서드 추출_ 의 반대

## 임시변수 내용 직접 삽입(Inline Temp)

## 임시변수를 메소드 호출로 전환(Replace Temp with Query)

## 직관적 임시변수 사용(Introduce Explaining Variable)

_임시변수를 메소드 호출로 전환_ 이 더 낫다

`final`

## 임시변수 분리(Split Temporary Variable)

임시변수를 재사용하지 말고 각 대입마다 다른 임시변수를 사용

누적용 임시변수(sum)는 제외

## 매개변수로의 값 대입 제거(Remove Assignments to Parameters)

매개변수로 값을 대입하는 코드가 있을 땐 _매개변수 대신 임시변수를 사용하게 수정하자._

_before refectoring_
{% highlight java %}
int discount(int input, int quantity, int yearToDate) {
    if (input > 50) input -= 2;
{% endhighlight %}

_after refectoring_
{% highlight java %}
int discount(int input, int quantity, int yearToDate) {
    int result = input;
    if (input > 50) result -= 2;
}
{% endhighlight %}

## 메서드를 메서드 객체로 전환(Replace Method with Method Object)

## 알고리즘 전환(Substitute Algorithm)

_before refectoring_
{% highlight java %}
String foundPerson(Stirng[] people) {
    for (int i = 0; i < people.length; i++) {
        if (people[i].equals("Don")) {
            return "Don";
        }
        if (people[i].equals("John")) {
            return "John";
        }
        if (people[i].equals("Kent")) {
            return "Kent";
        }
    }
    return "";
}
{% endhighlight %}

_afeter refectroing_
{% highlight java %}
String foundPerson(Stirng[] people) {
    List candidates = Arrays.asList(new String[]{"Don", "John", "Kent"});
    for (int i = 0; i < people.length; i++) {
        if (candidates.contains(people[i])) {
            return people[i];
        }
    }
    return "";
}
{% endhighlight %}

# 7장 객체 간의 기능 이동

## 메서드 이동(Move Method)

## 필드 이동(Move Field)

## 클래스 추출(Extract Class)

두 클래스가 처리해야할 기능이 하나의 클래스에 들어 있을 땐 _새 클래스를 만들고 기존 클래스의 관련 필드와 메서드를 새 클래스로 옮기자_

## 클래스 내용 직접 삽입(Inline Class)

_클래스 추출_ 의 반대

## 대리 객체 은페(Hide Delegate)

![Hide Delegate](/images/2016/08/hideDelegate.gif)

출처 : http://kancsuki.sed.hu/sites/kancsuki.sed.hu/files/teaching/ovrt/10/catalog/hideDelegate.html

## 과잉 중개 메서드 제거(Remove Middle Man)

_대리 객체 은페_ 의 반대

## 외래 클래스에 메서드 추가(Introduce Foreign Method)

헬퍼메소드

## 국소적 상속확장 클래스 사용(Introduce Local Extension)

사용 중인 서버 클래스에 여러 개의 메서드를 추가해야 하는데 클래스를 수정할 수 없을 땐 _새 클래스를 작성하고 그 안에 필요한 여러 개의 메서드를 작성하자. 이 상속확장 클래스를 원본 클래스의 하위 클래스나 래퍼 클래스로 만들자._

![Introduce Local Extension](/images/2016/08/introduceLocalExtension.gif)    

출처 : http://kancsuki.sed.hu/sites/kancsuki.sed.hu/files/teaching/ovrt/10/catalog/introduceLocalExtension.html

**기존 메서드를 재정의 하면 문제가 생길 수 있으므로 그냥 메서드 추가만 하자**

# 8장 데이터 체계화

## 필드 자체 캡슐화(Self Encapsulate Field)

## 데이터 값을 객체로 전환(Replace Data Value with Object)

## 값을 참조로 전환(Change Value to Reference)

## 참조를 값으로 전환(Change Reference to Value)

## 배열을 객체로 전환(Replace Array with Object)

## 관측 데이터 복재(Duplicate Observed Data)

## 클래스의 단방향 연결을 양방향으로 전환(Change Unidirectional Association to Bidirectional)

## 클래스의 양방향 연결을 단방향으로 전환(Change Bidirectional Association to Unidirectional)

## 마법 숫자를 기호 상수로 전환(Replace Magic Number with Symbolic Constant)

## 필드 캡슐화(Encapsulate Field)

## 컬렉션 캡슐화(Encapsulate Collection)

## 레코드를 데이터 클래스로 전환(Replace Record with Data Class)

## 분류 부호를 클래스로 전환(Replace Type Code with Class)

## 분류 부호를 하위클래스로 전환(Replace Type Code with Subclasses)

## 분류 부호를 상태/전략 패턴으로 전환(Replace Type Code with State/Strategy)

## 하위클래스를 필드로 전환(Replace Subclass with Fields)

# 9장 조건문 간결화

## 조건문 쪼개기(Decompose Conditional)

## 중복 조건식 통합(Consolidate Conditional Expression)

## 조건문의 공통 실행 코드 빼내기(Consolidate Duplicate Conditional Fragments)

## 제어 플래그 제거(Remove Control Flag)

## 여러 겹의 조건문을 감시절로 전환(Replace Nested Conditional with Guard Clauses)

## 조건문을 재정의로 전환(Replace Conditional with Polymorphism)

## Null 검사를 널 객체에 위임(Introduce Null Object)

## 어설션 넣기(Introduce Assertion)

# 10장 메소드 호출 단순화

## 메서드명 변경(Rename Method)

## 매개변수 추가(Add Parameter)

## 매개변수 제거(Remove Parameter)

## 상태 변경 메서드와 값 반환 메서드를 분리(Separate Query from Modifier)

## 메서드를 매개변수로 전환(Parameterize Method)

## 매개변수를 메서드로 전환(Replace Parameter with Explicit Methods)

## 객체를 통째로 전달(Preserve Whole Object)

## 매개변수 세트를 메소드로 전환(Replace Parameter with Method)

## 매개변수 세트를 객체로 전환(Replace Parameter with Object)

## 쓰기 메서드 제거(Remove Setting Method)

## 메서드 은폐(Hide Method)

## 생성자를 팩토리 메서드로 전환(Replace Constructor with Factory Method)

## 하향 타입 변환을 캡슐화(Encapsulate Downcast)

## 에러 부호를 예외 통지로 교체(Replace Error Code with Exception)

## 예외 처리를 테스트로 교체(Replace Exception with Test)
