---
layout: post
title: "DDD 06장 - 도메인 객체의 생명주기"
date: "2015-11-25 21:39"
tags: ddd
---

![도메인객체_라이프사이클](/images/2015/11/ddd-lifecycle.png)

*그림 1 | 도메인 객체 생명주기*

**도메인 객체 관리의 문제**

1. 생명주기 동안의 무결성 유지하기
2. 생명주기 관리의 복잡성으로 모델이 난해해지는 것을 방지하기

#### 해결을 위한 3가지 패턴

**1. Aggregate(집합체)** : Domain 캡슐화. 접근 Root

**2. Factory** : Domain 객체(또는 Aggregate) 생성 캡슐화

**3. Repository** : 인프라스트럭쳐 캡슐화 및 객체지향적 접근 제공

<br/>

## Aggregate(집합체)
