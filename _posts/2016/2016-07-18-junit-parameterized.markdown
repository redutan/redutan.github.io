---
layout: post
title: JUnit-Parameterized
date: '2016-07-18 16:45'
tags:
  - junit
---

# 설명

단위테스트를 진행하다 보면 여러가지 입력 값에 대한 테스트를 한 번에 수행할 필요가 있습니다.

예를 들면 추천 수가 100개 이상일 시, 월 로그인 횟수가 30회 이상 일 시, 금액이 10만원 이상일 시, 법인회원일 시 이러한 조건에 따라서 요구사항이 달라지는 경우가 많습니다.

아래 상세하게 예를 들어서 추천 수가

- 9개 미만이면 일반게시물 - 경계조건 추천 9
- 10개 이상이면 베스트 게시물 - 경계조건 추천 10
- 100개 이면 베스트게시물로 분류 - 경계조건 추천 100(99)

속성 상태가 로직의 변화를 일으키는 **경계조건** 을 테스트해야하는 경우가 있습니다. 특별히 경계조건을 테스트할 시 더 유리하지만 그것 뿐만 아니라 단순히 여러가지 값을 검증할 시에도 유용한 **Parameterized Test** 를 JUnit 예제로 알아보겠습니다.

# 예제

결제금액 별로 취소수수료 다른 조건을 Parameterized Test로 풀어보는 예제입니다.

## 예제조건

**가격 별 취소수수료**

- 0 ~ 9999 : 0%
- 10000 ~ 49999 : 10%
- 50000 ~ : 20%

## 예제코드

{% highlight java %}
/**
 * 가격 별 취소수수료
 * 0 ~ 9999 : 0%
 * 10000 ~ 49999 : 10%
 * 50000 ~ : 20%
 */
@RunWith(Parameterized.class)   // 파라메터화된 테스트를 위한 선언
public class RefundServiceTest {

    // 파라미터들 제공 메소드 : static 이면서 Collection을 반환해야한다. 경계영역이 잘 설정되어야함.
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][]{
                { 9999L,     0L},
                {10000L,  1000L},
                {49999L,  4999L},
                {50000L, 10000L}
        });
    }

    // @Parameter로 주입 시 public 으로 선언되어야 한다.
    @Parameter(0)   // data() 항 항목의 첫번째 인자
    public long amount;
    @Parameter(1)   // data() 항 항목의 두번째 인자
    public long refundFee;

    Order order;
    RefundService refundService;

    @Before
    public void setUp() throws Exception {
        refundService = new RefundService();
        order = new Order(amount);
    }

    @Test
    public void testGetRefundFee() throws Exception {
        assertThat(refundService.getRefundFee(order), is(this.refundFee));
    }
}

class Order {
    private long amount;

    public Order(long amount) {
        this.amount = amount;
    }

    public long getAmount() {
        return this.amount;
    }
}

class RefundService {
    public long getRefundFee(Order order) {
        long amount = order.getAmount();
        if (amount < 10000L)    // 0%
            return 0;
        else if (amount < 50000L)   // 10%
            return amount * 10 / 100;
        else    // 20%
            return amount * 20 / 100;
    }
}
{% endhighlight %}

## Point

- `@RunWith(Parameterized.class)` : 파라메터화된 테스트 클래스 선언
- `@Parameters` : 파라미터들 제공 메소드 어노테이션 : static 이면서 Collection을 반환해야한다. **경계영역이 잘 설정되어야함.**
- `@Parameter` : `@Parameters` 가 반환하는 항목 인자. 꼭 `public` 으로 선언.
    - `@Parameter(0)` 은 첫번째 인자, `@Parameter(1)` 은 두번째 인자... (0 base)
    - `@Parameter` 대신에 인자 순서대로 생성자로 받아도 된다.
{% highlight java %}
long amount;
long refundFee;

RefundServiceTest(long amount, long refundFee) {
    this.amount = amount;
    this.refundFee = refundFee;
}
{% endhighlight %}


## 추가사항

`@Parameters` 즉 여러가지 입력 값을 반환하는 메소드는 위와 같이 간단한 경우에는 직접 값을 코드에 입력 가능하나, 입력 값이 매우 복잡한 경우에는 **json** 이나 **excel** 또는 헬퍼객체를 통해서 입력 받는 것을 추천합니다.

# 참조

- https://github.com/junit-team/junit4/wiki/Parameterized-tests
- [테스트주도개발 실천법과 도구[채수원]](http://www.hanbit.co.kr/store/books/look.php?p_code=B3818551654)
