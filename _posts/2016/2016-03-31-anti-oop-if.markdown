---
layout: post
title: 'Anti-OOP : if 를 피하고 싶어서'
date: '2016-03-31 21:32'
tags:
  - OOP
---

실제 개발을 진행하다보면 `if-else` 구문이나 `switch-case` 구문을 정말 많이 사용하게 됩니다.
논리적 사고 방식을 기반으로 할 시 **분기** 처리는 필수적인 영역인데 이 때 유용하게 사용할 수 있습니다.
하지만 절차적 프로그래밍에서 객체지향적 프로그래밍으로 **패러다임 전환** 이 일어나면서 로직 분기문은
과거 절차적 프로그래밍의 유산으로 객체지향적 사고방식을 방해하는 요소 중 하나가 되었습니다.

제가 현업에서 업무를 진행할 시 팀원들에게 가장 많이 하는 말 중에 하나가 **가능한 분기문을 없애야 한다.**
입니다. 그런데 해당 내용에 대해서 말로써 표현을 할려고하니 이해가 힘든부분이 있어서 이렇게 포스팅을 하게
되었습니다.

_단순 반복적인 분기문을 최소화하려는 노력을 기울이면 그 댓가로 객체지향적 사고와 코드_ 를 얻을 수 있습니다.

> 좀 더 일반화 하면 [DRY(중복배제)][c7510147] 를 지킬려고 하는-boiler plate한 코드를 줄일려는- 노력을 하다 보면
그 해답 중 하나인 **객체지향적 사고방식** 을 얻을 수 있습니다.

  [c7510147]: https://ko.wikipedia.org/wiki/%EC%A4%91%EB%B3%B5%EB%B0%B0%EC%A0%9C "DRY"

아래 예제를 통해서 알아보겠습니다.

# 쇼핑몰 할인정책

많은 분들이 이해하기 쉬운 쇼핑몰 도메인으로 예제를 꾸며 보았습니다.

같은 종류의 분기처리가 코드 곳곳에서 중복해서 발생하는데 처리하는 도메인 로직은 미묘하게 다른 경우가 많죠.
_실제 아래와 같은 코드를 현업에서 정말 많이 볼 수 있습니다._

아래 예제는 **할인정책을 분기로 처리하는 코드** 입니다.

{% highlight java %}
public class PaymentService {
    // 실시간 할인내역 확인
    public Discount getDiscount(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NAVER:네이버검색-10%, DANAWA:다나와검색-15% FANCAFE:팬카페-1000원)
        String discountCode = ...;

        // 할인금액
        long discountAmt = 0;
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            discountAmt = productAmt * 0.1;
        } else if ("DANAWA".equals(discountCode)) { // 다나와검색 할인
            discountAmt = productAmt * 0.15;
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페인입 할인
            if (productAmt < 1000)  // 할인쿠폰 금액보다 적은경우
                discountAmt = productAmt;
            else
                discountAmt = 1000;
        }
        return Discount.of(discountAmt, ...);
    }

    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NAVER:네이버검색-10%, DANAWA:다나와검색-15% FANCAFE:팬카페-1000원)
        String discountCode = ...;

        // 결제금액
        long paymentAmt = 0;
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            paymentAmt = productAmt * 0.9;
        } else if ("DANAWA".equals(discountCode)) { // 다나와검색 할인
            paymentAmt = productAmt * 0.85;
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페인입 할인
            if (productAmt < 1000)  // 할인쿠폰 금액보다 적은경우
                paymentAmt = 0;
            else
                paymentAmt = productAmt - 1000;
        } else {
            paymentAmt = productAmt;
        }
        ...
    }
    ...
{% endhighlight %}

같은 유형의 분기문이 메소드 곳곳에서 발견됩니다.

위와 같은 코드가 보이면 일반적으로 중복 제거를 위해서 **메소드 추출** 을 통한 리팩토링을 하게 됩니다.

# Step 1. 메소드 추출

{% highlight java %}
public class PaymentService {
    // 실시간 할인내역 확인
    public Discount getDiscount(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NAVER:네이버검색-10%, DANAWA:다나와검색-15% FANCAFE:팬카페-1000원)
        String discountCode = ...;

