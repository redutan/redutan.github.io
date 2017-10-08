---
layout: post
title: 클린소프트웨어 Part 4. 급여관리 시스템 패키징
date: '2017-07-12 14:33'
tags:
  - books
  - clean-software
  - design-pattern
---

# 패키지 설계의 원칙

패키지 응집도(cohesion)와 결합도(coupling)에 대한 원칙을 알아보자.

## 패키지

* 모듈의 구체화
* 클래스 보다 더 큰 무엇
    * 클래스 보다 더 큰 추상화 레벨 지닌다.
    * 구체적으로 클래스의 묶음

## 응집도 : 재사용 릴리즈 등가 원칙(REP)

> 재사용 단위가 릴리즈의 단위다.

**패키지의 모든 클래스가 재사용 가능해야한다.**

### 재사용 릴리즈의 조건

1. 통보
2. 안정성
3. 지원

## 응집도 : 공통 재사용 원칙(CRP)

> 패키지 안의 클래스들은 함께 재사용되어야 한다. 어떤 패키지의 클래스 하나를 재사용한다면 나머지도 모두 재사용한다.

* **연관된 클래스들 끼리 묶어서 군집(패키지)을 이루게 한다.**
* 연관성이 낮으면 같이 묶지 않는다.

## 응집도 : 공통 폐쇄 원칙(CCP)

> 같은 패키지 안의 클래스들은 동일한 종류의 변화에는 모두 폐쇄적이어야 한다. 패키지에 어떤 변화가 영향을 미친다면 그 변화는 그 패키지의 모든 클래스에 영향을 미쳐야 하고 다른 패키지에는 영향을 미치지 않아야 한다.

* 패키지 SRP
* `ISSUE` 대부분의 애플리케이션에서 재사용성 보다는 유지보수성이 더 중요하다? - p.326

### CCP vs CRP

* CCP를 확대하면 CRP가 축소 : 전체 패키지 수가 줄어듬 : 개발 용이성
* CRP를 확대하면 CCP가 축소 : 전체 패키지 수가 늘어남 : 재사용성
* 서로 **상충**
* `ISSUE` 따라서 패키지 구성은 *개발 용이성* -> **재사용성** 으로 옮겨가면서 진화한다. - p.326, p.333
    * 이것이 역전이 되는 경우도 있다.

## 안정성 : 의존 관계 비순환 법칙(ADP)

> 패키지 의존성 그래프에서 순환을 허용하지 말라.

### 주간빌드

* 주 4일 개발 + 주 1일 통합
* 프로젝트의 복잡도가 커짐에 따라 통합 비용이 점점 더 커지게 되는 악순환

**Worst Practice**

### 의존관계 순환을 없애기

**Best Practice**

### 의존관계 순환이 생기는 경우

