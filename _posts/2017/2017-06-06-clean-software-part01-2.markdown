---
layout: post
title: 클린소프트웨어 Part 1. 애자일 개발(2)
date: '2017-06-06 18:23'
tags:
  - books
  - clean-software
  - refectoring
---

# Chapter 4. 테스트

**테스트의 가치**

* 검증
* 설계

## 테스트 주도 개발

1. 검증하는 테스트가 존재
2. 호출자 입장에서 모듈을 설계
2. 테스트 가능한 프로그램을 설계하도록 강제
3. 테스트의 문서화

### 테스트 우선 방식 설계의 예

{% highlight java %}
public void testMove() {
    // Given
    WumpusGame g = new WumpusGame();
    g.connect(4, 5, "E");
    g.setPlayerRoom(4);
    // When
    g.east();
    // Then
    assertEquals(5, g.getPlayerRoom());
}
{% endhighlight %}

1. 새로운 `WumpusGame`를 생성
2. 방 4의 동쪽 통로를 통해 방 5와 연결
3. 플레이어를 방 4에 넣고
4. 동쪽으로 이동하면
5. 플레이어가 방 5에 있는 검증

* `Room`이라는 개념이 과연 필요한가?
    * `Room` 사이의 연결이 중요한 가 `Room` 자체가 중요한가?
* 어떤 함수들이면 충분히 구현 가능한가?

테스트가 아주 이른 시기에 주요한 설계의 이슈를 명백히 한다는 것이다.
**테스트를 먼저 작성하는 것은 설계 의사결정 차이를 식별하는 것이다.**

### 테스트 분리

![결합된 Payroll 모델](https://www.plantuml.com/plantuml/img/Iyv9B2vM24YiBChFoU5A1lESCrAJiyEBCajIYnIgkHI0GBiSn0EBQsXorKBLkUOMvEHNfgR252KdvYINvYIMf0AD0oe3YnNa5vS0kRcfUILOTBeabYGc9HR3JKXFBO59mGqeHHQgvO8wLK5NrmxPeIZYC0rO1M5sSc4uGWz99m00)

*결합된 Payroll 모델*

* 단위테스트는 Mock object가 요구된다.

![테스트를 위해 의사 객체를 사용하는 분리된 Payroll](https://www.plantuml.com/plantuml/img/XPB13i8W44Jl_GgE6bC_mOjwDF5Wudd5jJQbXHGQqsZ_NOg0W2HownjcTeTK06sWGY9wVXsegdb7dWNHXpAGaXnXx3bZXjITmlu65CdsZhGvzxN-jhVdithCn6YBfQ5JujktWl4HCJHHO7HWe52FiZR31PTAenOxzITj1mek8AFK2fMJez0XnLCn5ROaASjDM2tYpfQ5ReFYPwLru8nUz8HIMNBWj0d7VcbY3P4VcRZrNg-uHUZwg3us5nXVyME2jKrrzsEaF2sJqzEtp8f-yXi0)

*테스트를 위해 의사 객체를 사용하는 분리된 Payroll*

{% highlight java %}
public void testPayroll() {
    // Given
    MockEmployeeDatabase db = new MockEmployeeDatabase();
    MockCheckWriter w = new MockCheckWriter();
    Payroll p = new Payroll(db, w);
    // When
    p.payEmployees();
    // Then
    assertTrue(w.checksWereWrittenCorrectly());
    assertTrue(db.paymentsWerePostedCorrectly());
}
{% endhighlight %}

*TestPayroll*

### 운 좋게 얻은 분리

**코드보다 테스트를 먼저 작성하면 설계가 개선된다.** : 여기서 운좋게 얻은 분리는 테스트를 먼저 작성해서 얻은 것이다.

## 인수 테스트

* 단위테스트 : 화이트박스 테스트
* 인수테스트 : 블랙박스 테스트

인수 테스트는 보통 스크립트를 통해서 만든다. 개인적으로는 [Cucumber][210f7703] 같은 툴

  [210f7703]: https://cucumber.io/ "Cucumber"

**장점**

1. 문서화가 된다.
2. 시스템 아키텍처에 영향

### 인수 테스트의 예

{% highlight c %}
AddEmp 1429 "Robert Martin" 3215.88
Payday
Verify Paycheck EmpId 1429 GrossPay 3215.88
{% endhighlight %}

*인수 테스트 스크립트의 한 예*

**XML로 인스 테스트 스크립트를 분리**

{% highlight xml %}
<AddEmp PayType="Salaried">
    <EmpId>1429</EmpId>
    <Name>Robert Martin</Name>
    <Salary>3215.88</Salary>
</AddEmp>
{% endhighlight %}

`AddEmp 1429 "Robert Martin" 3215.88`를 xml로 추출

{% highlight xml %}
<PayCheck>
    <EmpId>1429</EmpId>
    <Name>Robert Martin</Name>
    <GrossPay>3215.88</GrossPay>
</PayCheck>
{% endhighlight %}

`Verify Paycheck EmpId 1429 GrossPay 3215.88`를 xml로 추출

### 운 좋게 얻은 아키텍쳐

XML 추출을 통해서

* AddEmp(소스 트랜잭션)를 분리
* Payday, PayCheck(지급 수표 출력)를 분리

## 결론

* **검증**
* **문서화**
* **아키텍쳐와 설계에 긍정적 피드백**

# Chapter 5. 리팩토링

> 외부 행위를 바꾸지 않으면서 내부 구조를 개선하는 방법으로. 소프트웨어 시스템을 변경하는 프로세스 - Martin Fowler

* *긁어서 부스럼을 만들고 치우고 개선한다.*
* 개발에 주의를 기울이고 최선을 다해야 한다. - 리팩토링으로 실현

**모듈의 세 가지 기능**

1. 실행 중 동작하는 기능 : 모듈의 존재 이유
2. 변경 기능
    3. 변경하기 어려운 모듈은 그것이 제대로 동작한다 하더라도 망가진 것이며, *리팩토링* 이 필요하다.
3. 가독성
    4. 가독성이 떨어지는 것도 *리팩토링* 이 필요하다.

## 소수 생성기: 리팩토링의 간단한 예

Skip - 주로 메소드 추출을 통한 리팩토링으로 가독성을 확보하는 것이 주 내용이다.

## 결론

* 리팩토링에 투자하는 시간은 가까운 미래의 수고에 비하면 극히 적은 것이다 : **효율성 큼**
* 저녁식사 후 부엌을 청소하는 것과 비슷하다 : **부채 최소화**
* 지속적으로 **코드의 깔끔함** 을 유지하는 것이 중요하다.

# Chapter 6. 프로그래밍 에피소드

[https://github.com/redutan/bowling-cleansoftware][b440c18b]

위 github 링크를 커밋로그와 함께 참고

  [b440c18b]: https://github.com/redutan/bowling-cleansoftware "볼링게임 github 링크"