        // 할인금액
        long discountAmt = getDiscountAmt(discountCode, productAmt);
        ...
    }

    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NAVER:네이버검색-10%, DANAWA:다나와검색-15% FANCAFE:팬카페-1000원)
        String discountCode = ...;

        // 결제금액
        long paymentAmt = productAmt - getDiscountAmt(discountCode, productAmt);
        ...
    }

    private long getDiscountAmt(String discountCode, long productAmt) {
        long discountAmt = 0;
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            discountAmt = productAmt * 0.1;
        } else if ("DANAWA".equals(discountCode)) { // 다나와검색 할인
            discountAmt = productAmt * 0.15;
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페인입 할인
            if (productAmt < 1000)  // 할인쿠폰 금액보다 적은경우
                discountAmt = productAmt;
            else
                discountAmt = 1000;
        }
        return discountAmt;
    }
    ...
{% endhighlight %}

위와 같인 `getDiscountAmt` 메소드 추출만 하는 것만으로 코드의 중복이 제거되고 깔끔해졌습니다.

하지만 우리의 목표는 여기가 아닙니다. 위와 같은 메소드 추출은 기존 절차적 프로그래밍에서 가능한 리팩터링
기법입니다. 여기에서 중요한 것은 `getDiscountAmt` 즉 **할인에 관한 정책 분기를 결제 서비스
객체의 책임으로 두는게 맞느냐** 입니다. 할인과 결제는 분리되는 것이 나은 것 같습니다.

_두 관계를 더 구제척으로 톺아보면 쇼핑몰 도메인 상 할인과 결제가 연관관계(Association)를 가지고는
있으며, 강결합이 아닌 유연성을 가져야합니다._

또한 이 객체 뿐만 아니라 다른 객체에서 할인 로직을 사용할 시에는 `getDiscountAmt` 메소드가 또 중복으로
발생하게 됩니다. if-else도 같이 중복되겠죠.

여기에서부터 **추상화 시각** 이 필요합니다. 추상화 시각을 바탕으로 추상화할 영역을 추출해야합니다.
추상화를 시도할 시 **책임(역할)을 기반으로 분리할 도메인 로직의 핵심을 집어내야합니다.** 현재 예재에서 분리할
도메인 로직은 바로 **할인** 입니다. 할인 로직의 핵심은 바로 할인금액을 구하는 것이라고 할 수 있죠.
즉 `getDiscountAmt` 가 분리할 핵심적인 행위(메소드)가 되며, 추상화 시키면(interface) 되는 것입니다.

추후에 설명 드리겠지만 여기에서 행하는 추상화는 바로 **일반화** 가 되겠습니다.

이제 `할인할 수 있는(할인 금액을 구하는 것)` 추상화를 기반으로 해당 인터페이스를 추출하겠습니다.
구현을 간단히 설명하면 각 할인정책이 하나의 클래스로 분리된다고 보시면 됩니다. 이러한 리팩터링 기법을
**인터페이스 추출** 이라고 부릅니다.

    기본적으로 클래스 추출 후 다형성이 요구될 경우 인터페이스 추출을 하지만 클래스 추출 과정은 생략 하였습니다.
    클래스 추출 자체가 강결합이고 `SRP`(책임분리)만 충족할 뿐 더 중요한 `OCP`(유연성)는 만족하지 못하기 때문입니다.

> 객체지향 5원칙 SOLID : [https://ko.wikipedia.org/wiki/SOLID][a18cd70a]

  [a18cd70a]: https://ko.wikipedia.org/wiki/SOLID "SOLID"

# Step 2. 핵심 인터페이스 추출

{% highlight java %}
public interface Discountable {
    /** 할인없음 */
    Discountable NONE = new Discountable() {
        @Override
        public long getDiscountAmt(long originAmt) {
            return 0;
        }
    };

    long getDiscountAmt(long originAmt);
}

class NaverDiscountPolicy implements Discountable {
    @Override
    public long getDiscountAmt(long originAmt) {
        return originAmt * 0.1;
    }
}

class DanawaDiscountPolicy implements Discountable {
    @Override
    public long getDiscountAmt(long originAmt) {
        return originAmt * 0.15;
    }
}

class FancafeDiscountPolicy implements Discountable {
    private long discountAmt = 1000L;

    @Override
    public long getDiscountAmt(long originAmt) {
        if (originAmt < discountAmt)
            return originAmt;
        return discountAmt;
    }
}
{% endhighlight %}

{% highlight java %}
public class PaymentService {
    // 실시간 할인내역 확인
    public Discount getDiscount(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NONE:할인없음, NAVER:네이버검색-10% 할인, FANCAFE:팬카페-1000원 할인)
        String discountCode = ...;
        // 할인정책
        Discountable discountPolicy = getDiscounter(discountCode);

        // 할인금액
        long discountAmt = discountPolicy.getDiscountAmt(productAmt);
        ...
    }

    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NONE:할인없음, NAVER:네이버검색-10% 할인, FANCAFE:팬카페-1000원 할인)
        String discountCode = ...;
        // 할인정책
        Discountable discountPolicy = getDiscounter(discountCode);

        // 결제금액
        long paymentAmt = productAmt - discountPolicy.getDiscountAmt(productAmt);
        ...
    }

    // 팩토리 메서드
    private Discountable getDiscounter(String discountCode) {
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            return new NaverDiscountPolicy();
        } else if ("DANAWA".equals(discountCode)) { // 다나와검색 할인
            return new DanawaDiscountPolicy();
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페 할인
            return new FancafeDiscountPolicy();
        } else {
            return Discountable.NONE;
        }
    }
    ...
{% endhighlight %}