![의존관계 순환이 생기는 패키지다이어그램](https://dooray.com/plantuml/img/AqXCpavCJrNmhNGiACZ9J4uioSpFuog0YuOa5cSN8_5TCXDpyjEBkL3KA-ZfsS7LGcce648zb0KLHzPjfV2cOyxRcJFURDZnPk46BW00)

* 릴리즈 시 서버 다운을 시키고 한 번에 모든 패키지의 버전을 다 올려서 배포
    * 실질적으로 하나의 큰 패키지 상태
* 테스트 시 전체 빌드
* 롤백 시도 마찬가지 - 전체 롤백

*왠지 모르게 MSA와 연결이 된다.*

## 순환을 끊기

### DIP 적용

![DIP 적용](https://dooray.com/plantuml/img/AqXCpavCJrNmhNJ9JCp9JozMgEPI08BCl9BKehJ4v5I5YE3KehBK8h1eSavYSR5215SjLm5SdsD1GKvcSc99PduUL2z4LQH2Pcv1JcfkQbv9CToGMgu81LrTEpWVLRkUdXt28Lm0)

*[RMI](https://en.wikipedia.org/wiki/Java_remote_method_invocation)의 그것과 유사하다.*

### 순환을 끊는 공통 패키지 생성

![순환을 끊는 공통 패키지 생성](https://dooray.com/plantuml/img/AqXCpavCJrNmhNGiACZ9J4uioSpFuog0YuOa5cSN8_5TCXDpyjCH8eb-gUK143ONYXaAUdfsSFsOCgZwmAgWEc0sm5bWSHGD0000)

## 하향식 설계
* `ISSUE` 패키지 구조는 하향식으로 설계할 수 없다?? - p.333
    * 시스템을 설계할 때 초기에 나오는 것 가운데 패키지 구조가 포함되지 않는다.
    * 아! 오히려 구체화된 class 등을 만들면서 패키지 구조를 만들어가는 **상향식**을 말하는 것 이다.
* 패키지 구조를 먼저 설계하면 실패할 가능성이 크다.
* 패키지 의존 관계 구조는 시스템의 논리적 설계와 함께 진화해야 한다.

## 안정성 : 안정된 의존 관계 원칙(SDP)

> 의존은 안정적인 쪽으로 향해야 한다.

* SDP가 적용된 패키지는 쉽게 변화도록 설계가 되어 있어서 **변경되리라 예상할 수 있다.**

### 안정성

* 안정성은 변화를 만들기 위해 필요한 일의 양과 관련되어 있다.

  - 세워진 동전을 넘어뜨리려면 일의 양이 적기 때문에 불안정적
  - 탁자를 넘어뜨리려면 상당한 노력이 필요하므로 안정적

![X : 안정적인 패키지](https://dooray.com/plantuml/img/AqXCpavCJrLmv2g0ifpWB6SuAuBBKK5Fpmv8Eq5fPoWD0000)

*X : 안정적인 패키지* : 독립적인 X

![Y : 불안정한 패키지](https://dooray.com/plantuml/img/AqXCpavCJrM8v2g0iXpXB2SuovahKa5Fpmue1w6TeBGp5m00)

*Y : 불안정한 패키지* : 의존적인 Y

### 모든 패키지가 안정적일 필요는 없다.

* 만약 모든 패키지가 안정적이라면 시스템은 변경할 수 없게 될 것이다.
* 공통패키지를 이용 불안정성을 유지해서 확장가능하게 한다.

## 안정성 : 안정된 추상화 원칙(SAP)

> 패키지는 자신이 안정적인 만큼 추상적이기도 해야 한다

* 안정성과 추상성 사이의 관계를 정한다.
    * 안정성 : 확장불가능
    * 추상성 : 확장가능
* 추상클래스(인터페이스)를 통해서 추상성과 안정성 사이의 균형을 확보
* **패키지의 성격에 따라 적절한 추상성을 확보해야한다.**

![stable-abstractions principle](http://www.lexicalscope.com/blog/wp-content/uploads/2012/10/plot-old.png)

*주계열 (붉은색) - A:추상성, I:불안정성*

* `0,0` : 안정적이고 구체적 : 고통의 지역(Zone of Pain)
    * ex) DB, `StringUtils`
* `1,1` : 의존성이 없고(불안정적) 추상적 : 쓸모없는 지역(Zone of Uselessness)
* `1,0 ~ 0, 1` 붉은색 직선(주계열) : 너무 추상적이지도 않고 너무 불안정적 하지도 않음
    * **주계열 양 끝점이 이상적인 위치**

# 팩토리 패턴

## DIP 위반

* 사실 `new` 키워드를 사용하기만 하면 DIP 위반
* 필요하면 DIP는 위반하는 것이 맞다.

## 팩토리 패턴으로 리팩토링

### Before

![팩토리 적용 전](https://dooray.com/plantuml/img/Iyv9B2vMSCx9JCqhuShCAqajIajCJbK8paWiIELA1ai65vOc5gKgPEOMvAJc0fKLeyWwPnObvs2HXHYfeAjhXogWfsS7Cz5A8RaeDR44HGfg75mA0000)

### After

![팩토리 적용 후](https://dooray.com/plantuml/img/Iyv9B2vMSCx9JCqhuShCAqajIajCJbK8paWiI7LBJ2x9BwfKgEPI00BjtCJirE32qiIYL0rDX8XpKMPo3aYabYiPR1QoLi_SWXo5J22HcWiq7rKEtJQOTh0D69gWiiwPHK3RC6KX7b3GrRN38G2p5CDrUdfsC3kj59ABKXDBKh4hWbeDLmG0)

**주의사항**

만약 client 에서 `new ShapeFactoryImpl()`를 하게되면 역시나 DIP를 위반하기 때문에 미리 `ShapeFactory` 를 의존성 주입 받아야 한다.

## 의존 관계 순환

{% highlight java %}
interface ShapeFactory {
    makeSquare()
    makeCricle()
}
{% endhighlight %}


* `Shape`의 파생형(Inheritance type)마다 메소드가 하나가 있다.
* `Shape`의 파생형이 하나 추가될 때 마다 의존하게 되는 **의존 관계 순환**이 생갤 수 있다.

### 개선

{% highlight java %}
class ShapeFactoryImpl implements ShapeFactory {
    public Shape make(String shapeName) throws Exception {
        if (shapeName.equals("Circle"))
            return new Circle();
        else if (shapeName.equals("Square"))
            return new Square();
        else
            throw new Exception("ShapeFactory cannot crate " + shapeName);
    }
}

{% endhighlight %}

**하지만**

* OCP 위반
* 타입안정성(type-safty) 없음

### 추가개선

{% highlight java %}
interface ShapeInstantiatable {
    Shape newInstance();
}
enum ShapeType implements ShapeInstantiatable {
    SQUARE {
        @Override
        public Shape newInstance() {
            return new Square();
        }
    },
    CIRCLE {
        @Override
        public Shape newInstance() {
            return new Circle();
        }
    };
}
class ShapeFactoryImpl implements ShapeFactory {
    public Shape make(@NonNull ShapeType shapeType) throws Exception {
        return shapeType.newInstance();
    }
}

{% endhighlight %}

## 대체할 수 있는 팩토리

**다형적인 팩토리**

* `AbstractMemberFactory`
    * `DbMemberFactory`
    * `ApiMemberFactory`
    * `XmlMemberFactory`

## 결론

* 모든 관계마다 DIP를 적용하기 위해 팩토리를 적용했다간 Interface + Factory Hell에 빠질 것이다.
* **팩토리 패턴을 사용하면 인터페이스에만 의존하면서도 구체적 객체들을 만들 수 있으므로, 구체 클래스의 변경이 잦을 때 이 패턴이 큰 도움이 된다.**

# 급여관리 사례 연구 - 2부

## 패키지 구조와 표기법

![패키지 구조와 표기법](https://dooray.com/plantuml/img/VPEnJiCm48PtFuMVe8-0L0L6f8h00npEfR5m7Cjs4HLYgDXOMPYOEZB2Iz7o3YIj8PVOQRRaE___-UwF0N47I-G6rhPH2enHKe2NZUQFZBUFKE0SNnpnlCS4NblG3aJtDMzMLY1b-E0NO1tQkEg9cU0UsqhjqPOWa5FF_eJlrLnqX5Ybs_p--jW_Rtusdxltj_tBTlUw9T9X50Luk0M7gJcS1qR8HMrKiqPwgnfzrHHnUwr8ZTfV7l1GlFz-iaKdAYnM2YUryr0Gg_-Ha_g74x8COvzqoGPbiTUG9np9ScID6LWwZXpQ2rwXG6aypKfPIYKd6flta2ZTwFOzZNAqzr9WLe5T7dSN_GUpsHaDw5G9JIGRnBEZer2nSSYwcf2N9FlVU9AEzfMieNDZlm00)

*급여 관리 패키지 다이어그램의 한 예 - p.355*

## 공통 폐쇄 원칙(CCP) 적용하기

![공통 폐쇄 원칙(CCP) 적용](https://dooray.com/plantuml/img/XLGxRW8n4Ett57i2JX0XHH291Wa5Se31EmjMl6lBDb5Gb0AbIvicLIhAbCcb0hb339YiVxkudFVczsOy0iWzB30Ni9QSCH22KGAQyZf_odBp20Ebyia9lcxW8qXhZR84WlbCGgs0790fNqc2sNCcREmuQEFdHcmyD3vg0LAXzAgO6VFaQEvAPrifGCCS54OfRIz3YxTxqaemuntZl0BgwtDZg7oD67PNbfJHb5wsYC71cvPfJIrdIfPOh1WUm2G3nwYNl6ZFhTg7vNHN7QCehwynKEP4Fnb7d1MHSKWb4ksOOQwqD4aiFbX2lR2LklcR22sCd-_F_SlcT_Er-tZVlsvtRrkdPcMrKQafLrZhitZ-Bhj8PtpYpSVNPcGMZwqJ4xJjyAgY1k8r5gsaAJ2GD4b8wXSGkuHifoOwTluOhN00il2BH7MB1AdGxhTs5imJvgusfd3rRIYK3vIWgL0kiBrDnxchnJGjEL3delHw_SIvon8NyC2yHEdxLo5CP3-LP-utQnlOWtKBK5VmxFnRemAQBn51uJP716-a4HeUrek4gzv__mce4I41XaeaA9uSgTFj3Ffip_u0)

*급여 관리 애플리케이션의 폐쇄된 패키지 계층 구조의 예 - p.357*

* 책임이 없는 패키지 : `payrolldatabaseimpl`, `textparser` : 인프라적 요소들
* 책임이 있는 패키지(독립적인 패키지) : `payrolldomain`, `applicatoin` : 도메인적 요소들

`ISSUE` **세부적인 것들은 중요한 아키텍처(구조)적인 결정에 의존해야지 의존의 대상이 되면 안된다.**

## 재사용 릴리즈 등가 원칙(REP) 적용하기

비슷한 류의 다른 어플리케이션을 구축할 시

* 구체적 요소는 재사용 불가능 : 정책이 다름
    * `classifications`, `methods`, `schedules`, `affiliations`
* 추상적 요소는 재사용 가능 : 시스템은 비슷함
    * `payrolldomain`, `application`, `payrolldatabase`
    * `payrolldatabaseimpl` 도 어쩌면 재사용 가능

*급여 관리 애플리케이션의 폐쇄된 패키지 계층 구조의 예 - p.357* 는 나름 좋은 패키지 구조이나 CRP 원칙을 제대로 지키지 않았다.

1. `paymentdomain` 는 재사용 최소 단위가 아니다.
2. **`Transaction`을 분리해야함**

![개정된급여관리클래스다이어그램](/images/2017/07/renewal-classdiagram.png)

*p.361*

* 이렇게 하므로써 설계에 추상 레이어(Transaction 관련)가 하나 더 추가된다
* `payrolldomain` 패키지를 `transaction` 없이 재사용할 수 있게 되었다.

### trade-off

* 프로젝트의 **재사용성과 유지보수성은 개선**되었다.
* 하지만 패키지가 5개 추가되었고 **의존 관계는 더 복잡**해졌다.

## 결합과 캡슐화

* 패키지내 클래스들을 캡슐화
* default(package) 접근제어자를 이용하자.
    * `public`, `protcted`, `package`, `private`
* ex) `TimeCard`, `SalesReceipt`

## 측정법

pass

## 측정값을 급여 관리 애플리케이션에 적용하기

* 팩토리를 생성해서 의존성을 낮춘다.
* 팩토리 초기화는 MainApplication에서 책임진다.
    * 아님 IoC컨테이너를 이용
* `classifications`, `methods`, `schedules`, `affiliations`는 응집력 있는 것이 낫다.
    * **트랜잭션 별로 분류**가 기능 별로 분류 보다 중요하다.
    * 어짜피 하나의 Aggregate 이기 때문이다.
    * 해당 사례에서는 이게 맞는 접근이지만, 도메인에 따라 달라질 수 있다.
