---
layout: post
title: "OS X - Java 개발자를 위한 프로그램 설치"
date: "2015-09-25 13:16"
tags: ["osx", "install"]
---

회사에서 새로운 맥북을 지급 받은 것을 바탕으로 포스팅을 함.

순전히 본인의 취향이 반영된 것임을 미리 알려드림.

# App Store

### Xcode (필수아님)
- App Store : https://itunes.apple.com/kr/app/xcode/id497799835?mt=12

### Xcode Command Line Tools
```bash
$ xcode-select --install
```

### Alfred
- Spotlight 대처
- App Store : https://itunes.apple.com/kr/app/alfred/id405843582?mt=12

### Evernote
- App Store : https://itunes.apple.com/kr/app/evernote/id406056744?mt=12

### PDF Reader X
- App Store : https://itunes.apple.com/kr/app/pdf-reader-x/id684812309?mt=12

### Microsoft Remote Desktop
- App Store : https://itunes.apple.com/kr/app/microsoft-remote-desktop/id715768417?mt=12

# Homebrew
- OS X 용 패키지 관리자
- http://brew.sh/index_ko.html

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Homebrew extension Cask
- Homebrew의 확장 저장소
- 각종 retail 어플리케이션 지원

```bash
brew cask install google-chrome
```

### Chrome
- 웹브라우저

```bash
brew cask install google-chrome
```

### iTerm
- OS X 용 터미널 프로그램

```bash
brew cask install iterm2
```

### Docker
- 가상환경 컨테이너
- virtualbox + docer2boot + docker 통합 설치

```bash
brew install cask dockertoolbox
```

# Dev

### Java SDK
```bash
brew cask install java
```

### Oracle sqldeveloper
- 오라클 클라이언트 툴
- http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html

### MySQL Workbench
- MySQL 클라이언트 툴

```bash
brew cask install mysqlworkbench
```

### SourceTree

```bash
brew cask install sourcetree
```

### IntelliJ

```bash
brew cask install intellij-idea
```

### STS (eclipse)

```bash
brew cask install sts
```

### Atom
- Chromium 기반 에디터

```bash
brew cask install atom
```

### StarUML
- 무료 UML 툴

```bash
brew cask install staruml
```

- markdown-writer 패키지 추가 : jekyll 관리용

# Utils

### Office
- 적절한 오피스 프로그램 설치요망

##### OpenOffice : 무료
```bash
brew cask install openoffice
```

##### iWork : 신규 맥 또는 iOS 기기구입자 **무료**
- Pages : https://itunes.apple.com/kr/app/pages/id409201541?mt=12
- Numbers : https://itunes.apple.com/kr/app/numbers/id409203825?mt=12
- Keynote : https://itunes.apple.com/kr/app/keynote/id409183694?mt=12

##### Microsoft Office

### jekyll
- 정적페이지 생성 툴
- Github pages 등을 이용한 블로그 관리

```bash
sudo gem install jekyll
```

### XMind
- 무료 마인드맵 툴

```bash
brew cask install xmind
```

### Skype
- 영상통화

```bash
brew cask install skype
```
# Shell

### tree
- 폴더 구조를 트리형태로 노출

```bash
brew install tree
```

# Font

### Source Code Pro
- https://github.com/adobe-fonts/source-code-pro

### 네이버 나눔글꼴
- http://hangeul.naver.com/font

### Hack
- https://github.com/chrissimpkins/Hack#desktop-usage


> 참조 : https://gist.github.com/kevinelliott/3135044