인터페이스를 적절하게 추출하였으나, 팩토리 메소드가 해당 객체 내에 있기 때문에 아직 완벽하게 클라이언트
(`PaymentService`) 객체와 할인 구상클래스가 강결합 상태입니다. 현재 상태로 봐서는 메소드 추출 시점과
별로 나아진 점이 없네요. 아직 리팩터링이 부족합니다.

`getDiscounter(String)` 팩토리 메서드. 즉 할인정책을 생성하는 메소드도 분리해야 할 것 같습니다.
이번에도 이전과 마찬가지로 인터페이스 추출을 시도할 건데 여기에서 분리할 것은 바로 **할인 정책 생성**
입니다.

생성 패턴 중 가장 대중적인 **추상 팩토리 패턴** 을 사용하겠습니다.

# Step 2-2. 추상 팩토리 패턴

{% highlight java %}
/** 할인 생성 팩토리 */
public interface DiscounterFactory {
    Discountable getDiscounter(String discountName);
}

public class SimpleDiscounterFactory {
    @Override
    Discountable getDiscounter(String discountName) {
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            return new NaverDiscountPolicy();
        } else if ("DANAWA".equals(discountCode)) { // 다나와검색 할인
            return new DanawaDiscountPolicy();
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페 할인
            return new FancafeDiscountPolicy();
        } else {
            return Discountable.NONE;
        }
    }
}
{% endhighlight %}

{% highlight java %}
    DiscounterFactory discounterFactory = new SimpleDiscounterFactory();

    // 실시간 할인내역 확인
    public Discount getDiscount(...) {
        ...
    }

    // 결제처리
    public void payment(...) {
        ...
    }

    private Discountable getDiscounter(String discountCode) {
        return discounterFactory.getDiscounter(discountCode);
    }
{% endhighlight %}

이제야 깔끔하게 역할이 분리되었으며, 추상화를 바탕으로 유연함의 기틀이 마련되었습니다.

## 추상 팩토리 정리

### 역할(책임)

- 결제 : `PaymentService`
- 할인 : `Discountable`
- 할인생성 : `DiscounterFactory`

### 유연성확보

**`DIP`를 통해서 강결합의 구상클래스가 아니라 유연한 인터페이스를 바탕으로 객체 의존성 확보**

## 클래스 다이어그램

![Factory](/attach/2016/POEAA/ClassDiagram-AntiOOP-Factory.png)

실제로 분기문을 최소화 하기 위해서 가장 많이 사용되는 패턴이 바로 이 **추상 팩토리 패턴** 입니다.
또한 추상 팩토리 패턴을 사용하면서 코드 중복을 없애기 위해 90% 이상 **템플릿 메소드 패턴** 을 병행해서
사용하게 되니 두 패턴은 거의 같이 사용하게 된다고 봐도 무방하다. _위 예제는 템플릿 메소드 패턴을 적용할
필요가 없어서 사용하지 않았습니다._

_참고로 템플릿 메소드 패턴은 자바에서 귀중한 상속(`extends`)을 사용하기 때문에 주의를 요합니다._
주류적인 행위에 대한 것이 아니면 해당 패턴을 사용하는 것보다는 헬퍼클래스 [구성을 통한 위임][e726d400] 을 이용하는 것을 추천
합니다.

  [e726d400]: http://redutan.github.io/2016/02/26/effective-java2-chapter04#rule-16------ "Rule 16 - 계승(상속)하는 대신 구성하라"

