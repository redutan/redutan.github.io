---
layout: "post"
title: "JsonPath와 Spring Test 간 버전 호환성"
date: "2016-02-28 00:37"
tags:
    - spring
    - jsonpath
---

Spring MVC Test를 위해서 `com.jayway.jsonpath.json-path`(`jsonPath` 메서드) 라이브러리를 사용할 시 아래와 같은 오류가 발생하는 경우가 있습니다.

> java.lang.NoSuchMethodError: com.jayway.jsonpath.JsonPath.compile(Ljava/lang/String;[Lcom/jayway/jsonpath/Filter;)Lcom/jayway/jsonpath/JsonPath;

*상세오류로그는 아래 참조*

확인결과
**JsonPath 라이브러리와 Spring MVC Test와 버전 호환성에 이슈가 있어서 관련 표를 정리해봅니다.**

java버전 `1.8` 기준으로 테스트했습니다.

- 2016-02-27 기준
- json-path `0.8.1` 이상
- spring-test `3.2.0` 이상 (spring-mvc test 최초 릴리즈가 `3.2` 부터임)

| spring-test | json-path | 비고 |
|-----------------|-----------|----|
| `4.2.5.RELEASE` | `0.8.1`, `0.9.0`, `1.0.0`, `1.2.0`, `2.0.0`, `2.1.0` | |
| `4.1.9.RELEASE` | `0.8.1`, `0.9.0`, `1.0.0`, `1.2.0`, `2.0.0`, `2.1.0` | |
| **`4.1.2.RELEASE`** | `0.8.1`, `0.9.0`, `1.0.0`, `1.2.0`, `2.0.0`, `2.1.0` | `4.1.2.RELASE` 이상 모두 성공 |
| `4.1.1.RELEASE` | `0.8.1`, `0.9.0` | |
| `4.1.0.RELEASE` | `0.8.1`, `0.9.0` | |
| `4.0.9.RELEASE` | `0.8.1`, `0.9.0` | |
| `4.0.0.RELEASE` | `0.8.1`, `0.9.0` | |
| `3.2.16.RELEASE` | `0.8.1`, `0.9.0`  | |
| **`3.2.2.RELEASE`** | `0.8.1`, `0.9.0` | `3.2.2.RELEASE`부터 `0.9.0` 이하 성공 |
| `3.2.1.RELEASE` | | 모두 실패 |
| `3.2.0.RELEASE` | | 모두 실패 |

### 테스트코드

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = "classpath:root-context.xml")
public class JsonControllerTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setUp() throws Exception {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac)
                .alwaysDo(print())
                .build();
    }

    @Test
    public void getNameCard() throws Exception {
        this.mockMvc.perform(get("/namecards/{0}", 1L))
                .andExpect(status().isOk())
                .andExpect(content().contentType("application/json;charset=UTF-8"))
                .andExpect(jsonPath("$.seq", is(1)))
                .andExpect(jsonPath("$.name", is("정명주")))
                .andExpect(jsonPath("$.company", is("red")))
                ;
    }
}
{% endhighlight %}

*상세오류로그*
{% highlight java %}
java.lang.NoSuchMethodError: com.jayway.jsonpath.JsonPath.compile(Ljava/lang/String;[Lcom/jayway/jsonpath/Filter;)Lcom/jayway/jsonpath/JsonPath;

	at org.springframework.test.util.JsonPathExpectationsHelper.<init>(JsonPathExpectationsHelper.java:54)
	at org.springframework.test.web.servlet.result.JsonPathResultMatchers.<init>(JsonPathResultMatchers.java:43)
	at org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath(MockMvcResultMatchers.java:146)
	at io.redutan.jsonpath.controller.JsonControllerTest.getNameCard(JsonControllerTest.java:47)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
	at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:74)
	at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:83)
	at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:72)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:231)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:88)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:71)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:174)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:119)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:42)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:234)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:74)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
{% endhighlight %}

*JsonPath Version* [https://github.com/jayway/JsonPath/blob/master/changelog.md](https://github.com/jayway/JsonPath/blob/master/changelog.md)

*Spring Version*
 [http://mvnrepository.com/artifact/org.springframework/spring-core](http://mvnrepository.com/artifact/org.springframework/spring-core)
