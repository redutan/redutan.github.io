---
layout: "post"
title: Effective Java 2/E - Chatper 09 예외 (2)
date: "2016-03-11 00:33"
tags:
  - book
  - effective-java2
---

# Rule 61 - 추상화 수준에 맞는 예외를 던져라

**상위 계층 추상화에 맞는 예외로 변환해서 던져야 한다.**

_example_

{% highlight java %}
// 예외 변환 (exception translation)
try {
    // 낮은 수준의 추상화 계층
    ...
} catch(LowerLevelException e) {
    throw new HighLevelException(...);
}
{% endhighlight %}

### Exception chaining(예외 연결)

{% highlight %}
// 예외 연결
try {
    // 낮은 수준의 추상화 계층
    ...
} catch(LowerLevelException cause) {
    throw new HighLevelException(cause);
}
{% endhighlight %}

예외 스택을 통해서 하위 예외를 추적할 수 있다.

### 주의사항

**남용하면 안 된다.**

가능하면 하위계층에서 예외가 생기지 않도록 하는 것이다.

- 하위계층 호출 시 인자 유효성 강화
- 치명적인 상황(경고수준)이면 상위 계층에서 하위계층 예외를 격리시킨다. **이 때 로깅을 꼭 남기도록 한다.**
    - 추후 해당 오류를 분석해야함

# Rule 62 - 메서드에서 던져지는 모든 예외에 대해 문서를 남겨라

모든 예외를 javadoc `@throws`를 통해서 명확하게 기술한다.

**일반적인 예외를 던지는 것은 금한다.**

- ex) `throws Exception`, `throws Throwable`

무점검 예외를 통해서 메서드를 성공적으로 호출하기 위한 선행 조건을 알 수 있다.

_`throws`를 이용할 시에는 점검예외만 사용하자. 무점검 예외는 javadoc 문서만으로도 충분하다._

- 점검지정예외와 무점검지정예외를 구분하는 일반적인 규약이다.

**동일한 예외(ex:`NullPointerException`)를 던지는 메서드가 많다면 메서드 마다문서를 만들지 않고,
햬당 예외 클래스에 문서화 해도 된다.**

# Rule 63 - 어떤 오류인지를 드러내는 정보를 상세한 메시지에 담으라




# Rule 63 - 어떤 오류인지를 드러내는 정보를 상세한 메시지에 담으라

# Rule 64 - 실패 원자성 달성을 위해 노력하라

# Rule 65 - 예외를 무시하지 마라
