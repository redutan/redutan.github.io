---
layout: post
title: "DDDQ 3장 모델 주도 설계"
date: "2015-10-10 21:13"
tags:
  - "ddd"
---

> **DDDQ(Domain Driven Design Quickly) - 도메인 주도 설계란 무엇인가?** 라는 책의 간단 요약 정리

# 모델 주도 설계

**프로젝트 진행 과정을 도메인 모델을 통해서 밀접하게 연결시킨다.**

기능, 분석, 설계, 구현등의 일련의 과정이 **도메인 모델** 을 통해서 이루어지고 상호 간 피드백이 활성화되어야 한다.
여기에 **유비쿼터스 언어** 를 적극적으로 사용한다.
또한, 코드는 모델을 밀접하게 반영한다.

도메인 모델 주도 설계를 이용하여 구현 시 OOP가 적합하다.

### 모델 주도 설계를 위한 블록

![MODEL-DRIVEN DESIGN의 언어로 구성된 내비게이션 맵](/images/2015/10/ddd-diagram.png)

*이미지출처 : http://www.infoq.com/minibooks/domain-driven-design-quickly*


### 계층형 아키텍처

![Layred Architecture](/images/2015/10/DDDQ-layeredArchitecture.png)

*이미지출처 : http://wikibook.co.kr/article/layered-architecture/*

복잡성 제거를 위해서 레이어로 분할

- 레이어 내 응집력 강화
- 하위 레이어에만 의존적
- 도메인 레이어를 두어서 도메인 자체 표현에 집중

1. 사용자 인터페이스 (Presentation) : 표현
2. 애플리케이션 레이어 (Service) : 위임, 처리
3. 도메인 레이어 (Domain) : 비즈니스로직, 상태
4. 인프라스트럭처 레이어 (Repository 등) : 영속화, 통신 등

### 엔티티(Entity)

- 연속성 : 동일한 객체의 상태가 생명주기 내에서 변화하는 것
- 유일성 : 생명주기 내 객체가 유일하게 지속될 수 있게 하는 것, 유일한 식별자 존재

*엔티티의 구현 : 식별자를 만들어 내는 작업*

DB테이블의 주식별자로 엔티티의 식별자를 만들어도 된다. - 주로 이렇게 한다.

### 값 객체(Value Object)

**정의** : 도메인의 어떠한 측면을 표현하는 데 사용되지만 식별자가 없는 객체.
주로 수치와 같은 상대적으로 단순한 값을 표현하고 불변성(수정할 수 없음)을 가짐

- 값 객체는 라이프사이클을 신경쓰지 않아도 된다. 즉 쉽게 생성되고 폐기할 수 있다.
- 불변 객체
  - 만약 변경이 필요하다면 새로 만들거나 복사해서 변경한 후 반환한다 - _방어 복사_
  - 참조 : http://download.oracle.com/global/kr/magazine/23winter_tech2.pdf

**엔티티와 대조**

- 불변 != 상태가 변함 = 연속성
- 식별자가 필요없음 != 식별자가 필요함 = 유일성

값 객체 자체로는 의미가 약하다. Entity 속성으로 많이 쓰이고 엔티티와 함께 존재할 때 완전한 의미를 가진다.

### 서비스(Service)

- 인터페이스 행위(operation)와 위임
- 도메인 객체의 행위로 포함하기 힘든 경우 서비스 행위로 정의 - 즉 도메인 객체의 역할로 할당하기 힘든 경우
- 역으로 도메인 객체에 속하는 역할을 서비스 행위로 만들면 안된다 - 서비스 역할이 도메인 영역을 심하게 침범하면 계층 간 강결합이 일어난다.
- 어떤 행위(계좌이체)에 대한 도메인 객체 간(송금계좌와 입금계좌) 느슨한 결합

**특징**

1. 서비스에 의해 수행되는 오퍼레이션은 일반적으로 엔티티 또는 값 객체에 속할 수 없는 도메인의 개념을 나타낸다.
2. 수행되는 오퍼레이션은 도메인의 다른 객체를 참조한다.
3. 오퍼레이션은 상태를 저장하지 않는다. (stateless)

### 모듈(Module)

**도메인 모델의 구조화된 일부분**

**모듈화** : 관련된 개념과 작업을 조직화하여 복잡도를 감소시키는 기법

- 모듈 기반 높은 응집도 > 모듈 간 결합도는 낮아짐
  - 통신 응집도 : 같은 데이타
  - 기능 응집도 : 유사한 행위

모듈에 유비쿼터스 언어로 된 이름을 부여하라

### 집합(Aggregate)

**관계의 단순화**

1. 모델의 핵심 사항이 아닌 관계가 있다면 그 관계를 제거한다.
2. 다수성(N)의 숫자는 제약사항(constraint)-또는 한정자(qualifier)-을 추가하여 감소시킨다
3. 많은 경우 양방향 관계는 단방향 관계로 대처될 수 있다.

관계의 단순화를 거친다고 할 지라도 객체 사이에는 아직도 많은 복잡한 관계가 존재한다.
여기에서 모든 객체의 관계가 일관성을 유지하는 것은 매우 어렵다.

**관계의 일관성**

- 데이터 무결성 보장
- 불변식?? 강제

**집합으로 관계의 일관성을 유지하자**

