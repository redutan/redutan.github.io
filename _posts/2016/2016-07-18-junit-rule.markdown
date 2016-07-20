---
layout: post
title: JUnit-Rule
date: '2016-07-18 18:09'
tags:
  - junit
---

Rule은 하나의 테스트 클래스 내에서 동작 방식을 재정의 하거나 추가하기 위해 사용됩니다. 특히 다른 테스트클래스에서도 재사용성할 때 유용합니다.

**기본 Rule 클래스**

| 규칙이름 | 설명 |
|---------|------|
| `TemporaryFolder` | 임시폴더 관리. 테스트 후 삭제 |
| `ExternalResources` | 자원(DB, 파일, 소켓) 관리 |
| `ErrorCollector` | 지속적 테스트 실패 수집 |
| `Verifier` | 별개 조건 확인 (vs `assert*`)|
| `TestWatcher` | 테스트 인터셉터 (starting, succeeded, failed, finished...) |
| `TestName` | 테스트 메소드명을 알려줌 |
| `Timeout` | 테스트 클래스 전역 timeout 설정 (vs `@Timeout`) |
| `ExpectedException` | 예외 직접 확인 (vs `@Expected`) |
| `DisableOnDebug` | Rule 디버그 비활성화 데코레이터 |
| `RuleChain` | 복수 Rule chaining 복합체 |
| `ClassRule` | 테스트슈트 전체에 Rule 적용 |

**상속구조**

![](/images/2016/07/junit-rule-inherit.png)

## `TemporaryFolder`

{% highlight java %}
public static class HasTempFolder {
  @Rule
  public TemporaryFolder folder = new TemporaryFolder();

  @Test
  public void testUsingTempFolder() throws IOException {
    File createdFile = folder.newFile("myfile.txt");
    File createdFolder = folder.newFolder("subfolder");
    // ...
  }
}
{% endhighlight %}

## `ExternalResources`

외부 자원(파일, 소켓, DB커넥션 등) 관리. `@Before`, `@After`과 별 차이가 없으나, 특정 자원이 다른 테스트케이스에서 재사용성이 요구될 경우 유용함

{% highlight java %}
public static class UsesExternalResource {
  Server myServer = new Server();

  @Rule
  public ExternalResource resource = new ExternalResource() {
    @Override
    protected void before() throws Throwable {
      myServer.connect();
    };

    @Override
    protected void after() {
      myServer.disconnect();
    };
  };

  @Test
  public void testFoo() {
    new Client().run(myServer);
  }
}
{% endhighlight %}

## `ErrorCollector`

오류를 계속 쌓아둘 수 있으며, 오류가 발생해도 테스트를 계속 진행시킬 수 있을 때 유용하다.

- `checkThat`
- `checkSucceeds`
- `addError`

{% highlight java %}
public static class UsesErrorCollectorTwice {
  @Rule
  public ErrorCollector collector = new ErrorCollector();

  @Test
  public void example() {
    collector.addError(new Throwable("first thing went wrong"));
    collector.addError(new Throwable("second thing went wrong"));
  }

  @Test
  public void testGetAmount() throws Exception {
    Order order1 = ...
    collector.checkThat(10000, order1.getAmount());  // 오류가 발생해도 아래 줄 실행됨
    Order order2 = ...
    collector.checkThat(20000, order2.getAmount());
  }
}
{% endhighlight %}

## `Verifier`

테스트 자체를 검증하는 `assert*` 메소드와는 달리, 일반적으로 테스트케이스 실행 후 만족해야하는 환경조건이나 Global조건(객체들의 종합 상태)을 검사하는데 쓰인다.

{% highlight java %}
private static String sequence;

public static class UsesVerifier {
  @Rule
  public Verifier collector = new Verifier() {
    @Override
    protected void verify() {
      sequence += "verify ";
    }
  };

  @Test
  public void example() {
    sequence += "test ";
  }

  @Test
  public void verifierRunsAfterTest() {
    sequence = "";
    assertThat(testResult(UsesVerifier.class), isSuccessful());
    assertEquals("test verify ", sequence);
  }
}
{% endhighlight %}

## `TestWatcher`

- `apply`
- `succeeded`
- `failed`
- `skipped`
- `starting`
- `finished`

## `TestName`

그냥 테스트 이름 호출할 뿐

{% highlight java %}
public class NameRuleTest {
  @Rule
  public TestName name = new TestName();

  @Test
  public void testA() {
    assertEquals("testA", name.getMethodName());
  }
}
{% endhighlight %}

## `Timeout`

테스트 클래스 전역 timeout 설정

{% highlight java %}
public static class HasGlobalTimeout {
  public static String log;

  @Rule
  public TestRule globalTimeout = new Timeout(20);

  @Test
  public void testInfiniteLoop1() {
    log += "ran1";
    for(;;) {}
  }

  @Test
  public void testInfiniteLoop2() {
    log += "ran2";
    for(;;) {}
  }
}
{% endhighlight %}

## `ExpectedException`

`@Expected`는 단순하게 클래스 타입만 검증하면 메세지도 검증가능하며, 확장하면 코드성 Exception 검증도 가능

{% highlight java %}
public static class HasExpectedException {
  @Rule
  public ExpectedException thrown = ExpectedException.none();

  @Test
  public void throwsNothing() {
  }

  @Test
  public void throwsNullPointerException() {
    thrown.expect(NullPointerException.class);
    throw new NullPointerException();
  }

  @Test
  public void throwsNullPointerExceptionWithMessage() {
    thrown.expect(NullPointerException.class);
    thrown.expectMessage("happened?");
    thrown.expectMessage(startsWith("What"));
    throw new NullPointerException("What happened?");
  }
}
{% endhighlight %}

## `DisableOnDebug`

`-Xdebug` 또는 `-agentlib:jdwp` 옵션 시 비활성화

{% highlight java %}
@Rule
public TestRule timeout = new DisableOnDebug(new Timeout(20));
{% endhighlight %}

## `RuleChain`

말 그대로 Rule chaining

{% highlight java %}
public static class UseRuleChain {
    @Rule
    public TestRule chain = RuleChain
                           .outerRule(new LoggingRule("outer rule"))
                           .around(new LoggingRule("middle rule"))
                           .around(new LoggingRule("inner rule"));

    @Test
    public void example() {
        assertTrue(true);
    }
}
{% endhighlight %}

## `ClassRule`

{% highlight java %}
@RunWith(Suite.class)
@SuiteClasses({A.class, B.class, C.class})
public class UsesExternalResource {
  public static Server myServer = new Server();

  @ClassRule
  public static ExternalResource resource = new ExternalResource() {
    @Override
    protected void before() throws Throwable {
      myServer.connect();
    };

    @Override
    protected void after() {
      myServer.disconnect();
    };
  };
}
{% endhighlight %}

# 참고로 커스텀 룰 구현도 가능합니다.

[https://github.com/junit-team/junit4/wiki/Rules#custom-rules](https://github.com/junit-team/junit4/wiki/Rules#custom-rules)

# 참조

- [https://github.com/junit-team/junit4/wiki/Rules](https://github.com/junit-team/junit4/wiki/Rules)
- [테스트주도개발 실천법과 도구[채수원]](http://www.hanbit.co.kr/store/books/look.php?p_code=B3818551654)
