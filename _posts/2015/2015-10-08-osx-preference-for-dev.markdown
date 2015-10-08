---
layout: post
title: OS X - 개발자를 위한 환경설정
date: '2015-10-08 11:56'
tags:
  - osx
  - configuration
---

## 시스템 환경설정

![Preference](/images/2015/10/osx-01-preference.png)

#### 트랙패드
- 탭 클릭 **활성화** : 기본 클릭은 힘듬

![Touchpad-tabClick](/images/2015/10/osx-02-tabclick.png)

#### 디스플레이
- 해상도 재조정

![Display](/images/2015/10/osx-03-display.png)

#### 키보드
- 모든 F1, F2 등의 키를 표준 기능 키로 사용 **활성화** : 개발 시 단축키 편의
- 키 반복 및 지연 설정 : 빠른 입력이 가능하게

![Keyboard-keyboard](/images/2015/10/osx-04-1-keyboard.png)

- 한/영키 단축키 변경 (Swap) : 지연 없는 빠른 한/영 전환

![Keyboard-keymap](/images/2015/10/osx-04-2-keyboard-change.png)

#### Spotlight
- 단축키 변경 : 기본 단축키가 `⌃ Space`인데, IDE 에서 자동완성 단축키로 사용되기 때문에 변경요망
- 본인은 `⇧⌘ Space`로 수정했다.

![Spotlight-search](/images/2015/10/osx-05-spotlight.png)

#### 손쉬운 사용
- 확대/축소 기능 **활성화**

![EasyUse](/images/2015/10/osx-06-easyuse.png)

#### 배터리
- 배터리 퍼센트 표기 **활성화**

![Bettery](/images/2015/10/osx-07-bettery.png)

## 쉘 환경설정

#### `PATH` 설정
- `~/bin` 폴더를 설정해서 사용자 스크립트나 바로가기를 관리한다.

```bash
vi ~/.bash_profile

# path
PATH=$PATH:~/bin;
export PATH

# :wq
```

#### `ll` alias
- OS X 에서는 `ll` 명령어가 지원되지 않아서 alias로 생성한다.

```bash
vi ~/.bash_profile

# alias
alias ll="ls -lGaf"

# :wq
```

#### Homebrew
- 만약 Homebrew를 사용한다면 Github 인증관리를 위한 키를 등록한다.
 - `brew` 실행 시 `Github API Rate limit exceeded` 와 같은 오류가 발생하므로 설정해야함.
 - 참조링크 : https://gist.github.com/christopheranderton/8644743

```bash
vi ~/.bash_profile

# alias
export HOMEBREW_GITHUB_API_TOKEN=???????????????????????????????

# :wq
```

> [Homebrew Install][a38ca92d]

#### Java
- 만약 Java를 설치한다면 `JAVA_HOME`를 설정하는 것이 좋다.
- _실제 `JAVA_HOME` 정보는 OS 버전별로 바뀔 수 있으니 유의하자._

```bash
vi ~/.bash_profile

# java
export JAVA_HOME="/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home"

# :wq
```

> [Java Install][0e59b43a]

  [a38ca92d]: /2015/09/25/osx-program-for-dev/#homebrew "Homebrew Install"
  [0e59b43a]: /2015/09/25/osx-program-for-dev/#dev "Java Install"
