---
layout: post
title: Effective Java 2/E - Chatper 10 병행성 (1)
date: '2016-03-13 01:39'
tags:
  - book
  - effective-java2
---

# Rule 66 - 변경 가능 공유 데이터에 대한 접근은 동기화하라

**상호 배제성뿐 아니라 스레드 간의 안정적 통신을 위해서도 동기화는 반드시 필요하다.**

`Thread.stop`는 폐기된 API이므로 절대로 사용하지 마라.

**읽기 연산과 쓰기 연산에 전부 적용하지 않으면 동기화는 아무런 효과도 없다.**

## 동기화를 구현하는 방법

### 1. `synchronized`
{% highlight java %}
// 적절히 동기화한 스레드 종료 예제
public class StopThread {
    private static boolean stopRequested;
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while(!stopRequested())
                    i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
{% endhighlight %}

### 2. `volatile`
{% highlight java %}
// 적절히 동기화한 스레드 종료 예제
public class StopThread {
    private static volatile boolean stopRequested;
    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while(!stopRequested)
                    i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
{% endhighlight %}

`volatile`은 상호배제를 구현하지 않지만 가장 최근에 기록된 값을 조회하도록 보장한다.

*example 잘못된 `volatile`*
{% highlight java %}
// 잘못된 예제 - 동기화가 필요하다!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
{% endhighlight %}

문제는 `++`연산자가 **원자적이지 않다는데** 있다. 이 연산자는 두가지 연산을 순서대로 시행한다.

1. 값을 읽는다
2. 새로운 값(+1을 더한 값)을 필드에 쓴다.

첫번째 해결방법은 `synchronized`

두번째 해결방법은 아래 3번째 동기화 구현방법을 참고!!

### 3. Atomic 클래스

*example*
{% highlight java %}
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
{% endhighlight %}

## 동기화의 솔루션의 역발상

**변경 가능 데이터를 공유하지 않는 것이다** - 동시성 이슈가 발생하지 않음

1. **변경 불가능 데이터(불변객체)를 공유**
2. **변경 가능한 데이터는 한 스레드만 이용하도록 한다 (공유하지 않음)**

*실질적으로 변경 불가능한 객체(effectively immutable)*

특정 스레드만이 데이터 객체를 변경할 수 있도록 하고, 변경이 끝난 뒤에야 다른 스레드와 공유하도록 할 때는
객체 참조를 공유하는 부분에만 동기화를 적용하는 객체

## 결론

**변경 가능한 데이터를 공유할 때는 해당 데이터를 읽거나 쓰는 모든 스레드는 동기화를 수행해야 한다**

# Rule 67 - 과도한 동기화는 피하라

동기화 시 발생할 수 있는 문제

1. 교착상태(deadlock)
2. 비결정적 동작(nondeterministic behavior)

**동기화 메서드나 블록 안에서 클라이언트에게 프로그램 제어 흐름을 넘기지 마라**

동기화가 적용된 영역(synchronized) 안에서는 재정의 가능 메서드나 클라이언트가 제공한 함수 객체 메서드를
호출하지 말라는 것 - **불가해(alien) 메서드이기 때문** -
_제어권이 다른 객체로 위임되는데 위임되는 메서드 안에서는 동기화 제어를 할 수 없기 때문이다._

## 불가해 메서드 오류와 해결

*example 불가해 메서드 동기화 오류*
{% highlight java %}
private void notifyElementAdded(E element) {
    synchronized(observers) {
        for (SetObserver<E> observer : observers) {
            observer.added(this, element);
        }
    }
}
{% endhighlight %}

*example 솔루션 1 : 방어복사 + open call*
{% highlight java %}
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<SetObserver<E>>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
{% endhighlight %}

*example 솔루션 2 : Concurrent Collection*
{% highlight java %}
// 다중 스레드에 안전한 구독자 집합 : CopyOnWriteArrayList 이용
private final List<SetObserver<E>> observers =
        new CopyOnWriteArrayList<SetObserver<E>>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}
public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}
private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers) {
        observer.added(this, element);
    }
}
{% endhighlight %}

## 핵심

**동기화 영역 안에서 수행되는 작업의 양을 가능한 줄여야 한다.**

동기화의 진짜 비용은 **병렬성을 활용할 기회를 잃는다는 것** 이다.

