---
layout: "post"
title: 클린소프트웨어 Part 2. 애자일 설계(2)
date: "2017-06-07 21:13"
tags:
  - books
  - clean-software
---

# 리스코프 치환 원칙(LSP)

## 리스코프 치환 원칙(LSP)

> 서브타입(subtype)은 그것의 기반 타입(base type)으로 치환 가능해야 한다.

일반적인 **상속 (의 행위)** 관한 원칙

*LSP 위반은 잠재적은 OCP 위반이다.*

## IS-A

* 고양이는 동물이다.
* Cat is an Animal

### 무제

// TODO

*영속 집합 계층구조*

하지만 여기에서 Add 메서드가 특정 타입인 경우 `PersistentObject`에서 파생된 것이 아닌 경우에는 LSP를 위배하게 된다.

### LSP를 따르는 해결책

**LSP가 깨지는 메서드는 서브 클래스로 분리하고 문제 없는 메서드는 기반 클래스로 분리한다.**

// TODO

*LSP를 따르는 해결책*
