---
layout: "post"
title: 클린소프트웨어 Part 3. 급여관리 사례 연구
date: "2017-06-26 20:19"
tags:
  - books
  - clean-software
---

# 커맨드 패턴

![커맨드 패턴](https://dooray.com/plantuml/img/oymhIIrAIqnELN3EpyrDp4jHgEPI00Bjb7mDJQvQBW00)

{% highlight java %}
public inteface Command {
  void do();
}
{% endhighlight %}

## 단순한 커맨드 적용

> 커맨더 패턴은 명령의 개념을 캡슐화 함으로써 의존성을 없앤다.

* 센서와 동작의 연결

![Sensor가 주도하는 Command](https://dooray.com/plantuml/img/oymhIIrAIqnELN3EpyrDp4lXIiv9B2vM24xDAyulue9G2hfsS6a0)

## 트랜잭션

트랜잭션도 일종의 명령이므로 이를 커맨드 형식으로 캡슐화할 수 있다.

### 물리적, 시간적 분리

관심사 분리

## 되돌리기

![커맨드 패턴의 undo 변형](https://dooray.com/plantuml/img/oymhIIrAIqnELN3EpyrDp4jHgEPI00Bjb7mDJGYhD0_ChkK20000)

1. 명령스택이 존재한다
2. 어떤 명령을 실행한다.(do)
3. 되돌리고 싶을 시 명령스택에서 꺼낸 후 되돌린다.(undo)

*명령을 취소하는 방법은 명령을 수행하는 방법을 아는 코드와 함께 있어야한다.*

# 액티브 오브젝트 패턴

* 커맨드 패턴의 응용
* `ActiveObjectEngine`는 `Command`객체의 연결리스트
* 엔진에 새로운 명령을 추가할 수도 있고
* `run()`을 호출할 수도 있다. - 루프를 돌면서 명령들을 실행

{% highlight java %}
class ActiveObjectEngine {
  LinkedList<Command> commands = new LinkedList<>();

  public void addCommand(Command c) {
    commands.add(c);
  }

  public void run() throws Exception {
    while (!commands.isEmpty()) {
      Command c = commands.getFirst();
      commands.removeFirst();
      c.execute();
    }
  }
}
interface Command {
  void execute() throws Exception;
}
{% endhighlight %}

멀티스레드 상황에서 스레드 간 협력이 필요할 시 non-block 기반 처리가 가능하게 하는 패턴

**재미있는 것은 말 그대로 명령이 계속 활성화된 상태로 진행된다는 점이다**

# 템플릿 메서드 패턴

![템플릿 메소드 패턴 클래스 다이어그램](https://dooray.com/plantuml/img/IqmgBYbAJ2vHICv9B2vMS8HodS6yQYu58D0kISqjo4aiIVLDBSd8Jz7G18ig5vScmGLgkI3QdVFpaejIIr8D5L8hIbBpKh0RY5Uh4IbQ0G00)

상속을 통한 코드 재사용을 위한 패턴

# 