**static 필드(또는 단일객체 내 필드)를 변경하는 메서드가 있을 때는 해당 필드에 대한 접근을 반드시 동기화
해야 한다.**

## 결론

- **동기화 영역 안에서 불가해(alien) 메서드를 호출하지 않는다.**
- **동기화 영역 안에서 작업을 최소화 한다.**
- **가변 클래스 내 동기화가 필요한지 검토하고 필요하다면 동기화 한다.**

# Rule 68 - 스레드보다는 실행자와 태스크를 이용하라

## 실행자 프레임워크(Executor Framework)

`Task` 실행 프레임워크

*example*
{% highlight java %}
// 실행자 생성
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(runnable); // 실행
executor.shutdown();    // 셧다운
{% endhighlight %}

**추가기능**

1. 특정 태스크가 종료되길 대기
2. 임의의 태스크들이 종료되기를 대기 : `invokeAny`, `invokeAll`
3. 실행자 서비스가 자연스럽게 종료될 수 있도록 대기 : `awaitTermination`
4. 태스크가 끝날 때 마다 결과 조회 : `ExecutorCompletionService`

## Thread pool

멀티쓰레드를 사용하기 위한 풀

`java.util.concurrent.Executors.*`
`ThreadPoolExecutor`

- 일반적인 상황 : `Executors.newCachedThreadPool`
- 부하가 큰 상황 : `Executors.newFixedThreadPool`

## 결론

중요한 것은 `Thread`가 아니다. **작업과 실행 메커니즘이 분리된 것이다.**

### 작업

**중요한 것은 작업이다.**

*종류*

- `Runnable`
- `Callable` : `Runnable`과 다르게 반환값이 존재한다.

### 실행 메커니즘

**실행 메커니즘은 실행자 서비스 이다.** 그리고 우리는 이것을 자바API가 제공해주는 *실행자 프레임워크*를
통해서 사용하면된다.

*추가 tip*

`java.util.Timer` 대신 `ScheduleThreadPoolExecutor`을 추천

- 추상화 수준이 높아서 사용하기 편리하고 유연성이 높음
- 타이밍이 정확함
- 멀티스레드 이용 가능
- 무점검 예외 복구 : 예외를 핸들링 할 필요가 없음

# Rule 69 - wait나 notify 대신 병행성 유틸리티를 이용하라

과거처럼 `wait`나 `notify`를 직접 구현하지 말고, 자바 플랫폼(1.5이상)이 제공하는 **고수준
병행성 유틸리티**(high-level concurrency utility)를 이용하라.

**병행성 유틸리티 분류**

- 실행자 프레임워크 : Rule 67
- 병행 컬렉션(concurrent collection) : 이번 규칙
- 동기자(synchronizer) : 이번 규칙

## 병행 컬랙션

표준 Collection 인터페이스(ex:`List`, `Queue`, `Map`)에 고성능 병행 컬렉션 구현을 제공하며,
**병행성을 높이기 위해 동기화를 내부적으로 처리**

*그래서 컬렉션 외부에서 병행성을 처리하는 것이 불가능.* 락을 걸어봐야 락 중복으로 인해 성능만 나빠짐

*위 문제를 해결하기 위해서 상태 종속 변경 연산을 제공.* : 몇가지 기본 연산들을 하나의 원자적 연산으로 묶은 것

예를들면 `ConcurrentMap.putIfAbsent(K, V)` : 키에 해당하는 키에 해당하는 값이 없을 때문 주어진
값을 집어 넣고, 해당 키에 대응하여 저장되어 있었던 기존 값을 반환. 값이 없으면 null 반환

*example*
{% highlight java %}
// ConcurrentMap으로 구현한 병행 정규화 맵 : 최적이 아님
private static final ConcurrentMap<String, Stirng> map =
        new ConcurrentHashMap<String, String>();

public static String intern(String s) {
    String previousValue = map.putIfAbsent(s, s);
    return previousValue = null ? s : previousValue;
}
{% endhighlight %}

*example 개선*
{% highlight java %}
// ConcurrentMap으로 구현한 병행 정규화 맵 - 더 빠르다!
public static String intern(String s) {
    String result = map.get(s);
    if (null == null) {
        result = map.putInAbsent(s, s);
        if (result == null)
            result = s;
    }
    return result;
}
{% endhighlight %}


### 추천 클래스

#### `ConcurrentHashMap`

#### `CopyOnWriteArrayList`

