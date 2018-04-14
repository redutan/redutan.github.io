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

```bash
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
```

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
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
```

* 본 예제에서는 `config`와 `discovery` 서비스를 한 애플리케이션에서 관리하는 것으로 설정했습니다.

*bootstrap.yml*
```yml
spring:
  application:
    name: config-client
  cloud:
    consul:
      host: localhost # (1)
      port: 8500
      config:
        format: FILES # (2)
```

1. 서버 접속 정보를 설정합니다.
2. format 방법이 여러가지가 있는데, `FILES`가 가장 직관적이고 스프링부트 설정파일과 유사합니다.

## Spring Cloud Consul 기본 설정

*Spring Cloud Consul 기본 Key 예시*
```bash
config/application.properties      # (1), (2)
config/application-dev.properties  # (3)
config/config-client.yml           # (4)
config/config-client-dev.yml       # (5)
```

1. 기본적으로 `config` 리소스로 시작합니다.
    * 커스터마이징 가능 : `spring.cloud.consul.config.prefix`
2. `config/application.properties` : 기본적으로 모든 애플리케이션에 바인딩 될 구성
3. `config/application-dev.properties` : 모든 애플리케이션에서 `dev` 프로필이 활성화 된 경우 바인딩 될 구성
4. `config/config-client.yml` : `config-client` 애플리케이션에 바인딩 될 구성
5. `config/config-client-dev.yml` : `config-client` 애플리케이션에 `dev` 프로필이 활성화 된 경우 바인딩 될 구성
    * `,`로 프로필을 구분하지만 커스터마이징 가능 : `spring.cloud.consul.config.profileSeparator`

> `spring.cloud.consul.config.format=FILES` 구성을 통해서 `yml`, `properties` 형식을 혼용해서 사용할 수 있습니다.

만약 위 구성에서 `config-client` 애플리케이션에 `dev` 프로필이 활성화 된 경우 아래 처럼 구성이 로딩되며 **제일 마지막 구성으로 덮어쓰게** 된다

1. `config/application.properties`
2. `config/application-dev.properties`
3. `config/config-client.yml`
4. `config/config-client-dev.yml`

*Consul Web UI에서 구성하기*

![Consul Web UI에서 구성하기](/images/2018/04/consul-web-ui-configuration.png)

### 구성 우선순위 예시

`config/config-client-dev.yml`
```yml
message:
  hello: "Hello ConfigClient dev"
```

*ConfigClientAppliation.java*
```java
@Bean
CommandLineRunner init(@Value("${message.hello:Not found message!!}") String hello) {
    return (String... args) -> log.info(hello);
}
```

* 위 상황에서 `ConfigClientAppliation`을 `dev` 프로필로 기동시키면 `Hello ConfigClient dev` 로그를 확인할 수 있다.
* 기존에 먼저 바인된 구성에 중복되는 키가 있으면 무조건 덮어쓰게 되며, 없으면 기존 구성을 그대로 사용합니다.

### properties VS yml

만약 `config/application.properties`와 `config/application.yml` 동시에 있다면 어떻게 될까?

**답은 `config/application.properties`의 구성이 바인딩 됩니다.**

## Consul Configuration Summary

`Consul Configuration`는 `Zookeepr Configration`과 유사한 부분이 많습니다. 그리고 구성 서버로써 기본적인 기능은 모두 만족하는 것 같습니다.
하지만 `Config Server`에 비하면 부족한 점이 보이는데요.

1. **구성파일이 안전하지 못하며**(구성파일을 실수로 삭제하면 복원불가)
2. scale-out(수평확장) 시 **구성파일 동기화하기 힘듭니다.**(각 서버 간 KV Store를 동기화)
    * 물론 유료 라이선스를 이용하면 되긴하는데, OpenSource 라이선스 버전으로는 운영과 설정이 작업이 더 발생합니다.

`Config Server`의 경우 형상관리(ex:git)에 의존하기 때문에 삭제해도 복원이 쉬우며, 저장소가 분리되어 있고 분산되기 때문에 고가용성을 위해 쉽게 서버를 scale-out 할 수 있습니다.

물론 *consul* Web UI만 제공되는 것이 아니라 cli도 제공되긴 하지만 확실히 **운영 상에서 부담이 생길 수 밖에 없을 것 같습니다.**

# Consul Discovery

## Service Discovery

*consul* 클라이언트는 api서비스 또는 mysql과 같은 플랫폼을 제공 할 수 있으며, consule discovery를 이런 클라이언트를 등록 관리합니다.
그래서 다른 애플리케이션에서 의존하는 서비스를 DNS 또는 HTTP를 사용하여 쉽게 찾을 수 있습니다. 소위 말하는 LoadBalancer를 대신할 수 있으며 동적으로 서비스 등록관리를 할 수 있습니다.
최근 클라우드 환경에서는 기본적으로 필요한 기능 중 하나입니다.

## Configuration

*pom.xml 상 의존성 정보*
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

*bootstrap.yml*
```yml
spring:
  application:
    name: edge-service  # (1)
  cloud:
    consul:
      host: localhost   # (2)
      port: 8500        # (2)
      config:
        format: FILES
