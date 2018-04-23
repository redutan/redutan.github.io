---
layout: "post"
title: "상속의 위험성"
date: "2018-04-21 23:26"
tags:
    - OOP
---

# ToC

1. [동기](#동기)
2. [사전지식](#사전지식)
3. [LSP](#lsp)
4. [Practice](#practice)
5. [Summary](#summary)

# 동기

어느 날이었습니다. 팀 내에서 상속에 대한 이야기를 하다가 주니어가 그럼 상속은 어떻게 써야하는지 궁금해 했습니다.
그런데 이 상속의 위험성을 설명하려고 하니, 말로만 하기에는 부족한 것 같고, 그렇다면 worst-case 코드를 보여줘야 하는데 시간이 없었습니다.
그래서 상속의 위험성에 대한 소스코드와 그것을 설명하는 블로그 아티클을 작성해야겠다는 생각이 들었습니다.

# 사전지식

* Java 언어를 기본으로 설명합니다.

## 가변과 불변

* 객체지향은 가변(mutable)을 캡슐화(또는 관리)해서 복잡성을 제어합니다.
* 하지만 근본적으로 가변은 부수효과(side-effect)를 동반합니다.
* 그래서 가능하면 가변을 최소화 하는 것이 유리합니다.
* 이를 해결하기 위해서 불변(immutable)을 이용하는 것도 좋은 방법입니다.

## 접근제어자

* `public` : 모두 접근 가능
* `protected` : 자식 클래스와 같은 패키지 상에서 접근 가능
* `default(package)` : 같은 패키지 상에서 접근 가능
* `private` : 클래스 내에서만 접근 가능

> `public`, `protected`는 열려 있으며, `default`, `private`는 닫혀 있다. - Joshua bloch

* **접근제어는 가능한 닫혀 있는 것이 좋습니다.**
* `public` field는 캡슐화되지 않으므로 **Evil**으로 규정합니다.

## 불변식(불변조건)

클래스 불변식(Class Invariant)은 해당 클래스의 오브젝트가 가지는 제약사항을 말합니다.
즉 불변식이 깨지면 해당 객체는 *유효하지 않다고 봐야하며*, 애플리케이션 내 클래스의 계약을 위배했으므로, *문제를 발생시킵니다.*

예를 들어서 분수를 나타내는 클래스가 있다고 가정해 보겠습니다.

```java
class 분수 {
    public int 분자;
    public int 분모;

    @Override
    public String toString() {
        return 분자 + "/" + 분모;
    }
}

class 분수Test {
    @Test
    public void test분수_invalid() {
        분수 분수객체 = new 분수();
        분수객체.분자 = 1;
        분수객체.분모 = 0;  // !!! 분모는 0이 아니여야함 (불변식이 깨짐)
    }
}
```

* 내부 필드가 public이기 때문에 캡슐화를 통해서 불변식을 강제할 수가 없습니다.
* 불변식 제약사항을 강제하는 메소드를 재정의함으로써도 깨질 수도 있습니다.

# LSP

> 서브타입(sub-type)은 그것의 기반 타입(base-type)으로 치환 가능해야 한다.

그냥 단순하게 기반 타입으로 치환만 된다는 것을 의미하지는 않습니다. 기반 타입의 행위들을 서브 타입의 행위들로 대치해도 문제가 없고, 불변식도 깨지지 않아야 함을 의미합니다.
LSP를 자체를 설명하기 보다는 LSP가 위배되는 상황을 통해서 역으로 LSP를 알아보겠습니다.

유명한 Rectangle(직사각형) - Square(정사각형) 예제를 통해서 이를 확인해 보겠습니다.

```java
class Rectangle {
    private int width;
    private int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public final int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(height);
    }

    @Override
    public void setHeight(int height) {
        this.setWidth(height);
    }
}
```

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuKhEIImkLWXAJIv9p4lFILMevb9GACzCASa0qXcfcUaP9K16SMf9E4XC0ooZ2H7n0CjgG1I1nD9JInoBKXCrDBbgeSO65vOc5gKgf5QKfEQbeDj2s52GGGv0dK1t0W00)

위 정도면 충분히 LSP 를 만족한다고 보입니다. 과연 그럴까요? 
먼저 `Square` 클래스의 불변식을 알아봅시다. 정사각형이기 때문에 길이와 높이가 같은 것이 불변식입니다.
`Rectangle`는 어떤 불변식을 가질까요? 길이와 높이가 무조건 같이 변경되면 직사각형의 불변식이 위배됩니다. - 두 타입 간 *충돌이 발생하는 느낌도 있습니다.*

*`Rectangle` 불변식이 깨지는 테스트*

```java
@Test
public void testRectanbleInvariant() {
    Rectangle rectangle = new Square();
    rectangle.setWidth(10);
    rectangle.setHeight(5);

    // 과연 답이 50이 나오는가? 25가 나와서 테스트는 실패하고, LSP가 만족되지 않음을 의미한다.
    assertThat(rectangle.getArea(), is(50));
}
```

즉, `Square`는 길이와 높이를 무조건 같이 변경하게 되지만, `Rectangle`는 길이와 높이가 같이 변경되면 예상치 못한 부수효과로 인해 불변식이 깨지게 됩니다.
`Rectangle`의 불변식이 깨지게 되어서 결국 LSP도 위배하게 됩니다. - 하지만 `Square`의 불변식은 유지됩니다.

> 물론 불변식이 깨진다고 해서 무조건 LSP가 위배되는 것은 아닙니다.

> 위 상황에서 `setWidth(int)`, `setHeight(int)`를 재정의 하지 않고 `setLength(int)`와 같은 메서드를 구현하는 방법도 존재하나 그런 경우에는 width, height가 각각 변경될 수 있으므로 `Square`의 불변식(width, height는 동시에 변경되어야함)이 깨질 수 있습니다.

## Solution

먼저 상속을 유지한 상태에서 해결 방안을 알아보겠습니다.

```java
class Retangle {
    private final int width;
    private final int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public final int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    public Square(int length) {
        super(length, length);
    }
}
```

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuKhEIImkLWXAJIv9p4lFILMevb9GACzCASa0qXcfcUaP9K16K2f4LWCiemELq0JAfAUME1Qb9cfeSjL2ZGekB4qiIbL8hIX9pKj1CnaggP6JcfTUaW7Ium1K17G60000)

부수효과는 가변메서드(Setter)에서 발생합니다. 그럼 애초에 원인이 되는 가변을 모두 제거해서 위와 같이 **불변(Immutable)을 통해서 문제를 해결**할 수 있습니다.

## Another Solution

다른 방법을 알아볼까요?
애초에 이 애플리케이션 세계에서 사각형과 정사각형은 상속구조가 어울리지 않는 것 같습니다.

```java
interface Shape {
    int getArea();
}

final class Rectangle implements Shape {
    private final int width;
    private final int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

final class Square implements Shape {
    private Rectangle target;

    public Square(int length) {
        setLength();
    }

    public void setLength(int length) {
        this.target = new Rectangle(length, length);
    }

    @Override
    public int getArea() {
        return target.getArea();
    }
}
```

![](http://www.plantuml.com/plantuml/png/TKyn3i8m3Dpz2eyjWWzqAZiJVO5fZoHI4fN45GFgtudQGbcOBD-TxyvjLaaw1KykAj9TUd1dPGI_YDb0pmbIrJHJxoLdlg9NYSQ3NHWz0gBcduEd6zIMQU6CLUAYN-NLmXmtOlVh7fEaFsQbcO4sQ-JDWtYJLnxHgAqBaA6NPVbYC-qTJuTFGBEvKOiub7VV)

상속 보다는 합성(Composition) 원칙에 입각해서 위와 같이 수정하는 것도 한 방법입니다.

* *실제로 중요한 것은 넓이를 구하는 행위이지 사각형이냐, 정사각형이냐는 그 다음 문제입니다.*
* 또한 합성을 이용하면 Setter('setLength')가 있더라도 불변식이 깨지는 부수효과가 발생하지 않습니다.

# Practice

아래 다양한 예시를 통해서 더 안전한 상속을 구현하는 방법을 알아보겠습니다.

## 메서드 재정의

메서드가 재정의 불가능하게 `final`로 닫는 것이 좋습니다.

### Bad

* [BadSuperObject.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/override/BadSuperObject.java)

### Good 
* [GoodSuperObject.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/override/GoodSuperObject.java)

## Support 타입

상속을 단순 코드 재사용으로 사용하는 경우(추상 메서드가 없는 경우)에는 합성(Composition)을 사용하는 것이 좋습니다.

### Bad

* base : [ProcessSupport.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/support/ProcessSupport.java)
* sub : [BadMainProcess.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/support/BadMainProcess.java)

### Good

* helper : [ProcessHelper.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/support/ProcessHelper.java)
* client : [GoodMainProcess.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/support/GoodMainProcess.java)

## Template

템플릿 메소드 패턴의 경우 아래와 같은 코드로 정형화 하는 것이 좋습니다. - 이것은 상속을 이용한 Good Practice 중 하나입니다.

* 템플릿 패턴은 변하는 부분과 변하지 않는 부분의 관심사 분리가 중요합니다.
* 변하는 부분은 다형성을 위해 열어두고 변하지 않는 부분은 불변 템플릿(final)으로 만듭니다.

*Good Sample 중 일부 코드*
```java
public abstract class AbstractSafePrefixContentHolder implements ContentHolder {
    // 가능한 필드는 닫고 불변화 시킨다. 접근이 필요할 때만 점진적으로 연다.
    private final String content;

    public AbstractSafePrefixContentHolder(String content) {
        this.content = Objects.requireNonNull(content); // 여기에서 제약조건을 추가할 수 있다. : 선행조건으로 불변식 강제
    }

    @Override   // 템플릿 : 재정의 불가능하게 final
    public final String getContent() {
        return getPrefix() + content;
    }

    // 다형성으로써 추상 메서드만 오픈시킨다.
    abstract protected String getPrefix();
}
```

### Bad

* base : [AbstractPrefixContentHolder.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/template/AbstractPrefixContentHolder.java)
* sub : [BadContentHolder.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/template/BadContentHolder.java)

### Good

* base : [AbstractSafePrefixContentHolder.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/template/AbstractSafePrefixContentHolder.java)
* sub : [GoodContentHolder.java](https://github.com/redutan/dangers-of-inheritance/blob/master/src/main/java/io/redutan/dangers/inheritance/template/GoodContentHolder.java)

# Summary

* 불변식을 지킵니다.
* 접근제어는 가능한 닫습니다 : field는 `private`로
* 가능한 변경을 최소화 합니다 : `final`
* 불변을 이용하거나, 인터페이스를 통한 합성으로 변경해 봅니다.

## 예외

* 하지만 모든 경우에서 위 원칙을 지키는 것은 힘들수도 있습니다. 
* 상속의 위험성을 모두 파악한 상태에서 문서(javadoc)를 통해서 제약사항을 명시해서 언어로써 강제하는 것이 아니라 프로그래머가 스스로 제약사항을 지키게 하는 것도 한 방법입니다.
    * [effective-java2-chapter04#rule-17---계승을-위한-설계와-문서를-갖추거나-그럴-수-없다면-계승을-금지하라](http://redutan.github.io/2016/02/26/effective-java2-chapter04#rule-17---%EA%B3%84%EC%8A%B9%EC%9D%84-%EC%9C%84%ED%95%9C-%EC%84%A4%EA%B3%84%EC%99%80-%EB%AC%B8%EC%84%9C%EB%A5%BC-%EA%B0%96%EC%B6%94%EA%B1%B0%EB%82%98-%EA%B7%B8%EB%9F%B4-%EC%88%98-%EC%97%86%EB%8B%A4%EB%A9%B4-%EA%B3%84%EC%8A%B9%EC%9D%84-%EA%B8%88%EC%A7%80%ED%95%98%EB%9D%BC)
* 그리고 특정 도메인(ex:환경설정)에서는 위 상속의 위험성을 무시할 수도 있습니다.

# Reference

github : [https://github.com/redutan/dangers-of-inheritance/](https://github.com/redutan/dangers-of-inheritance/)

* [http://redutan.github.io/2016/02/26/effective-java2-chapter04](http://redutan.github.io/2016/02/26/effective-java2-chapter04)
* [http://redutan.github.io/2017/06/10/clean-software-part02-2](http://redutan.github.io/2017/06/10/clean-software-part02-2)
* [https://en.wikipedia.org/wiki/Class_invariant](https://en.wikipedia.org/wiki/Class_invariant)
* [https://ko.wikipedia.org/wiki/리스코프_치환_원칙](https://ko.wikipedia.org/wiki/%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84_%EC%B9%98%ED%99%98_%EC%9B%90%EC%B9%99)