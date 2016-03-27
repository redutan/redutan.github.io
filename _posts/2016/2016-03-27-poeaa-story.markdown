---
layout: post
title: POEAA 이야기
date: '2016-03-27 14:56'
tags:
  - POEAA
  - book
---

# 도메인 로직 패턴

- 트랜잭션 스크립트(Transaction script)
- 테이블 모듈(Table module)
    - RecordSet을 이용하여 다양한 테이블을 투영할 수 있다.
- 도메인 모델(Domain model)

# 데이터 원본 계층

- 트랜잭션 스크립트
    - 행 데이터 게이트웨이
    - 테이블 데이터 게이트웨이
    - 낙관적 오프라인 잠금
- 테이블 모듈
    - 레코드집합(RecordSet)
    - 테이블 데이터 게이트웨이
- 도메인 모델
    - 활성 레코드(Active Record)
    - 데이터 매퍼

# 프레젠테이션 계층

- MVC
    - 컨트롤러
        - 페이지 컨트롤러
        - 프런트 컨트롤러
    - 뷰
        - 템플릿 뷰 : 서버페이지
        - 변환 뷰 : XSLT
        - 2단계 뷰 : Layout
- 하위계층
    - 원격파사드(Service?) + 데이터 전송 객체(DTO)


> Unknown

_1부 4장 동시성_

고립성(isolation)쪽 레벨이 정확하게 이해가 안됨 : [https://support.microsoft.com/ko-kr/kb/601430](https://support.microsoft.com/ko-kr/kb/601430)
로 이해가 됨
