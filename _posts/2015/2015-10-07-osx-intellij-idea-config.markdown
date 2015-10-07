---
layout: post
title: Intellij-IDEA 14 환경설정 - OS X
date: '2015-10-07 14:22'
tag:
  - intellij
  - osx
  - configuration
---

## 사전작업

1. Homebrew 설치 : http://brew.sh/
 - os x용 패키지설치관리자
 - Homebrew cask extension도 추가로 설치 : [OS X에 Homebrew와 Cask Extension 설치][fed7853c]
2. JDK 설치. 가능하면 현재 최신 버전으로 설치한다.
 - Homebrew를 이용한 설치 : [OS X에 Homebrew로 JDK설치][99e9fe27]
 - Web에서 직접 다운로드 : http://www.oracle.com/technetwork/java/javase/downloads/index.html
3. JAVA_HOME 환경변수 설정
 - `~/.bash_profile` 파일에 아래와 같이 추가해서 환경변수 설정을 한다.

```bash
$ vi ~/.bash_profile
# vi
export JAVA_HOME = /System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home
# :wq (저장)
```

> 환경변수 적용을 위해서 사용자계정 로그오프 후 재접속 해야한다.

**IntelliJ-IDEA 설치**

 - Homebrew를 이용한 설치 : [OS X에 Homebrew로 IntelliJ 설치][670fa358]
 - web에서 직접 다운로드 : https://www.jetbrains.com/idea/download/

## IntelliJ 환경설정

#### Editor > General > Appearance

- `Show line nubmers` 활성화
- `Show whitespaces` 활성화

![Editor > General > Appearance](/images/2015/10/01_editorAppearance.png)

#### Font

- `Save As...` 클릭 후 새로운 Scheme를 저장
- Primary 폰트 설정
- Secondary 폰트 설정 (Primary 폰트에 없는 문자-예를들면 한글-에 대한 대처폰트)

![Font](/images/2015/10/02-font.png)

#### Auto Import

- `Optimize imports on the fly` 활성화
- `Add unambiguous imports on the fly` 활성화

![Auto Import](/images/2015/10/03_autoImport.png)

#### File Encoding

- `Transparent native-to-ascii conversion` 활성화

![File Encoding](/images/2015/10/04_fileEncoding.png)

#### Time Tracking

- `Enable Time Tracking` 활성화

![Time Tracking](/images/2015/10/05_timeTracking.png)


  [fed7853c]: /2015/09/25/osx-program-for-dev/#homebrew "OX X에 Homebrew와 Cask Extension 설치"
  [670fa358]: /2015/09/25/osx-program-for-dev/#sourcetree "OS X에 Homebrew로 IntelliJ 설치"
  [99e9fe27]: /2015/09/25/osx-program-for-dev/#dev "OS X에 Homebrew로 JDK설치"
