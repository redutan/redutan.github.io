---
layout: "post"
title: Effective Java 2/E - Chatper 10 병행성 (1)
date: "2016-03-11 22:16"
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

# Rule 69 - wait나 notify 대신 병행성 유틸리티를 이용하라
