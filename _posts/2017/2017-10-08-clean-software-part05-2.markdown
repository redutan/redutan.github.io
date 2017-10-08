---
layout: post
title: 클린소프트웨어 Part 5. (2) 기상 관측기 사례 연구
date: '2017-10-08 16:08'
tags:
  - books
  - clean-software
---

# 사례 연구: 기상관측기

저가형 소프트웨어 님버스

* 1.0 : 경쟁사를 대응하고자 하는 저가형 솔루션
    * 만들수록 적자가 나는 구조 (비싼 하드웨어)
    * 현재 시간이 없어서 고급하드웨어를 사용
* 2.0 : 이후 경쟁사를 따돌리는 완성형 저가형 솔루션
    * 1.0은 비싼 하드웨어 이므로 2.0으로 빨리 변환해야함

## 님버스-LC 소프트웨어 설계

* 하드웨어 독립적 아키텍처
* 테스트
    * mocking을 통한 단위테스트
* 스케줄러
    * 정기적으로 기상관측 정보를 계측
* 기압동향
* UI 독립적

## 스케줄러 생각해 보기

1. 정기적으로 계측하기 위해서 스케줄러가 필요하다.
2. 스케줄러가 *UI*와 모든 *감지기*와 연결되어 있다
    3. 이것은 감시기가 추가되거나 UI에 의해서 변경이 닫혀있지 않게 된다 - OCP 위반
4. UI와의 결합은 *옵저버 패턴*으로 해결 - p.461 ~ 462
5. 감지기와의 결합은 *Listener*로 해결(일종의 콜백 개념) - p.463
    6. 감지기 자신이 풀링 주기를 알아야 한다!

## 감지기(Sensor)

![감지기 클래스 다이어그램](/images/2017/10/sensor-class-diagram.png)

**1 Cycle**

1. 알람시계(스케줄러)가 감지기를 깨운다.
2. 감지기는 `check()` 메소드를 통해서 시작한다.
    3. 먼저 계측값을 읽어온다.
    4. 읽어온 계측값을 옵저버를 통해서 UI에 발행

**하드웨어 다형성**

* `TemperatureSensor#check()` : 템플릿 메소드
* `Nimbus1TemperatureSensor#read()` : 템플릿의 다형성 메소드
    * 즉 다른 감지기들도 `read()` 만 재정의 하면 된다.

## 감지기 개선

![감지기 브릿지 패턴 적용](/images/2017/10/sensor-apply-bridge.png)

*브릿지 패턴 적용*

![감지기 팩토리 패턴 적용](/images/2017/10/sensor-apply-factory.png)

*팩토리 패턴 적용*

## 스케줄러 개선

* `AlarmClock`도 감지기 개선과 유사하게 브릿지와 팩토리를 바탕으로 리팩토링 한다.
* `ISSUE` 자바에서는 객체의 이름(`String`)으로 객체를 생성할 수 있다.
    * 타입안정성이 없다. - 아래코드 참조

{% highlight java %}
public static void main(String[] args) {
    Class tkClass = Class.forName(args[0]);
    StationToolKit st = (StationToolKit) tkClass.newInstance();
}
{% endhighlight %}

# 패키징

* 이미 전에 배웠다시피 클래스를 설계하고 나서 패키지를 설계한다 : 상향식


1. UI가 다른 클래스들과 같은 패키지라서 분리하는 것이 좋을 것 같다. - p.470
2. `ISSUE` 그랬더니 순환참조(?)가 생겨버렸다. - p.472
3. 순환참조를 DIP를 이용해서 해결했다.
    4. `WeatherStationComponent`를 만들어서 해결 - p.473

# 영속화

## API

{% highlight java %}
interface PersistentImpl {
    void store(String name, Serializable obj);
    Object retrieve(String name);
    List directory(String regExp);
}
{% endhighlight %}

## 24시간 기록

## 일일(0 ~ 24시까지의) 최솟값과 최댓값

1. 감지기를 통해 온도의 변화가 있으면 최솟값, 최댓값을 다시 저장한다. : `currentReading()`
2. 00시가 되면 현재 온도를 조회한 수 새로운 일일 최솟값, 최댓값을 생성한다. : `newDay()`

*p. 477 ~ 478*

### 하지만

정책 (최솟값과 최댓값을 구하기)과 영속성 **관심사가 섞여 있다.**

* 이럴 때 해결방안은 역시나 *프록시 패턴*을 이용함

![프록시 패턴을 이용한 정책과 영속성 분리](/images/2017/10/segration-with-proxy.png)

*프록시 패턴을 이용한 정책과 영속성 분리*

* `HiLoDataDbProxy` : 영속성 담당
* `HiLoDataImpl` : 정책 담당

## 팩토리와 초기화

* `TemperatureHiLo` 입장에서는 `HiLoData`에만 의존하고 싶다.
* 하지만 누군가는 `HiLoDataImpl`, `HiLoDataDbProxy`를 생성해야한다.
* 이러한 것을 팩토리를 통해서 캡슐화한다.
* 팩토리를 사용하는 것은 다형성을 포함해서 복잡한 생성이나 구상체를 캡슐화 할 때도 유용하다.
* 여기에서는 `DataToolKit`라는 추상팩토리를 통해서 해결한다. - p.483

## 영속성으로 인한 패키지 구조 변경

* 영속성 레이어가 추가됨에 따라 패키지 구조를 다시 변경해야 한다.
* 패키지는 어플리케이션이 확장될 수록 계속 바뀔 수 있다.

## 그럼 영속성 팩토리는 누가 생성?

* `new persistence.DataToolKitImpl()` -> `persistence.Scope.init()` -> `main()`
* 하지만 위 케이스에서 일부 의존성 때문에 또 다른 팩토리를 생성하고 패키지를 분리하는 것은 *오버엔지니어링*인 것 같다. - p.486 *이것이 정말로 필요한가?*
    * `ISSUE` 현실적으로 요구사항에 맞게 일부 강결합은 인정해야할 것 같다.
* 애초에 이런 고민은 IoC컨테이너(ex: Spring 프레임워크)를 이용하면 쉽게 해결할 수 있다.

# 결론

* 해당 장을 보고 실제 설계 & 구현을 이루는 예시를 통해서 practice를 얻기를 바란다.
