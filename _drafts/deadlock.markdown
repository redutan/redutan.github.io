---
layout: "post"
title: "DeadLock"
date: "2016-06-29 11:44"
tag:
---

**정의**

프로세스들이 더 이상 진행하지 못하고 영구적으로 block된 상태

_예를들면 두 사람이 서로의 멱살을 잡으면서 서로 내 멱살이 필요하니 놓으라는 상태_

# 데드락 발생 4조건

1. 상호배제(Mutual exclusion)
2. 잠금과 대기(Hold & Wait)
3. 선점 불가(No preemption)
4. 순환 대기(Circular wait)

위 4가지 조건을 모두 만족해야 데드락이 발생한다.

# 데드락 발생조건 상세

각 조건별로 상세 내역을 알아보자.

## 상호 배제(Mutual exclusion)

한 번에 오직 한 개의 프로세스만이 자원에 접근 가능

_예를들면 데이터베이스 연결 풀, 쓰기용 파일 열기, 레코드 락, 세마포어_

만약 프로세스의 수보다 자원의 수가 더 많으면 실질적으로 상호 배제가 깨진다고 볼 수 있다.
자원을 접근함에 있어서 제약이 없기 때문이다.

하지만 대부분의 경우 접근할 자원이 부족한 경우가 많다.

## 잠금과 대기(Hold & Wait)

한 개 이상의 자원을 가진(Hold) 프로세스가 다른 프로세스 소유의 자원을 기다리는 것(Wait)

## 선점 불가(No preemption)

한 프로세스가 다른 프로세스로 부터 자원을 빼앗지 못하는 것

## 순환 대기(Circular wait)

**프로세스 연쇄적(환형)으로 자원을 대기하는 상태**

예를 들면

`P1`, `P2` : 프로세스

`R1`, `R2` : 자원

`P1`은 `R1`을 점유하고 있고 `P2`는 `R2`을 점유하고 있는 상태에서 `P1`은 `R2`가 필요하고, `P2`는 `R1`이 필요한 상태

![순환 대기](/images/2016/06/Deadlock_diagram.png)

*출처 [http://beginnersbook.com/2015/04/deadlock-in-dbms/](http://beginnersbook.com/2015/04/deadlock-in-dbms/)*

이것이 환형이기 때문에 프로세서와 리소스의 수는 각각 N개 이상일 수 있다.

# 데드락을 깨자

## 데드락 무시

이 방법은 해당 어플리케이션이 데드락이 발생하지 않는다고 가정하 하는 것이다. 즉, **실제로는 데드락은 무시하면 안된다.**
데드락 발생을 정상적인 동작이 아니라고 판단하는 것이고 최후의 수단으로 두는 방식

Ostrich algorithm : 데드락이 발생하면 시스템 재시작

## 데드락 감지

프로세스를 모니터링해서 데드락을 유발하는 프로세스를 감지한다. 그 후

1. 해당 프로세스를 죽인다.
2. 데드락 상태였던 프로세스가 죽고난 자원을 풀어서 다른 프로세스에서 선점할 수 있게 열어둔다.

## 데드락 예방

### 상호 배제 조건 깨기

**상호 배제 조건 자체를 비켜가기**

- 자원을 동시 사용가능하게 만듬 : ex) `AtomicInteger`
- 프로세스 수보다 자원의 수를 더 늘림 : 상호 배제할 필요가 없어짐

하지만 대부분의 경우 자원은 제한적이다. 상호 배제 조건을 깨기는 매우 힘든 경우가 많음

### 잠금과 대기 조건 깨기

**대기가 발생하지 않도록 해서 비켜가기**

각 자원을 점유하기 전에 필요한 자원들이 다 확보 가능한지 확인한다. 만약 단 하나의 자원이라도 점유하지 못한다면 지금까지 점유한 자원을 몽땅 내놓고 처음부터 다시 시작한다.

**단점**

- 기아(Starvation) :
- 라이브락(Livelock) :


### 선점 불가 조건 깨기

### 순환 대기 조건 깨기

**참고**

- 클린코드 : 로버트.C.마틴
- [https://en.wikipedia.org/wiki/Deadlock](https://en.wikipedia.org/wiki/Deadlock)
- [https://ko.wikipedia.org/wiki/%EA%B5%90%EC%B0%A9_%EC%83%81%ED%83%9C](https://ko.wikipedia.org/wiki/%EA%B5%90%EC%B0%A9_%EC%83%81%ED%83%9C)
- [http://beginnersbook.com/2015/04/deadlock-in-dbms/](http://beginnersbook.com/2015/04/deadlock-in-dbms/)
- [http://marsland.tistory.com/149](http://marsland.tistory.com/149)
- [http://wonjayk.tistory.com/251](http://wonjayk.tistory.com/251)