```

* Consul Configuration을 사용한다고 가정하였습니다.
1. application 이름을 등록하는 것은 중요합니다.
2. *consul* 서버를 설정해서 config, discovery 기능을 사용할 수 있게 합니다.

*{consul서버}/config/application.yml*
```yml
management.endpoints.web.base-path=/actuator    # (1)
```

1. spring-boot version 2 부터는 `management.endpoints.web.base`(as-is `managment.context-path`) 속성키로 변경되었습니다. 다음 설정에서 의존하기 위해서 미리 정의합니다.

*{consul서버}/config/edge-service.yml*
```yml
spring:
  cloud:
    consul:
      discovery:
        instance-id: ${spring.application.name}-${spring.application.instance-id:${random.value}}   # (1)
        health-check-path: ${management.endpoints.web.base-path}/health                             # (2)
        health-check-interval: 5s                                                                   # (3)

server:
  port: 0   # (4)
```

1. instance-id를 랜덤 기반으로 정의합니다. : `edge-service-1914caad6143e246306bb077b16f9484`
2. actuator의 health EndPoint를 바탕으로 헬스체크할 수 있게 설정합니다.
3. 헬스체크 주기는 5초로 설정합니다. (default 10s)
4. 0으로 설정하면 random으로 포트가 바인딩됩니다.

*EdgeServiceApplicaton.java*
```java
@SpringBootApplication
@EnableDiscoveryClient  // (1)
public class EdgeServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EdgeServiceApplication.class, args);
    }
}
```

1. `@EnableDiscoveryClient`를 통해서 Service Discovery의 클라이언트로 등록시킵니다.

자 이제 서비스를 기동하면 정상적으로 Consul에 등록되는 것을 확인할 수 있습니다.

*consul에 등록된 edge-service들*

![consul-edge-service](/images/2018/04/consul-edge-service.png)

## Call Service

*Call 구성도*

![call-wiki-service](http://www.plantuml.com/plantuml/png/bSuz3W8X40NWdbDCPRHWjxSmcvXu2MD1O2IJvMV3m6wylM2qaDXOyjxtmaoSLSh5Et5sX273S8AhZe6BaunfnNI38rme66XFqlY1ia8q5kKxRsxSQAPnKHPPT6NZhVtYBxatSGkS4of_w5V3ptDgS2SBEp34EjRm0Ix6kIoY--BV-OJ15E-U)

* *Client* : 호출자입니다. 여기서는 실제 고객이기 보다는 *edge-service*를 이용하는 다른 서버로 생각하시면 편할 것 같습니다.(ex:nginx, apache)
* *edge-service* : API-GW로 생각하면 편합니다.
* *wiki-service* : 실제 위키라는 도메인을 가지는 API 서비스입니다.
* *consul-server* : Service Discorvey와 Config를 책임집니다.

### 호출 흐름

1. *Client*가 *consul-server*을 통해서 *edge-service*의 정보를 제공받습니다. (DNS, HTTP)
2. 제공 받은 *edge-service*를 통해서 API를 호출합니다.
3. *edge-service*가 *consul-server*을 통해서 `downstream`할 *wiki-service* 정보를 제공 받아서 호출합니다.(1과 유사함)
4. *Client* 원하는 wiki 목록 응답을 받습니다.

### Wiki API on edge-service

최상위 페이지들을 조회하는 api(`/wiki/pages`)를 호출한다고 가정하겠습니다.

이럴 위해서 *edge-service*에서 API를 제공해야 합니다.

*edge-service의 Controller와 Service*
```java
@RestController
@RequestMapping("/wiki")
class WikiRestController {
    private final WikiService wikiService;

    public WikiRestController(WikiService wikiService) {
        this.wikiService = wikiService;
    }

    @GetMapping("/pages")
    public List<Page> topPages() {
        return wikiService.getTopPages();
    }
}

@Service
class WikiService {
    private final RestTemplate restTemplate;

    public WikiService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public List<Page> getTopPages() {
        return restTemplate.exchange(
                "//wiki-service/pages",     // (1)
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<Page>>() {
                })
                .getBody();
    }
}
```

1. 여기에서 가장 중요한 설정입니다. `//`를 통해서 protocol은 승계받은 상태에서 `wiki-service`를 즉 **서비스아이디**로 호출합니다.

그럼 위 호출을 가능하게 하는 설정은 무엇일까요? 바로 아래를 확인하시면 됩니다.

*로드밸런싱되는 `RestTemplate` 구성*
```java
    @Bean
    @LoadBalanced   // (1)
    RestTemplate loadBalancedRestTemplate() {
        return new RestTemplate();
    }
```

