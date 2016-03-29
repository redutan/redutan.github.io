---
layout: "post"
title: "Anti-OOP 분기절(if, switch)을 제거하라"
date: "2016-03-29 20:30"
tags:
    - OOP
---

# 쇼핑몰 할인정책

{% highlight java %}
class PaymentService {
    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 결제금액 (할인적용 후)
        long paymentAmt = 0;       
        // 할인구분명
        String discountName = ...;

        // 할인정책 (NONE:할인없음, NAVER:네이버검색-10% 할인, FANCAFE:팬카페-1000원 할인)
        String discountPolicy = ...
        ...
        // 할인정책 분기
        if ("NAVER".equals(discountPolicy)) {   // 네이버검색 할인
            paymentAmt = productAmt * 0.9;
        } else if ("FANCAFE".equals(discountPolicy)) {  // 팬카페 할인
            if (productAmt < 1000)  // 할인쿠폰 금액보다 적은경우
                paymentAmt = 0;
            else
                paymentAmt = productAmt - 1000;
        } else {
            paymentAmt = productAmt;
        }
        ...
        // 할인구분명 분기
        if ("NAVER".equals(discountPolicy)) {   // 네이버검색 할인
            discountName = "네이버할인 10%"
        } else if ("FANCAFE".equals(discountPolicy)) {  // 팬카페 할인
            discountName = "팬카페할인 1000원"
        } else {
            discountName = "할인없음"
        }
    }
}
{% endhighlight %}

# enum을 통한 리팩터링 1

{% highlight java %}
public enum DiscountPolicy {
    /** 할인없음 */
    NONE("할인없음") {
        public long getDiscountedAmt(long originAmt) {
            return originAmt;
        }
    },
    /** 네이버 할인 */
    NAVER("네이버 10% 할인", 10, 0) {
        public long getDiscountedAmt(long originAmt) {
            return originAmt * (100 - discountRate) / 100;
        }
    },
    /** 팬카페 할인 */
    FANCAFE("팬카페 1000원 할인", 0, 1000L) {
        public long getDiscountedAmt(long originAmt) {
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
class PaymentService {
    // 결제처리
    public void payment(...) {
        // 상품금액
        long productAmt = ...;
        // 결제금액 (할인적용 후)
        long paymentAmt = 0;
        // 할인구분명
        String discountName = ...;
        // 할인정책
        DiscountPolicy discountPolicy = DiscountPolicy.valueOf(discountName);
        ...
        // 할인정책(분기없음)
        if ("NAVER".equals(discountPolicy)) {   // 네이버검색 할인
            paymentAmt = productAmt * 0.9;
        } else if ("FANCAFE".equals(discountPolicy)) {  // 팬카페 할인
            if (productAmt < 1000)  // 할인쿠폰 금액보다 적은경우
                paymentAmt = 0;
            else
                paymentAmt = productAmt - 1000;
        } else {
            paymentAmt = productAmt;
        }
        ...
        // 할인구분명 분기
        if ("NAVER".equals(discountPolicy)) {   // 네이버검색 할인
            discountName = "네이버할인 10%"
        } else if ("FANCAFE".equals(discountPolicy)) {  // 팬카페 할인
            discountName = "팬카페할인 1000원"
        } else {
            discountName = "할인없음"
        }
    }
}
{% endhighlight %}

#
