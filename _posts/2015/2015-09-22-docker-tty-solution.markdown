---
layout: post
title: "Docker windows tty 오류 솔루션 - 'cannot enable tty mode on non tty input' "
date: '2015-09-22 11:23'
categories: vm docker
tags: [error, windows]
---

[Docker 윈도우즈 튜토리얼][DockerWindowsTutorial]을 윈도우즈 환경에서 따라가다 보면 막히는 부분이 하나 있습니다.

```Bash
$ docker run -it -name centos centos
cannot enable tty mode on non tty input
```

```Java
public class Main {
  private String value;
  public static final int MAX_CNT = 0;

  public void getValue() {
    return this.value;
  }
}
```

Docker내 이미지를 기동하기 위한 구문인데, 윈도우즈 환경에서는 위와 같은 오류가 발생한다.
`tty`(?) 관련해서 윈도우즈쉘이 적절하게 지원되지 않아서 인데 관련해서 공부를 해봐야하는 부분이다.

솔루션은 검색결과 여러가지 방법이 있었으나, 개인적으로 아래 방법을 추천합니다.

```bash
$ winpty docker run -it -name centos centos
[root@35fb8a39901b /]#
```

*생각보다 검색이 잘되지 않더라구요 ㅠ*

[DockerWindowsTutorial]: https://docs.docker.com/windows/started