1. 모든 의문점은 여기서 풀립니다. `@EnableDiscoveryClient`이 활성화 되어 있으면서 `RestTemplate` bean에 `@LoadBalanced`만 달아주면 모든 설정을 spring boot에서 자동으로 해줍니다. 개발자가 기억할 것은 *서비스 명* 뿐이죠!!

그럼 이제 *consul-server*, *edge-service*, *wiki-service* 모두 기동해 보겠습니다.

*IDEA 상 서버 구동 상태*

![applications-run-on-idea](/images/2018/04/applications-run-on-idea.png)

현재 클라이언트 구현이 힘들어서 그냥 Direct로 *edge-service* 노드 중 하나를 직접 호출해 보겠습니다.

*`http://edge-service/wiki/pages` 호출 결과*
```bash
GET http://127.0.0.1:64777/wiki/pages

HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sat, 14 Apr 2018 09:07:45 GMT

[
  {
    "pageId": 1,
    "title": "기획팀 Home",
    "content": "기획팀 대시보드 페이지"
  },
  {
    "pageId": 2,
    "title": "개발팀 Home",
    "content": "개발팀 대시보드 페이지"
  },
  {
    "pageId": 3,
    "title": "디자인팀 Home",
    "content": "디자인팀 대시보드 페이지"
  }
]

Response code: 200; Time: 21ms; Content length: 174 bytes
```

## Consul Discovery Summary

마지막으로 Consul Discovery를 정리해보겠습니다. 

* 기본적인 기능은 다른 서비스와 비교해서 전혀 문제가 없습니다. 오히려 더 나음 점도 있습니다.
    * [Consul vs. Other Software](https://www.consul.io/intro/vs/index.html) 참고
* *consul*에서 Service Discovery Server를 직접 제공하기 때문에 `Eukara Server`에 애플리케이션 사용입장에서는 운영비용이 줄어듭니다.
* 그렇다면 *consul* 자체에 대한 운영을 고민해야하는데, 고가용성을 위해서 *consul* 서버를 최소 2대 이상 구성해야할 것입니다.
* 문제는 고가용성을 확보하기 위한 중앙화된 등록정보 영속화 기능이나, 등록정보 복제가 필요한데, Open Source 라이선스 상으로는 제공되지 않습니다.
    * 이 말은 서버 한대만 구성이 가능하며 SPoF가 될 수 있습니다. (다른 방법이 있을 수 있겠지만, 결국 커스터마이징 포인트가 되는 것 입니다.)
    * 이를 해결하기 위해서는 최소 Pro 라이선스가 요구됩니다. [Enhanced read scalability](https://www.consul.io/docs/enterprise/read-scale/index.html)
    * Zookeeper 앙상블과 유사한 기능인데, Zookeeper와는 비교되는 점입니다.

### VS Zookeeper

* 개발과 운영 편의를 위한 툴 자체가 *consul*이 압도적으로 낫습니다. 도메인에만 집중할 수 있는 면에서 종합선물세트와 같은 *consul*이 더 낫습니다.
* 하지만 유로 라이선스가 요구됩니다.

> Zookeeper Discovery는 직접 프로토타이핑 한 적이 없어서 의견이 조심스럽습니다.

### VS Eureka

* 기본적인 기능과 편의성은 비슷합니다만, Advanced 기능은 *consul*이 조금 더 나은 것 같습니다.
    * 둘 다 WEB UI를 제공하며, spring-boot 상에서 Discovery Client 기능을 이용하는데, 무리가 없습니다.
* 역시나 유로 라이선스가 아닌 이상에는 Eureka가 더 낫지 않나 싶습니다.


# Summary

한마디로 *consul*은 **유료 라이선스 쓴다면 적극 추천!!!!**

* `Consul Configuration`는 별로인 것 같습니다. `Config Server`가 여러모로(안전한 구성파일, 유연한 확장) 더 나은 것 같습니다.
* `Consul Discovery`는 상당히 추천할만 합니다.
    * 하지만 협업에서 운영할려면 최소 Pro 라이선스가 요구됩니다.
    * 개인적으로는 `spring-cloud-netflix`에서 제공되는 `Eureka Server`가 기본적인 기능을 만족하면서 운영 비용을 낮출 수 있습니다.



## Reference

*example github* : https://github.com/redutan/spring-boot2-consul

* https://www.consul.io/
* http://cloud.spring.io/spring-cloud-static/spring-cloud-consul/2.0.0.M7/single/spring-cloud-consul.html
* https://cloud.spring.io/spring-cloud-static/spring-cloud.html
* https://www.ibm.com/support/knowledgecenter/en/SS4GSP_6.1.0/com.ibm.udeploy.admin.doc/topics/ha_md_overview.html

## PlantUml

```uml
@startuml
Client .> [consul-server] : Find service
Client -> [edge-service] : /wiki/pages
[edge-service] -> [wiki-service] : /pages(downstream)
[edge-service] ..> [consul-server] : config & discovery lookup
[wiki-service] ..> [consul-server] : config & discovery lookup
@enduml
```