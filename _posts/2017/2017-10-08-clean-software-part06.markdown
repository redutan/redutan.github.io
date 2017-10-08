---
layout: post
title: 클린소프트웨어 Part 6. ETS 사례 연구
date: '2017-10-08 16:26'
tags:
  - books
  - clean-software
---

# 비지터 패턴

> 클래스 계층 구조에 새로운 메소드를 추가할 필요가 있지만, 그렇게 하는 작업은 고통스럽거나 설계를 해치게 된다.

## 디자인 패턴의 비지터 집합

타입 계층 구조를 변경하지 않고도 새로운 메소드를 계층 구조에 추가할 수 있음

* 비지터(Visitor)
* 비순환 비지터(Acyclic Visitor)
* 데코레이터(Decorator)

## 비지터 패턴 상세

**이중 디스패치(dual dispatch)** : 실행되는 연산이 요청의 종류와 두 수신자의 타입에 따라 달라진다는 뜻

![비지터 패턴 클래스 다이어그램](/images/2017/10/visitor-pattern-class-diagram.png)

*비지터 패턴 클래스 다이어그램*

![비지터 패턴 시퀀스 다이어그램](/images/2017/10/visitor-pattern-sequence-diagram.png)

*비지터 패턴 시퀀스 다이어그램*

## 연산의 변화 vs 구조의 변화

* 새로운 구상체가 계속 추가된다면 Visitor은 독이 된다.
* 새로운 연산이 계속 추가된다면 Vistor은 좋은 선택이다.

## 비순환 비지터 패턴

* down-casting가 있어서 쓰기가 꺼려진다.
    * 타입안정성이 약해짐
* 만약 **generic**을 이용할 수 있다면 더 낫지 않을까?

{% highlight java %}
public void accept(ModemVisitor v) {
    try {
        ErniedModemVisitor ev = (ErnieModemVisitor) v;  // !!!
        ev.visit(this);
    } catch (ClassCastException e) {
    }
}
{% endhighlight %}

*p.505 ErnieModem.java 일부*

## 데코레이터 패턴

*소리나는 다이얼 모뎀을 만들고 싶다*

* OCP, SRP 원칙을 지킬려면 어떻게 해야할까?
* **데코레이터**!!!

{% highlight java %}
public class LoudDialModem implements Modem {
    private Model modem;

    public LoudDialModem(Modem m) {
        this.modem = m
    }
    public void dial(String pno) {
        modem.setSpeakerVolume(10); // 데코레이팅!!!
        modem.dial(pno)
    }
    ...
}
{% endhighlight %}

## 확장 객체 패턴

* 아답터 패턴과 유사하다.
* 단점은 역시나 많은 subclass를 동반하는 것이다.
* 장점은 OCP와 SRP를 지킬 수 있다

![확장 객체 패턴 클래스 다이어그램](/images/2017/10/extend-object-pattern-class-diagram.png)

*확장 객체 패턴 클래스 다이어그램*

참고 : [The Extension Objects Pattern.pdf](http://www.ccs.neu.edu/research/demeter/adaptive-patterns/visitor-usage/papers/plop96/extension-objects-gamma.ps)

# 스테이트 패턴

## 유한 상태 오토마타의 개괄

유한 상태 기계 : Finite State Machine
* ex) 개찰구

![개찰구 상태 다이어그램](/images/2017/10/turnstile-state-diagram.png)

*개찰구 상태 다이어그램*

## 구현기법 : 중첩된 switch/case

{% highlight java %}
switch (state) {
    case LOCKED :
        switch (event) {
            case COIN :
                state = UNLOCKED;
                turnstileController.unlock();
                break;
            case PASS :
                turnstileController.alarm();
                break;
        }
        break;
    case UNLOCKED :
        switch (event) {
            case COIN :
                turnstileController.thankyou();
                break;
            case PASS :
                state = LOCKED;
                turnstileController.lock();
                break;
        }
        break;
}
{% endhighlight %}

