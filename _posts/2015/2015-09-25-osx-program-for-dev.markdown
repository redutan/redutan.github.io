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
{% highlight bash %}
$ xcode-select --install
{% endhighlight %}

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

{% highlight bash %}
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

### Homebrew extension Cask
- Homebrew의 확장 저장소
- 각종 retail 어플리케이션 지원

{% highlight bash %}
brew cask install google-chrome
{% endhighlight %}

### Chrome
- 웹브라우저

{% highlight bash %}
brew cask install google-chrome
{% endhighlight %}

### iTerm
- OS X 용 터미널 프로그램

{% highlight bash %}
brew cask install iterm2
{% endhighlight %}

### Docker
- 가상환경 컨테이너
- virtualbox + docer2boot + docker 통합 설치

{% highlight bash %}
brew install cask dockertoolbox
{% endhighlight %}

# Dev

### Java SDK
{% highlight bash %}
brew cask install java
{% endhighlight %}

### Oracle sqldeveloper
- 오라클 클라이언트 툴
- http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html

### MySQL Workbench
- MySQL 클라이언트 툴

{% highlight bash %}
brew cask install mysqlworkbench
{% endhighlight %}

### SourceTree

{% highlight bash %}
brew cask install sourcetree
{% endhighlight %}

### IntelliJ

{% highlight bash %}
brew cask install intellij-idea
{% endhighlight %}

### STS (eclipse)

{% highlight bash %}
brew cask install sts
{% endhighlight %}

### Atom
- Chromium 기반 에디터

{% highlight bash %}
brew cask install atom
{% endhighlight %}

- markdown-writer 패키지 추가 : jekyll 관리용

### StarUML
- 무료 UML 툴

{% highlight bash %}
brew cask install staruml
{% endhighlight %}

# Utils

### Office
- 적절한 오피스 프로그램 설치요망

##### OpenOffice : 무료
{% highlight bash %}
brew cask install openoffice
{% endhighlight %}

##### iWork : 신규 맥 또는 iOS 기기구입자 **무료**
- Pages : https://itunes.apple.com/kr/app/pages/id409201541?mt=12
- Numbers : https://itunes.apple.com/kr/app/numbers/id409203825?mt=12
- Keynote : https://itunes.apple.com/kr/app/keynote/id409183694?mt=12

##### Microsoft Office

### jekyll
- 정적페이지 생성 툴
- Github pages 등을 이용한 블로그 관리

{% highlight bash %}
sudo gem install jekyll
{% endhighlight %}

### XMind
- 무료 마인드맵 툴

{% highlight bash %}
brew cask install xmind
{% endhighlight %}

### Skype
- 영상통화

{% highlight bash %}
brew cask install skype
{% endhighlight %}
# Shell

### tree
- 폴더 구조를 트리형태로 노출

{% highlight bash %}
brew install tree
{% endhighlight %}

# Font

### Source Code Pro
- https://github.com/adobe-fonts/source-code-pro

### 네이버 나눔글꼴
- http://hangeul.naver.com/font

### Hack
- https://github.com/chrissimpkins/Hack#desktop-usage


> 참조 : https://gist.github.com/kevinelliott/3135044