#### `BlockingQueue` : 봉쇄 연상(blocking operaton : `task`) 가능

`task`는 큐의 맨 앞(head) 원소를 제거한 다음 반환하는데, 큐가 비어있는 경우에는 대기(wait)

*생산자-소비자큐 라고 불리기도 함*

생산자 스레드가 큐에 작업들일 집어 넣고, 소비자 스레드가 꺼내어 처리

`ThreadPoolExecutor`을 비롯한 대부분의 `ExecutorService` 구현은 `BlockingQueue` 를 사용

## 동기자

쓰레드들의 병행성을 제어해서 서로 상호협력이 가능하게 함

### `CountDownLatch`

일회성 배리어(barrier)로서 하나 이상의 스레드가 작업을 마칠 때까지 다른 여러 스레드가 대기하게 한다.
대기 중인 스레드가 진행할 수 있으려면 그 횟수만큼 `countdown` 메서드가 호출되어야 한다.

*example*
{% highlight java %}
// 작업의 병령 수행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency,
        final Runnable action) throws InterruptedException {
    // 작업 준비
    final CountDownLatch ready = new CountDownLatch(concurrency);
    // 작업 시작(타이머)
    final CountDownLatch start = new CountDownLatch(1);
    // 작업 완료
    final CountDownLatch done = new CountDownLatch(concurrency);
    for (int i = 0; i < concurrency; i++) {
        executor.execute(new Runnable() {
            public void run() {
                ready.countDown();  // 타이머에게 준비됨을 알림
                try {
                    start.await();  // 다른 작업스레드가 준비될 때까지 대기
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();   // 타이머에게 끝났음을 알림
                }
            }
        })
    }
    ready.await();      // 모든 작업 스레드가 준비될 때까지 대기
    long startNanos = System.nanoTime();
    start.countDown();  // 출발
    done.await();       // 모든 작업 스레드가 끝날 때까지 대기
    return System.nanoTime() - startNanos;
}
{% endhighlight %}

작업(`Runnable` 인자)을 병렬로 실행할 횟수(concurrency) 만큼 돌려서 실행되는 시간을 구하는 메서드

1. 작업을 쓰레드에 등록을 해서
2. 전체 작업 쓰레드가 준비를 대기한 다음
3. 시작시간을 재고
4. 전체 작업이 시작되고
5. 전체 작업이 마감이 된 후
6. 진행된 시간을 계산해서 반환

**주의사항**

- 실행자(`Executor`인자)가 주어진 병행수(concurrency) 보다 같거나 많은 수의 스레드를 동시에 생성할
수 있어야함
- 시간을 잴 때는 `System.currentTimeMillis` 대신 `System.nanoTime`를 사용한다.
    - 더 정밀하고, 시스템의 실시간 클락의 변동에도 영향을 받지 않는다.

### `Samaphore`

### 기타

`CycleBarrier`, `Exchanger`

### 레거시코드 (`wait`, `notify`)

*example 표준숙어*

{% highlight java %}
// wait 메서드를 사용하는 표준숙어
synchronized (obj) {
    while (실패조건)
        obj.wait(); // 락을 해제하고 대기풀(wating pool) 에서 락 획득 대기
    // 락을 획득(스레드를 깨움) 후 해야할 실제작업
}
{% endhighlight %}

*while 문으로 실패조건을 확인하는 이유?*

if로 체크할 경우 wait 이후에 다시 락을 획득 했을 시 실패 조건 재검증이 안되서 일관성이 깨질 수 있음

*락을 다시 획득하는(스레드를 다시 깨울) 경우 가능하면 `notify` 대신 `notifyAll`을 사용하라*

- JVM이 알아서 락을 획득할 스레드의 우선순위를 적절하게 정해줘서 영원히 잠드는(대기상태인) 스레드가 없을 것이다.
- 악의적으로 또는 객체의 API 노출 등의 실수로 `notify`가 호출되는 경우 무한 대기가 걸릴 수 있음.

## 결론

**자바API가 신규로 제공하는 고수준의 병행성 유틸리티를 사용하라.** notify, wait를 사용할 이유가 없다.

만약 기존 코드를 유지보수 해야하는 상황이라면 위의 표준숙어를 이용해서 안전하게 처리하고, 일반적으로
`notify` 대신 `notifyAll`을 사용하도록 하라

> 더 자세한 내용은 `Java Concurrency in Practice` 책을 잠조하라.
