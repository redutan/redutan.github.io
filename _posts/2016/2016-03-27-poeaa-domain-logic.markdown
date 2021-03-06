---
layout: "post"
title: "POEAA : 도메인 논리 패턴"
date: "2016-03-27 14:57"
tag:
    - POEAA
    - book
---

# 트랜잭션 스크립트

_비즈니스 논리를 프로시저별 구성해 각 프로시저가 프레젠테이션의 단일 요청을 처리하게 한다._

네이밍의 이유는 대부분의 경우 데이터베이스 트랜젝션 마다 트랜젝션 스크립트 하나가 있기 때문

- 단순함
- 복잡해지면 좋은 설계 상태 유지가 힘듬
- 코드 중복

대부분 프로젝트 패키지 구조는 트랜잭션 스크립트에 최적화 되어 있는 거 같다 : 데이타와 행위가 패키지로 분리됨

닭 잡을 때(단순 논리)는 닭 잡는 칼(트랜젝션 스크립트)로 충분한다.

# 도메인 모델

_동작과 데이터를 모두 포함하는 도메인의 객체 모델_

도메인 모델은 각 객체가 하나의 기업과 같이 복잡하거나 주문서의 내용 한 줄과 같이 간단한, 의미  있는 하나의 대상을 나타내는 상호 연결된 객체의 연결망으로 이뤄진다.

이러한 데이터와 프로세스는 프로세스와 작업 대상 데이터를 가깝게 배치하기 위한 클러스터를 형성한다
**동시에 캡슐화까지 연관지어서 생각해 볼 수 있다.**

- 패키지
- 모듈
- 라이브러리

단순 도메인 모델 (Anemic domain model) 보다는 풍족한 도메인 모델(Rich domain model)을 사용하도록 하자.

Fat Service(트랜잭션 스크립트에 어울림)보다는 Thin Service(도메인 모델에 어울림)를 지향하자.
서비스는 결국 파사드, 또는 위임자일 뿐이다. 풍부해야할 부분은 도메인 모델이어야 한다.

EJB는 이미 망했다는 것을 증명했다.

- 이는 그저 벤더(상용 EJB 컨테이너를 통한)들의 돈벌이 수단이었을 뿐이다.
- POJO는 새로운 대안을 제시했으며, 현재의 상황은(Spring) 이를 성공적으로 증명했다.

**“패러다임의 전환"**

행위와 데이터를 분리하는 절차적 사고 방식에서
연관된 행위와 데이터를 응집도 있게 한 객체(모델)로 생각하는 객체지향적 사고 방식 전환이 필요하다.

본인은 현재 범용적으로 퍼진 패키지 구조가 이러한 사고방식을 저해하는 큰 요인 중 하나라고 본다.

_DataSource Layer에서 데이터 매퍼와 잘 어울린다._

# 테이블 모듈

_데이터베이스 테이블이나 뷰의 모든 행에 대한 비즈니스 논리르 처리하는 단일 인스턴스_
DB에 어느정도 의존성이 있다.

도메인 모델과 가장 큰 차이점은 주문이 여러개인 경우 도메인 모델은 주문 수 만큼 객체를 사용하지만, 테이블 모듈은 모든 주물을 객체 하나가 처리한다는 것이다. 즉, 식별자가 존재하지 않는다.

_DataSource Layer에서 레코드 집합과 잘 어울린다._

# 서비스 계층 (Service Layer)

| 구분 | 도메인 모델 | 트랜잭션 스크립트 |
|-----|----------|--------------|
| 구현의 변형 | 도메인 파사드(Domain facade) | 작업 스크립트(Operation script) |
| 호출에 대한 고려 | Thin service | Fat service |
| 작업 식별 | 유스케이스 모델 | 사용자 인터페이스 |
