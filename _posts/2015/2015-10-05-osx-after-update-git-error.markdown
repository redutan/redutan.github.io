---
layout: post
title: OS X - OS 업데이트 후 git 등 오류 발생 시 솔루션
date: '2015-10-05 23:34'
tags:
  - error
  - osx
  - git
---

`macOs High Sierra` 업데이트 시 발생 - 그냥 연례행사 인듯...
`macOS Sierra` 업데이트 시 발생

*과거 `OS X El Capitan` 업데이트도 포함됨*

_언제나 처럼 먼저 업데이트하면 골치아픈 일이 많이 생기네요 ㅠ_

그러던 중에 `Xcode Command Line Tools` 의존성 이슈가 발생하는 경우가 생겼다.
`git`을 실행시키는데 아래와 같은 오류와 함께 실행이 되지 않는 케이스이다.

{% highlight bash %}
$ git --version
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
{% endhighlight %}

`XCode`를 재설치하면 해결되나 나와 같은 전체설치가 필요하지 않는 경우에는 `Xcode Command Line Tools`만 설치할 수 있다.

{% highlight bash %}
$ xcode-select --install
{% endhighlight %}

설치 후 2343 최신버전으로 확인된다.

{% highlight bash %}
$ xcode-select -v
xcode-select version 2343.
{% endhighlight %}

> _출처 : http://stackoverflow.com/questions/32893412/command-line-tools-not-working-os-x-el-capitan_
