---
layout: "post"
title: "Spring Cloud Consul"
date: "2018-04-13 20:04"
tags:
    - spring
    - spring-cloud
---

# ToC

1. [Consul](#consul)
2. [Consul Configuration](#consul-configuration)
3. [Consul Discovery](#consul-discovery)
4. [Summary](#summary)

# Consul

Consul은 인프라 전체에서 서비스를 발견하고 구성하는 도구입니다. 주요 기능은 아래와 같습니다.
* Service Discovery
* Health Checking
* KV Store : 서비스 구성 관리 가능 (for Spring Config)
* Multi Datacenter : SPoF 예방

[Consul](https://www.consul.io/)은 예전부터 spring-cloud 적용할 시 관심이 가던 컴포넌트 였었습니다. 
하지만 config는 `Config Server`(git or svn), discovery는 `Eureka Server` 를 주로 이용했었기 때문에 상대적으로 관심이 멀었지요.

그러던 중 팀을 옮기고 신규 프로젝트를 kickoff 함에 있어서 discovery와 config 서비스를 도입하면 어떠냐고 했다가 이에 관심을 가지는 `콤틴` 덕분에 *consul* 에도 관심이 생기기 시작했습니다.

과거 프로젝트에서는 예제도 상대적으로 부족하고 레퍼런스가 많은 인기 설정을 바탕으로 환경을 구성했는데, `콤틴`의 말을 듣고 보니, *consul*이 상당히 매력적으로 느껴졌습니다.
저도 이전에 *consul*을 리서치해 보았는데요. 상대적으로 Netflix OSS(eureka)를 선호하게 되었고, 라이선스 이슈 때문에 넘어갔었습니다.

그러나 이제 한 번 *consul*이 얼마나 가능성이 있는지 확인해보고 싶었습니다.

# 준비

## Install Consul

저는 macOs를 사용하기 때문에 패키지매니저 `brew`를 통해서 간단하게 설치하였습니다.

{% highlight bash %}
$ brew install consul
...
# 버전학인
$ consul -v
Consul v1.0.6
...
# 기동
$ brew services start consul
...
==> Successfully started `consul` (label: homebrew.mxcl.consul)
{% endhighlight %}

## Consul Web UI

[http://127.0.0.1:8500/](http://127.0.0.1:8500/)

![Consul Web UI](/images/2018/04/consul-web-ui.png)

> 다른 OS를 사용하는 경우에는 공식 홈페이지 [Getting Started](https://www.consul.io/intro/getting-started/install.html)를 참고해주세요

## Spring Boot Prject

간단합니다. 이미 *consul*은 spring-boot에 통합이 잘 되어 있어서 [SPRING INITIALIZR](https://start.spring.io/)로 쉽게 프로젝트 생성이 가능했습니다.
이번에는 config, discovery 둘 다 *consul* 기반으로 설정해 봅니다.

[SPRING INITIALIZR](https://start.spring.io/) 사이트를 통해서 가능하지만 저는 편하게 IDEA를 이용해서 생성하였습니다.

# Consul Configuration

*pom.xml 상 의존성 정보*
{% highlight xml %}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
{% endhighlight %}

* 본 예제에서는 `config`와 `discovery` 서비스를 한 애플리케이션에서 관리하는 것으로 설정했습니다.

*bootstrap.yml*
{% highlight yml %}
spring:
  application:
    name: consul-server
  cloud:
    consul:
      host: localhost # (1)
      port: 8500
      config:
        format: FILES # (2)
{% endhighlight %}

1. 서버 접속 정보를 설정합니다.
2. format 방법이 여러가지가 있는데, `FILES`가 가장 직관적이고 스프링부트 설정파일과 유사합니다.

## Spring Cloud Consul 기본 설정

*Spring Cloud Consul 기본 Key 예시*
{% highlight bash %}
config/application.properties      # (1), (2)
config/application-dev.properties  # (3)
config/consul-service.yml          # (4)
config/consul-service-dev.yml      # (5)
{% endhighlight %}

1. 기본적으로 `config` 리소스로 시작합니다.
    * 커스터마이징 가능 : `spring.cloud.consul.config.prefix`
2. `config/application.properties` : 기본적으로 모든 애플리케이션에 바인딩 될 구성
3. `config/application-dev.properties` : 모든 애플리케이션에서 `dev` 프로필이 활성화 된 경우 바인딩 될 구성
4. `config/consul-service.yml` : `consul-service` 애플리케이션에 바인딩 될 구성
5. `config/consul-service-dev.yml` : `consul-service` 애플리케이션에 `dev` 프로필이 활성화 된 경우 바인딩 될 구성
    * `,`로 프로필을 구분하지만 커스터마이징 가능 : `spring.cloud.consul.config.profileSeparator`

> `spring.cloud.consul.config.format=FILES` 구성을 통해서 `yml`, `properties` 형식을 혼용해서 사용할 수 있습니다.

만약 위 구성에서 `consul-service` 애플리케이션에 `dev` 프로필이 활성화 된 경우 아래 처럼 구성이 로딩되며 **제일 마지막 구성으로 덮어쓰게** 된다

1. `config/application.properties`
2. `config/application-dev.properties`
3. `config/consul-service.yml`
4. `config/consul-service-dev.yml`

*Consul Web UI에서 구성하기*

![Consul Web UI에서 구성하기](/images/2018/04/consul-web-ui-configuration.png)

### 구성 우선순위 예시

`config/consul-service-dev.yml`
{% highlight yml %}
message:
  hello: "Hello Consul dev"
{% endhighlight %}

*ConsulServiceAppliation.java*
{% highlight java %}
@Bean
CommandLineRunner init(@Value("${message.hello:Not found message!!}") String hello) {
    return (String... args) -> log.info(hello);
}
{% endhighlight %}

* 위 상황에서 `ConsulServiceAppliation`을 `dev` 프로필로 기동시키면 `Hello Consul dev` 로그를 확인할 수 있다.
* 기존에 먼저 바인된 구성에 중복되는 키가 있으면 무조건 덮어쓰게 되며, 없으면 기존 구성을 그대로 사용합니다.

### properties VS yml

만약 `config/application.properties`와 `config/application.yml` 동시에 있다면 어떻게 될까?

**답은 `config/application.properties`의 구성이 바인딩 된다.**

## Consul Configuration Summary

`Consul Configuration`는 `Zookeepr Configration`과 유사한 부분이 많습니다. 그리고 구성 서버로써 기본적인 기능은 모두 만족하는 것 같습니다.
하지만 `Config Server`에 비하면 부족한 점이 보이는데요.

1. **구성파일이 안전하지 못하며**(구성파일을 실수로 삭제하면 복원불가)
2. scale-out(수평확장) 시 **구성파일 동기화하기 힘듭니다.**(각 서버 간 KV Store를 동기화)

`Config Server`의 경우 형상관리(ex:git)에 의존하기 때문에 삭제해도 복원이 쉬우며, 고가용성을 위해 쉽게 서버를 scale-out 할 수 있습니다.

물론 *consul* Web UI만 제공되는 것이 아니라 cli도 제공되긴 하지만 이는 확실히 **운영 상에서 부담이 생길 수 밖에 없을 것 같습니다.**

# Consul Discovery

// TODO

# Summary

* `Consul Configuration`는 별로인 것 같습니다. `Config Server`가 여러모로(안전한 구성파일, 유연한 확장) 더 나은 것 같습니다.
* `Consul Discovery`는 상당히 추천할만 합니다.

## Reference

* *github* : 
* https://www.consul.io/
* http://cloud.spring.io/spring-cloud-static/spring-cloud-consul/2.0.0.M7/single/spring-cloud-consul.html
* https://cloud.spring.io/spring-cloud-static/spring-cloud.html
* https://www.ibm.com/support/knowledgecenter/en/SS4GSP_6.1.0/com.ibm.udeploy.admin.doc/topics/ha_md_overview.html



