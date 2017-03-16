---
layout: post
title: Intellij-IDEA 14 Plugin - OS X
date: '2015-10-07 15:13'
tag:
  - intellij
  - osx
  - plugin
---

### 외부 플러그인 설치 방법

1. Preferences > Plugins 에서 찾고자할 플러그인을 검색 - _그림 1-1_
2. 중앙 하단 `Browse repositories...` 클릭 - _그림 1-1_
3. 좌측 검색된 목록에서 설치할 플러그인을 선택하고 우측 상세에서 `Install plugin` 클릭 - _그림 1-2_
4. 확인 창이 뜨면 확인을 누르고, `Restart Intellij IDEA` 클릭 - _그림 1-3_

![그림 1-1](/images/2015/10/intellijPlugins_1-1.png)
_그림 1-1_

![그림 1-2](/images/2015/10/intellijPlugins_1-2.png)
_그림 1-2_

![그림 1-3](/images/2015/10/intellijPlugins_1-3.png)
_그림 1-3_

## Lombok plugin

- `CTW` 기반 반복성 코드 생성 라이브러리

`Lombok plugin`을 플러그인 검색을 통해서 설치

#### Annotation Processors 설정

Lombok를 IDE에서 사용하기 위해서 `Annotation Processors`를 활성화
![그림 1-4](/images/2015/10/intellijPlugins_1-4.png)

#### Lombok plugin 설정

Lombok를 IDE에서 사용하기 위해서 Lombok 환경설정에서 사용 옵션을 활성화
![그림 1-5](/images/2015/10/intellijPlugins_1-5.png)

## Grep Console

- 콘솔 출력을 가독성 있게 보여줌

`Grep Console`을 플러그인 검색을 통해서 설치

## JavaDoc

- JavaDoc 헬퍼 플러그인

`JavaDoc`을 플러그인 검색을 통해서 설치

## Checkstyle

- 코딩컨벤션제어

`Checkstyle-IDEA`을 플러그인 검색을 통해서 설치

## PMD

- 정적코드분석

`PMDPlugin`을 플러그인 검색을 통해서 설치

## Findbugs

- 정적코드분석

`Findbugs-IDEA`을 플러그인 검색을 통해서 설치

## Atlassian Connector

- Atlassian 제품(JIRA, Bamboo, Crucible, FishEye) 연동 플러그인
- 개인적으로 JIRA 연동을 위해서 설치함. 다른 종류의 ITS를 사용한다면 해당 플러그인이나 기본 Task 플러그인 사용요망

`Atlassian Connector for IntelliJ IDE`를 플러그인 검색을 통해서 설치

## Eclipse Code Formatter

- IntelliJ를 이클립스 기본 코딩 스타일로 적용
- 이클립스 사용자와 협업을 한다면 설정을 추천
- 형상관리툴에서 코드포맷(특히 import) 이슈를 많이 해결해 준다.

`Eclipse Code Formatter`를 플러그인 검색을 통해서 설치

## Markdown support

- 마크다운 미리보기 등 지원
- github를 사용하면 유용함

# Save Action

- 저장 시 Auto import, Reformat (`⌥⌘L`, `⌥⇧⌘L`) 을 자동화
- Code style 설정 따라감.

# Presentation Assistant

- IDE 하단에 단축키를 노출시켜줌 (win, mac 둘 다)
- 코딩 시연, 스터디, 강좌, 페어프로그래밍 등에 유용함

![Presentation Assistant](https://plugins.jetbrains.com/files/7345/screenshot_14337.png)

*https://plugins.jetbrains.com/plugin/7345-presentation-assistant*


*설정참조*

![Eclipse Code Formatter Configuration](/images/2015/10/intellijPlugin_eclipseCodeFormatter.png)