- 각 집합은 하나의 root를 지닌다.
- `root`는 엔티티
- `root`는 집합된 다른 객체들에 대한 참조를 가지고 있다.
- `root`를 통해서만 객체의 외부에서 접근가능
  - 즉, 다른 객체들은 집합에 속한 객체들을 직접 변경할 수가 없음
  - 오직 `root`를 통해서면 내부 객체들의 변경을 요청할 수 있음
  - `root`를 통한 행위는 제어가 가능하다. - 이 제어를 통해서 **관계의 일관성** 을 유지할 수 있다.
  - 만약 외부에서 내부 객체의 참조를 요청하면 복사해서 전달하면 된다. - *방어 복사*

*예제*

![Aggregate Example](/images/2015/10/DDDQ-aggregate-1.png)

- `Customer` : 집합의 `root`
- 나머지 객체들은 값 객체(Value Object)

### 팩토리(Factory)

복잡한 엔티티나 집합을 생성하는 행위를 root 엔티티를 통해서 생성하는 것은 해당 엔티티의 책임으로 부적절하다.

하나의 객체를 생성하는 것은 그 차제로 주요 오퍼레이션에 해당하지만, 복잡하게 조합된 오퍼레이션을 이미 생성된 객체가 부담하는 것은 적절하지 않다.
이러한 책임(복잡한 객체를 생성하는 것) 생성된 객체와 결합시키는 것은 이해하기 어려운 조악한 설계를 낳는다.

**복잡한 객체 생성의 절차를 캡슐화할 수 있는 새로운 개념 = 팩토리**

팩토리는 객체 생성에 필요한 지식을 캡슐화하는 데 사용되며 집합(과 같이 복잡한 객체)를 생성하는 데 특히 유용하다.

팩토리는 생성과 관련된 책임을 진다. 모델에는 포함이 되면서 *도메인 책임을 가지지 않는다.*

구현방법

- [팩토리 메서드 패턴][8125b582] : 집합의 `root`에서 내부 객체를 생성해서 반환할 때 사용
- [추상 팩토리 패턴][ec38ff3a]

팩토리를 사용할 때 중요한 점은 객체 생성의 책임 뿐만 아니라 **제약조건과 불변식에 관한 규칙** 도 포함된다
- 값 객체 팩토리 값 객체에 대한 불변성을 지원해야한다.
- 엔티티 팩토리는 엔티티 유일성을 위해서 식별자 생성의 책임도 가질 수 있다.

**팩토리가 대신 생성자를 이용해야 하는 경우**

- 생성 작업이 복잡하지 않다.
- 객체의 생성이 다른 객체의 생성과 연관되어 있지 않으며 모든 속성이 생성자를 통해 전달되어야 한다.
- 클라이언트가 구현에 관심이 있어서, 사용할 전략(Strategy) 패턴을 선택하려고 한다.
- 클래스가 바로 해당 타입이다. 관련된 계층 구조가 없어서 concrete 구현 목록에서 선택할 필요가 없다.

팩토리는 새로운 객체를 생성하는 책임을 가진다. *DB에 존재하는 객체 메모리에 올리는 작업(일종의 조회)과 구분되어야 한다.*

  [8125b582]: https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4 "팩토리 메서드 패턴"
  [ec38ff3a]: https://en.wikipedia.org/wiki/Abstract_factory_pattern "추상 팩토리 패턴"

### 리파지토리(Repository)

도메인과 인프라스트럭쳐(ex:DB)의 계층 분리

- DB를 통해서 객체를 조회하는 책임을 단순하게 해결할려면 도메인을 사용하는 클라이언트에서 DB를 조회하면 된다.
- 하지만 그로 인해 도메인 자체가 인프라스트럭쳐에 강결합이 이루어진다. 그리고 **도메인의 격리** 를 위배하게 되어서 도메인 모델을 위태롭게 한다.
- 그리고 엔티티와 값 객체는 단순한 데이터 컨테이너(DTO)가 되버리고 도메인 로직은 인프라스트럭쳐와 클라이언트 코드로 옮겨가게 된다.
모델은 위태로워 지고 클라이언트 코드는 파멸로 간다.
- 도메인과 인프라스트럭쳐가 강결합인 상태에서 DB를 변경한다면?

**(영속화 된) 객체 참조를 얻는 로직을 캡슐화하기 위해 리파지토리를 사용**

- 클라이언트(도메인 로직)는 객체를 가져오는 인터페이스만 필요할 뿐이다.
- 인프라스트럭쳐 또한 은닉되므로 클라이언트는 인프라스트럭쳐와 결합도가 낮아진다.
- 자연스럽게 모델은 명확해 지고 본연 도메인 로직에 집중된다.

![Repository 요청 구조](/images/2015/10/DDDQ-repository-1.png)

**명세(Specification)** 를 이용한 복잡한 검색 지원

*참조 Url : http://vandbt.tistory.com/4*

##### 팩토리와 리파지토리 비교

- 둘은 모델 중심 설계의 패턴이며, 생명주기를 관리한다.
- 팩토리는 객체의 생성 (無 -> 有)
- 리파지토리는 영속화된 객체를 조회

*리파지토리를 이용한 영속화 흐름도*

![리파지토리와 팩토리를 이용한 흐름도](/images/2015/10/DDDQ-repository-sequence1.png)

1. `Customer` 객체 생성을 팩토리에 요청한다.
2. 생성된 `Customer` 객체를 반환받는다.
3. 생성된 `Customer(C0123)` 객체의 추가(영속화)를 리파지토리에 요청한다.
4. 리파지토리 내부(구현체)에서 DB 입력을 실행한다.