> 추상 팩토리 패턴 : [https://en.wikipedia.org/wiki/Abstract_factory_pattern][25ced73b]

> 템플릿 메소드 패턴 : [https://en.wikipedia.org/wiki/Template_method_pattern][878df5f5]

  [25ced73b]: https://en.wikipedia.org/wiki/Abstract_factory_pattern "Abstract factory pattern"
  [878df5f5]: https://en.wikipedia.org/wiki/Template_method_pattern "Template method pattern"

## 결론

이 정도 리팩터링 해도 객체지향적이다 라고 말할 수 있습니다. 만약 새로운 할인 정책이 추가되면 `PaymentService`와
같은 클라이언트들은 변경할 필요 없이 `Discountable`를 확장하여 새로운 할인정책 구상클래스를 추가하고,
`SimpleDiscounterFactory`에 `else if` 하나만 추가하면 기능 확장이 가능한 것이다.

지금까지 위에서 설명한 것이 바로 `OCP`이다.

**(`PaymentService`) 변경(수정)에는 닫혀 있고, 확장(할인정책 추가/변경)에는 열려 있다.**

단순히 분기문을 객체지향적으로 처리하고자 한 시도에서 SOLID 중 가장 중요한 원칙 중 하나인 `OCP`를 이해하게
되었습니다.

## 또다른 시선

잠깐 다른 방향으로 리팩터링을 해보는 것도 알아보겠습니다.

만약 다루는 속성(데이타)이 정적이라면 추상 팩터리 패턴 대신 enum을 사용하는 것을 고려해봐도 좋습니다.

# enum 기반 리팩터링

{% highlight java %}
@Getter
public enum DiscountPolicy implements Discountable {
    /** 네이버 할인 */
    NAVER(10, 0L) {
        @Override
        public long getDiscountAmt(long originAmt) {
            return originAmt * this.discountRate / 100;
        }
    },
    /** 다나와 할인 */
    DANAWA(15, 0L) {
        @Override
        public long getDiscountAmt(long originAmt) {
            return originAmt * this.discountRate / 100;
        }
    },
    /** 팬카페 할인 */
    FANCAFE(0, 1000L) {
        @Override
        public long getDiscountAmt(long originAmt) {
            if (originAmt < this.discountAmt)
                return originAmt;
            return this.discountAmt;
        }
    }
    ;
    private final int discountRate;
    private final long discountAmt;

    DiscountPolicy(int discountRate, long discountAmt) {
        this.discountRate = discountRate;
        this.discountAmt = discountAmt;
    }
}
{% endhighlight %}

{% highlight java %}
@Slf4j
public class PaymentService {
    // 실시간 할인내역 확인
    public Discount getDiscount(...) {
        ...
    }

    // 결제처리
    public void payment(...) {
        ...
    }

    private Discountable getDiscounter(String discountCode) {
        if (discountCode == null)
            return Discountable.NONE;
        try {
            return DiscountPolicy.valueOf(discountCode);
        } catch (IllegalArgumentException iae) {
            log.warn("Not found discountCode : {}", discountCode);
            return Discountable.NONE;
        }
    }
    ...
{% endhighlight %}

위와 같이 enum 별로 할인 인터페이스를 구현하게 만들어서 깔끔한 코드를 만들 수 있습니다.
이전의 추상 팩토리 패턴과는 달리 분기문 자체가 아예 필요없고, 확장도 enum 상수를 하나 구현하면 되므로,
더 나아보입니다.

하지만 위에서도 언급했다시피 데이타가 정적 즉 변화할 가능성이 없는 객체를 다룰 경우에만 사용하는 것이 좋습니다.
단점을 살펴보면

## enum 사용 시 단점

- enum이기 때문에 상태가 변화할 수 없다.
- 상속이나 확장이 일부 제한되기 때문에 유연성이 조금 떨어진다.

만약 예를 들어서 네이버의 할인율이 10%인데 갑자기 15%로 변경될 경우 어플리케이션을 다시 배포해야 합니다.
물론 위 추상 팩토리 패턴을 이용할 경우에도 마찬가지 입니다만, 조금 더 개선하면 유연하게 대응할 수 있습니다.

다음 예재를 통해서 알아봅시다.

# Step 3. 도메인 객체(Entity) 기반 리팩터링

{% highlight java %}
@Entity
@Inheritance
public abstract class AbstractDiscounter implements Discountable {
    @Id @GeneratedValue @Column
    private long id;
    @Column
    private String code;
    @Column
    private String name;
}

/** 할인율 */
@Entity
@DiscriminatorValue("RATE")
public class RateDiscounter extends AbstractDiscounter {
    @Column
    private int rate;

    @Override
    public long getDiscountAmt(long originAmt) {
        return originAmt * rate / 100;
    }
}

/** 금액할인 */
@Entity
@DiscriminatorValue("AMT")
public class AmtDiscounter extends AbstractDiscounter {
    @Column
    private long amt;

    @Override
    public long getDiscountAmt(long originAmt) {
        if (originAmt < amt)
            return originAmt;
        return amt;
    }
}
{% endhighlight %}

## 실제 테이블에 저장된 데이타

**Discounter 테이블**

| *id | dtype | *code | name | rate | amt |
|----:|:-------:|:------:|:------:|------:|-----:|
| 1 | RATE | NAVER | 네이버 | 10 | 0 |
| 2 | RATE | DANAWA | 다나와 | 15 | 0 |
| 3 | AMT | FANCAFE | 팬카페 | 0 | 1000 |

_*가 표기된 칼럼은 값이 Unique하다_

<br/>

{% highlight java %}
@Repository
public interface DiscounterRepository extends
        JpaRepository<AbstractDiscounter, Long> {
    /**
     * 할인코드로 할인 조회
     */
    AbstractDiscounter findByCode(String code);
}
{% endhighlight %}

{% highlight java %}
@Component
class SimpleDiscounterFactory {
    @Autowired
    DiscounterRepository discounterRepository;

    @Override
    public Discountable getDiscounter(String discountCode) {
        if (discountCode == null)
            return Discountable.NONE;
        AbstractDiscounter discounter =
                discounterRepository.findByCode(discountCode);
        return discounter == null ? Discountable.NONE : discounter;
    }
}
{% endhighlight %}

{% highlight java %}
public class PaymentService {
    @Autowired
    DiscounterFactory discounterFactory;
    // 실시간 할인내역 확인
    public Discount getDiscount(...) {
        ...
    }

    // 결제처리
    public void payment(...) {
        ...
    }

    private Discountable getDiscounter(String discountCode) {
        return discounterFactory.getDiscounter(discountCode);
    }
    ...
{% endhighlight %}

스프링 프레임워크와 JPA를 사용한다는 가정으로 예제코드를 표현했습니다.

## with JPA

- 변경하는 값을 조회하기 위해서 `Repository`를 사용하였습니다.
- 다형성을 위해서 **상속 관계 매핑** 을 이용하였습니다.

### 상속 관계 매핑 다이어그램

![상속 관계 매핑 다이어그램](/attach/2016/POEAA/ClassDiagram-AntiOOP-SSR.png)


현업에서는 복잡한 도메인 로직이 많습니다. 위 예와 같이 만약 네이버 할인이 15%로 변경할 경우
이전과 같은 방식으로 구현할 시에는 어플리케이션을 재배포 해야 대응이 가능합니다. 근본적인 원인은
엔터프라이즈 어플리케이션에서 **대부분의 객체는 상태가 언제나 쉽게 객체(Entity-도메인 객체) 이기 때문입니다.**

도메인 로직을 품은 객체가 상태가 정적인 경우는 거의 없습니다.

> 변하지 않는 것은 모든 것은 변한다는 사실뿐이다. -톰피터스

**이렇게 구성하면 분기문으로 처리할 내용을 각각의 Entity 객체로 분리해서 처리하면 되어서 분기문을 최소화
(또는 제거)할 수 있으며, 자주 변하는 요구사항과 객체 상태의 변화(할인정책 변화)에 대해서도 유연하게
대응할 수 있게 됩니다.**

## 숨겨진 리팩터링

코드를 잘 보시면 `RateDiscounter`(할인율), `AmtDiscounter`(금액할인) 두개의 구상 클래스가 있는 것이 확인됩니다.
기존에 `NaverDiscountPolicy`, `DanawaDiscountPolicy`를 보면 할인율을 통한 할인이라는 같은 로직을 같는 클래스 였는데,
각각 구현되는 것을 분류를 통해서 `RateDiscounter`로 추상화 시킨 것입니다.

만약 구글 20% 할인 같은 정책이 추가되면 `Discounter` 테이블에 해당 정책을 하나 추가하면 됩니다.
전혀 코드 변경 없이 정책 확장이 가능하게 되는 것입니다. 그리고 해당 할인금액을 조회하는 행위(메소드)도
Entity 안에 있기 때문에 객체지향의 근본인 **연관된 상태와 행위가 가지는 객체**가 되어서 더욱 **응집력**이 높아집니다.

# 결론

- Domain model with JPA 짱짱맨????
- **객체지향적 프로그래밍을 통해서 분기문을 없애는 노력을 기울이면 유연하고 응집력 있는 코드를 얻음**

## 테스트 가능한 예제코드

[https://github.com/redutan/anti-oop.git](https://github.com/redutan/anti-oop.git)
