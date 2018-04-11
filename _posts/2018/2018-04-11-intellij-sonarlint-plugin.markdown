---
layout: "post"
title: "IntelliJ SonarLint Plugin"
date: "2018-04-11 15:33"
tags:
    - intellij
---

# 동기

## 시작은 popit 아티클

어느날 popit에 sonarqube를 이용한 코드 자동리뷰 아티클이 올라왔습니다.

* [http://www.popit.kr/내코드를-자동으로-리뷰해준다면-by-sonarqube/](http://www.popit.kr/내코드를-자동으로-리뷰해준다면-by-sonarqube/)

조직장(`요다`)이 이것을 팀 내에 적용해보자고 해서 (이미 저희 팀은 sonarqube로 정적분석을 하고 있습니다.) 적용을 시작합니다.

생각보다 적용이 쉽지 않으며, 설정 포인트도 많습니다. 물론 한 번 힘들게 설정하면 그 이후에는 자동화되기 때문에 tradeoff 할 만 하지요. 적용할 프로젝트 수 자체가 많으면 초기 설정이 조금 힘들 순 있습니다. - 하지만 단순 반복 작업이기 때문에 *복사하기-붙여넣기*를 잘하면 되긴 합니다. -

**하지만** 코드를 작성하고, 커밋하고, 푸시하고, PullRequest를 올려야지 정적분석 피드백을 받을 수 있습니다. **즉, 코드 작성 시와 정적분석 피드백 시 사이의 시간 차가 너무 큽니다.**

그래서 **코드를 작성과 가능한 가까운 시간에 바로 피드백**을 받는 것이 더 좋지 않을까 하는 생각이 들었습니다.

## 대안 : SonarLint

> 그러던 중 팀원 `콤틴`이 IDEA plugin인 `SonarLint`를 사용하는 것을 추천하였습니다.

팀 내에서 IntelliJ IDEA를 사용하고 있었고, 확실히 합리적인 선택이 될 것 같았습니다.

# SonarLint 적용

## Install

* *preference > plugins*
* `SonarLint` 검색
* 결과창 내 `Search in repositories` 링크 선택 or 하딘 `Browse repositories...` 버튼 선택

![SonarLint 검색](/images/2018/04/sonarlint-search.png)

* `Install` and `Restart IntelliJ IDEA`

> Staring과 다운로드 갯수가 많고, 최근 업데이트도 지속되고 있는 것 같아서 믿을만한 플러그인 같습니다

## Install 확인

![SonarLint after install](/images/2018/04/sonarlint-after-install.png)

## Configuration

기본적인 설정만으로도 충분히 사용할 수 있으며 **sonarqube 서버가 없어도** 사용에 문제가 없습니다.

### SonarQube 서버 연동

서버 연동을 하게 되면 더 다양한 분석룰을 팀 내 전반적으로 일관성 있게 관리할 수 있습니다.

*preference > Other Settings > SoanrLint General Settings*

![SoanrLint General Settings](/images/2018/04/sonarlint-general-setting.png)

* `Automatically trigger analysis` 체크하는 것을 추천
    * 또는 해제하고 commit 시에만 분석하는것도 좋음

위 화면을 통해서 *적절하게* 설정하면 됨

> `SoanrLint Project Settings` 를 통한 설정도 가능

![SoanrLint Project Settings](/images/2018/04/sonarlint-project-setting.png)

### Commit 시 분석 추가

커밋(`Cmd + K`) 창에서

![SoanrLint Commit Setting](/images/2018/04/sonarlint-pref-commit.png)

* `Perform SonarLint analysis` 를 선택 (기본으로 체크되어 있음)
* **개인적으로 추천** commit 할 때마다 분석이 됨

## Usage

**현재 파일 분석**

mac : `Cmd + Shift + S`
or 

![SoanrLint Analyze Current](/images/2018/04/sonarlint-currentfile.png)

**그 외 가능한 사용법**

* 전체 파일 분석 : `Analyze All Files with SonarLint`
* VCS(ex: git)상 변경된 분석 검사 : `Analyze VCS Changed Files with SonarLint`

![SoanrLint IDEA Actio](/images/2018/04/sonarlint-action.png)

*`Cmd + Shift + A`, `SonarLint`로 검색한 액션들

## Analyze Example

{% highlight java %}
public class AccountRestController {
...
    @PostMapping
    public ResponseEntity<?> create(@RequestBody AccountDto.Create create) {
        Account created = accountService.create(create);
        return ResponseEntity.created(toLocationUri(created.getId())).build();
    }
...
{% endhighlight %}

* detection point : `ResponseEntity<?>`

![SoanrLint analysis Current](/images/2018/04/sonarlint-analysis-current.png)
