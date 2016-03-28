---
layout: post
title: POEAA 객체-관계형 구조 패턴 (2)
date: '2016-03-28 21:28'
tags:
  - book
  - POEAA
---

# 포함 값(Embedded Value)

_한 객체를 다른 객체의 테이블에 있는 여러 필드로 매핑한다_

![포함 값](attach/2016/POEAA/ClassDiagram-EmbeddedValue.png)

객체지향에서는 여러 작은 객체를 사용하는 것이 합리적이지만, 데이터베이스 테이블은 여러 작은 객체를
저장하는 데 적합하지 않다.

_의존 매핑_ 에서 의존자가 주로 1:1 관계로 값객체가 되는 특수 사례가 **포함 값** 이다.

- 값객체 (식별자 없음)
- 1:1
- 주로 간단한 의존자

> 직렬화 LOB와 비교할 여지가 많다. 복잡하거나 1:N인 경우에는 직렬화 LOB를 고려하라.

# 직렬화 LOB(Serialized LOB)

![직렬화 LOB](attach/2016/POEAA/ClassDiagram-SerializedLob.png)

## 구현방법

- BLOB
- CLOB
    - String
    - XML

> 개인적으로 해당 패턴은 매우 특별한 경우 제외하고 다른 방식을 사용하는 것이 낫다고 본다.

# 단일 테이블 상속

# 클래스 테이블 상속

# 구현 테이블 상속

# 상속 매퍼
