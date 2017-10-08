---
layout: post
title: 클린소프트웨어 Part 3. 급여관리 사례 연구
date: '2017-06-28 23:58'
tags:
  - books
  - clean-software
  - design-pattern
---

# 커맨드 패턴

![커맨드 패턴](/images/2017/06/command.png)

{% highlight java %}
public inteface Command {
  void do();
}
{% endhighlight %}

## 단순한 커맨드 적용

> 커맨더 패턴은 명령의 개념을 캡슐화 함으로써 의존성을 없앤다.

* 센서와 동작의 연결

![Sensor가 주도하는 Command](/images/2017/06/sensor-call-command.png)

## 트랜잭션

트랜잭션도 일종의 명령이므로 이를 커맨드 형식으로 캡슐화할 수 있다.

### 물리적, 시간적 분리

관심사 분리

## 되돌리기

![커맨드 패턴의 undo 변형](/images/2017/06/command-undo.png)

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

![템플릿 메소드 패턴 클래스 다이어그램](/images/2017/06/template-method-pattern.png)

* 상속(extends)을 통한 코드 재사용을 위한 패턴
* 상속(extends)을 통한 다형성 확보

# 스트레터지 패턴

![스트레티지 패턴 클래스 다이어그램](/images/2017/06/strategy-pattern.png)

* 인터페이스 구현(implements)을 통한 다형성 확보

**템플릿 메소드 패턴에 비해서 결합도가 낮음**

# 템플릿 메서드 패턴과 스트레터지 패턴

* 템플릿 메소드 패턴 : 일반적인 알고리즘으로 많은 구체적인 구현을 조작(상속을 통한)
* 스트레터지 패턴 : 각각의 구체적인 구현이 다른 많은 일반적인 알고리즘에 의해 조작(위임을 통한)

# 퍼사드 패턴

> 복잡하고 일반적인 인터페이스를 가진 객체 그룹에 간단하고 구체적인 인터페이스를 제공할 때 사용

![퍼사드 패턴 클래스 다이어그램](/images/2017/06/facade-pattern.png)

# 미디에이터 패턴

> 모든 클래스 간의 복잡한 상호작용을 캡슐화 하여 하나의 클래스에 위임하여 처리하는 패턴
> 각 클래스간 상호작용을 위한 관계가 필요하지 않음

![미디에이터 패턴 예시 시퀀스 다이어그램](/images/2017/06/mediator-pattern-sequence-diagram.png)

1. 리스트 상자(aListBox)는 지시자(director:aFontDialogDirector)에게 객체에게 자신이 변경되었음을 알립니다. : `widgetChanged()`
2. 지시자는 리스트 상자에서 선택된 부분이 무엇인지 알아옵니다. : `getSelection()`
3. 지시자는 입력 창(aEntryField)에 선택 부분을 전달합니다. : `setText()`
4. 입력 창에는 어떤 값이 포함됩니다. 지시자는 관련된 버튼을 활성화 합니다.

![미디에이터 패턴 예시 클래스 다이어그램](/images/2017/06/mediator-pattern.png)

## 미디에이터 패턴과 옵저버 패턴

* 유사한 듯 하면서도 다르다.
* 중재 객체와의 연관관계가 1:N 라는 개념의 차이가 가장 커 보인다.

# 퍼사트 패턴과 미디에이터 패턴

| 패턴 | 방향성 | 가시성 | 강제성 |
|-----|------|------|-------|
| 퍼사드 | 위로부터 단방향 | 가시적 | 강제적 |
| 미디에이터 | 아래로부터 양방향 | 비가시적 | 비강제적 |

# 싱글톤 패턴

# 모노스테이트 패턴

{% highlight java %}
public class Monostate {
    private static int x = 0;

    public Monostate() {

    }

    public void setX(int x) {
        this.x = x;
    }

    public int getX() {
        return x;
    }
}
{% endhighlight %}


# 싱글톤 패턴과 모노스테이트 패턴

* 둘다 안티패턴인지라 가능한 사용 안 하는 것이 좋다.
* 참고
  * [MonostatePattern](http://wiki.c2.com/?MonostatePattern)
  * [SingletonsAreEvil](http://wiki.c2.com/?SingletonsAreEvil)

# 널 오브젝트 패턴

![널 오브젝트 패턴 예시 클래스 다이어그램](/images/2017/06/null-object-pattern.png)

## 널 오브젝트 적용 전과 후

### before

{% highlight java %}
Employee e = employeeDao.getEmployee("Bob");
if (e != null && e.isTimeToPay(today))
  e.pay();
{% endhighlight %}

### after

{% highlight java %}
Employee e = employeeDao.getEmployee("Bob");
if (e.isTimeToPay(today))
  e.pay();
{% endhighlight %}

**null 비교가 없어져서 코드 가독성이 좋아지고 간결해짐**

## 널 오브젝트 상수화

* 그래도 null 비교와 같은 처리도 필요할 수 있다. 그래서 상수화 시키면 아래와 같은 처리가 가능하다.

{% highlight java %}
if (e == Employe.NULL) {
  ...
}
{% endhighlight %}
