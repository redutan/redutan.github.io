---
layout: post
title: JSTL Custom tag에 Spring Bean 주입하기
date: '2015-11-25 20:58'
tags:
  - spring
  - spring-mvc
  - jstl
---

어떻게든 `ApplicationContext`만 가져오면 되는데 이미 SpringMVC에서 다 제공을 해주더군요

1. `RequestContextAwareTag`을 상속한다.
    - 재미있는 부분은 해당 Tag클래스를 Spring Tag Library 구현체 모두 상속받아서 사용한다는 점입니다. 당연한 이야기 겠네요.
    - 예를 들면 `MessageTag`도 위 클래스를 상속받는데 jsp tag로 보면 `<spring:message>` 입니다.
2. `WebApplicationContext`를 얻는다.
3. 스프링 빈 주입

예제코드를 확인해봅시다.

{% highlight java %}
// 1. RequestContextAwareTag 상속
public class CodeTag extends RequestContextAwareTag {
    @Autowired
    CodeService codeService;

...
    @Override
    protected int doStartTagInternal() throws Exception {
        if (codeService == null) {
            // 2. WebApplicationContext를 얻는다.
            WebApplicationContext wac = getRequestContext().getWebApplicationContext();
            AutowireCapableBeanFactory beanFactory = wac.getAutowireCapableBeanFactory();
            // 3. 스프링 빈 주입
            beanFactory.autowireBean(this);
        }
        // TODO working
        return SKIP_BODY;
    }
}
{% endhighlight %}

**참 쉽죠 ~**

> 참조 : http://stackoverflow.com/questions/3445908/is-there-an-elegant-way-to-inject-a-spring-managed-bean-into-a-java-custom-simpl
