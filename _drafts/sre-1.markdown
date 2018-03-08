---
layout: "post"
title: "SRE Chaper 1. 소개"
date: "2018-03-08 13:05"
tags:
    book
    devops
---

## SRE(Site Reliabliity Engineering)

운영팀을 위한 소프트웨어 엔지니어

*DevOps와 SRE*

> DevOps가 SRE보다 좀 더 일반화된 개념. SRE를 독특ㄷ하게 확장된 형태의 DevOps로 분류하기도 함

## SRE 책임

1. 가용성(availability)
2. 응답시간(latency)
3. 성능(performance)
4. 효율성(efficiency)
5. 변화 관리(change management)
6. 모니터링(monitoring)
7. 위기 대응(emergency response)
8. 수용량 계획(capacity planning)

## SRE의 신조

### 지속적으로 엔지니어링에 집중

* 운영 50%
* **개발 50%**

**포스트모텀(postmortem:사후검시)**

일종의 *장애보고서*

> 비난하기 보다는 실수를 공유함으로써 엔지니어가 단점을 피하거나 숨기려 하지 않고 스스로 고쳐나라 수 있도록 유도

### 서비스의 안정성증 유지하면서 변화를 최대한 수용한다.

### 모니터링

* alerts : 즉각적으로 대응
* tickets : 즉각적인 대응이 필요하지 않은 상황. 추후 해결해도 됨
* logging : 향후 분석이나 조사

