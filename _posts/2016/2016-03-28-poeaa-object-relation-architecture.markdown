---
layout: post
title: POEAA 객체-관계형 구조 패턴 (1)
date: '2016-03-28 21:27'
tags:
  - book
  - POEAA
---

# 식별자 필드(Identity Field)

_인메모리 객체와 데이터베이스 행 간의 식별자를 유지 관리하기 위해 데이터베이스 ID 필드를 저장하는 객체_

## 고려사항

1. DB 키 선택
    2. 자연키 or 대리키
    3. 단일키 or 복합키
    4. 키의 형식 : 정수 or 문자열
2. 객체 식별자 필드
    3. 형식 일치 필드(정수키 = 정수필드)
    4. 키클래스(복합키)
3. 새로운 키 생성
    4. DB 자동 생성
    5. GUID(Globally Unique IDentifier) : 해시코드
    6. 직접 키 생성
        7. `SELECT MAX(IS_NULL(seq, 0)) + 1 FROM TABLE`
        7. 키 테이블

# 외래 키 매핑

_객체 간 연결과 테이블 간 외래 키 참조의 매핑_

![외래 키 매핑](attach/2016/POEAA/ClassDiagram-ForeignkeyMapping.png)

## 고려사항

**삭제 및 삽입**

- 데이터 변경 일관성 유지
- `all delete and insert`
- `diff` : 변경 부분만 반영

**역참조 추가**

- 양방향 객체 관계 설정

**컬렉션 차이**

- 1:N or N:1 or N:M

# 연관 테이블 매핑

**N:M**

![연관 테이블 매핑](attach/2016/POEAA/ClassDiagram-NM.png)

# 의존 매핑

_한 클래스가 자식 클래스의 데이터베이스 매핑을 수행하게 한다._

![의존 매핑](attach/2016/POEAA/ClassDiagram-DependentMapping.png)

예를들어

앨범의 트랙은 해당 트랙이 속한 앨법이 로드되거나 저장될 때마다 함께 로드되거나 저장될 수 있다.
데이터베이스의 다른 테이블에서 참조되지 않는 경우. 앨범 매퍼가 트랙에 대한 매핑까지 처리하게 되면
매핑 절차를 간소화 할 수 있다. 이 매핑을 **의존 매핑(Dependent Mapping)** 이라고 한다.

DB기준으로 의존자는 복합키(소유자 키 일부포함)이면 더 구현에 유리하다.

UML에서 소유자(앨범)와 해당 의존자(트랙) 간의 관계는 합성(Composition)으로 표시하는 것이 적합하다.

**요건**

- 의존자의 소유자는 정확히 하나여야 한다.
- 의존자의 소유자를 제외하고 다른 객체로부터의 참조가 없어야 한다.

> DDD에서 집합체(Aggregate)가 일부 연상된다.
