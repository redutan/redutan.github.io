---
layout: "post"
title: "Anti-OOP : 분기절(if, switch)을 제거하라"
date: "2016-03-29 20:30"
tags:
    - OOP
---

# 쇼핑몰 할인정책

{% highlight java %}
public class PaymentService {
    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 할인정책 (NONE:할인없음, NAVER:네이버검색-10% 할인, FANCAFE:팬카페-1000원 할인)
        String discountCode = ...
        ...
        // 결제금액
        long paymentAmt = 0;
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            paymentAmt = productAmt * 0.9;
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페 할인
            if (productAmt < 1000)  // 할인쿠폰 금액보다 적은경우
                paymentAmt = 0;
            else
                paymentAmt = productAmt - 1000;
        } else {
            paymentAmt = productAmt;
        }
        ...
        // 할인구분명 분기
        String discountName;
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            discountName = "네이버할인 10%"
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페 할인
            discountName = "팬카페할인 1000원"
        } else {
            discountName = "할인없음"
        }
    }
}
{% endhighlight %}

# enum을 통한 리팩터링

{% highlight java %}
public enum DiscountPolicy {
    /** 할인없음 */
    NONE("할인없음") {
        @Override
        public long getDiscountedAmt(long originAmt) {
            return originAmt;
        }
    },
    /** 네이버 할인 */
    NAVER("네이버 10% 할인", 10, 0) {
        @Override
        public long getDiscountedAmt(long originAmt) {
            return originAmt * (100 - discountRate) / 100;
        }
    },
    /** 팬카페 할인 */
    FANCAFE("팬카페 1000원 할인", 0, 1000L) {
        @Override
        public long getDiscountedAmt(long originAmt) {
            if (originAmt < discountAmt)
                return 0;
            return originAmt -= discountAmt;
        }
    }
    ;
    private final Stirng discountName;
    private final int discountRate;
    private final long discountAmt;

    DiscountPolicy(String discountName) {
        this(discountName, 0, 0L);
    }

    DiscountPolicy(String discountName, int discountRate, int discountAmt) {
        this.discountName = discountName;
        this.discountRate = discountRate;
        this.discountAmt = discountAmt;
    }

    public String getDiscountName() {
        return this.discountName;
    }

    /**
     * 할인 후 금액 반환
     */
    abstract long getDiscountedAmt(long originAmt);
}
{% endhighlight %}

{% highlight java %}
public class PaymentService {
    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NONE:할인없음, NAVER:네이버검색-10% 할인, FANCAFE:팬카페-1000원 할인)
        String discountCode = ...
        // 할인정책
        DiscountPolicy discountPolicy = DiscountPolicy.valueOf(discountCode);
        ...
        // 결제금액
        long paymentAmt = discountPolicy.getDiscountedAmt(productAmt);
        ...
        // 할인구분명 분기
        String discountName = discountPolicy.getDiscountName();
    }
}
{% endhighlight %}

# 전략패턴을 통한 리팩터링

{% highlight java %}
public interface Discountable {
    long getDiscountedAmt(long originAmt);
}

public interface DiscountPolicy extends Discountable {
    public static final DiscountPolicy NONE = new DiscountPolicy() {
        @Override
        public long getDiscountedAmt(long originAmt) {
            return originAmt;
        }

        @Override
        public String getDiscountName() {
            return "할인없음";
        }
    };

    String getDiscountName();
}

abstract class AbstractDiscountPolicy implements Discountable {
    private final String discountName;

    AbstractDiscountPolicy(String discountName) {
        this.discountName = discountName;
    }

    @Override
    public String getDiscountName() {
        return this.discountName;
    }
}

class NaverDiscountPolicy extends AbstractDiscountPolicy {
    static final int DEFAULT_RATE = 10;
    private int discountRate;

    NaverDiscountPolicy() {
        this(DEFAULT_RATE);
    }

    NaverDiscountPolicy(int discountRate) {
        super("네이버 "+ discountRate +"% 할인");
        this.discountRate = discountRate;
    }

    @Override
    public long getDiscountedAmt(long originAmt) {
        return originAmt * (100 - discountRate) / 100;
    }
}

class FancafeDiscountPolicy extends AbstractDiscountPolicy {
    static final long DEFAULT_AMT = 1000L;
    private int discountAmt;

    FancafeDiscountPolicy() {
        this(DEFAULT_AMT);
    }

    FancafeDiscountPolicy(int discountAmt) {
        super("팬카페 "+ discountAmt +"% 할인");
        this.discountAmt = discountAmt;
    }

    @Override
    public long getDiscountedAmt(long originAmt) {
        if (originAmt < discountAmt)
            return 0;
        return originAmt -= discountAmt;
    }
}
{% endhighlight %}

{% highlight java %}
public class PaymentService {
    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 할인코드 (NONE:할인없음, NAVER:네이버검색-10% 할인, FANCAFE:팬카페-1000원 할인)
        String discountCode = ...
        // 할인정책
        DiscountPolicy discountPolicy = getDiscountPolicy(discountCode);
        ...
        // 결제금액
        long paymentAmt = discountPolicy.getDiscountedAmt(productAmt);
        ...
        // 할인구분명 분기
        String discountName = discountPolicy.getDiscountName();
    }

    private DiscountPolicy getDiscountPolicy(String discountCode) {
        if ("NAVER".equals(discountCode)) {   // 네이버검색 할인
            return new NaverDiscountPolicy();
        } else if ("FANCAFE".equals(discountCode)) {  // 팬카페 할인
            return new FancafeDiscountPolicy();
        } else {
            return DiscountPolicy.NONE;
        }
    }
}
{% endhighlight %}