* turnstile : 개찰구

### 접근제어가 패키지인 상태변수

`ISSUE` 테스트를 진행할려면 테스트에서 상태를 변경할 수 있어야 한다.

* 이를 위해서 같은 패키지에서 만들어지는 테스트를 위해서 상태 변수를 패키지 접근제어로 변경하였다.
* effective-java에 의하면 패키지 접근제어도 캡슐화가 깨지지 않는 것으로 보이므로 큰 문제가 없다고 판단되며, 본인도 종종 이것을 애용하는 편이다.
    * 하지만 가능했다면, 패키지 생성자나 패키지 setter 메소드로 조금 더 제한하면 좋을 것 같다.
* 만약 C++이었다면 friend로 해결가능했을 것이다.

## 중첩된 switch/case의 장단점

* 간단한 상황에서는 가독성이 높고 효율적
* 복잡한 상황에서는 가독성이 떨어지고 유지보수도 어려워지며, 실수에 취약하다.
    * 중복코드도 많이 생긴다.

## 구현기법 : 전이 테이블 해석

{% highlight java %}
// 테이블 구축
public Trunstile(TurnstileController action) {
    turnstileController = action;
    addTransition(LOCKED, COIN, UNLOCKED, unlock());    // 마지막 메서드는 람다식으로 하면 더 나을 것 같다.
    addTransition(LOCKED, PASS, LOCKED, alarm());
    addTransition(UNLOCKED, COIN, UNLOCKED, thankyou());
    addTransition(UNLOCKED, PASS, LoCKED, lock());
}
...
// 전이 엔진
public void event(int event) {
    for (Transition transition : transitions) {
        // 아래 분기문은 transition의 메서드로 분기하면 좋을 것 같다.  : transition.isTransferable(state, event)
        if (state == transition.currentState && event == transition.event) {
            state = transition.newState;
            transition.action.execute();
        }
    }
}
{% endhighlight %}

### 전이 테이블 해석의 장단점

* 정규적이며, 가독성이 좋다.
* runtime 시 테이블 교체 가능(동적 제어)
* `ISSUE` 테이블의 양이 커지면 검색 시간이 오래 걸린다??
* `ISSUE` 테이블을 지원하기 위한 코드의 양도 많아 진다??

## 구현기법 : 스테이트 패턴

![스테이트 패턴 클래스 다이어그램](/images/2017/10/state-pattern-class-diagram.png)

* `ISSUE` 하지만 예제에서는 Turnstile가 구상상태(`TunstileLockedState` 등)에 의존하고 있어서 DIP 원칙이 깨지고 있다. - p.544

### Gof : 스테이트 패턴

![스테이트 패턴 by Gof](/images/2017/10/state-pattern-by-gof.png)

*Tcp를 예제로 한 스테이트 패턴 클래스 다이어그램*

* TCP Established : 연결상태
* TCP Listen : 연결대기
* TCP Closed : 종료

참고 : [TCP 연결 상태 의미](http://hyacinth.byus.net/moniwiki/wiki.php/TCP%20연결%20상태%20의미)

### 스테이트와 스트래터지

![strategy vs state](/images/2017/10/strategy-vs-state.png)

* 스테이트 : 상태의 다형성 확보
* 스트래터지 : 알고리즘(행위)의 다형성 확보
* context의 상태 변경이 일어나는가?

### 스테이트 패턴의 장단점

* OCP, SRP를 만족할 수 있다.
* 상태 마다 각각 subclass가 필요해서 비용이 많이 든다.
* 상태 기계의 모든 논리를 한 번에 파악할 수 없다 (vs 전이 테이블)

## 상태 기계 컴파일러(SMC)

**전이 테이블 + 스테이트 패턴** = 3차원 유한 상태 기계(THREE-LEVEL FINITE STATE MACHINE)

![SMC](/images/2017/10/smc-class-diagram.png)

### 상태 기계 컴파일러 장단점

* 그냥 좋다. = Best practice
