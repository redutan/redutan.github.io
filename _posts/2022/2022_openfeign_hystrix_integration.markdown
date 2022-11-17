spring-boot 2.4.x spring-cloud 2020.x 의존성 상황에서 feign.hystrix.enabled=true가 안됨
`feign.circuitbreaker.enabled=true` 로 바꿔보지만 openfeign과 hystrix가 잘 통합이 안 됨.

이유는 스프링 클라우드 팀에서 hystrix를 버림
실제 circuitbreak-starter 에서는 현재(2022) 기준 hystrix를 지원하지 않고 spring-cloud-netflix-hystrix도 버림

그럼 기존 spring-boot 2.0.x 등에서 업그레이드 하는 경우에는 하위 호환이 안됨

하지만 우리가 어떤 개발자임?? 스프링 설정을 강제로 먹이면 됨 ㅎㅎㅎ

1. 우선 `feign.circuitbreaker.enabled=true` 속성 자체를 지워야함. 하지만 `feign.hystrix.enabled=true` 속성은 유지 (아래 Config 참조)
2. 의존성에 아래처럼 feign-hystrix를 추가 단 `${feign-hystrix.version}` 버전은 feign-core 라이브러리와 버전을 맞추는 것이 좋음
```xml
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-hystrix</artifactId>
            <version>${feign-hystrix.version}</version>
        </dependency>
```
3. SpringCloudConfg를 하나 만들어서 적용. feign+hystrix가 잘된 버전의 `FeignClientsConfiguration`을 참고하면 됨
```java
        @Configuration
        @ConditionalOnClass({HystrixCommand.class, HystrixFeign.class})
        @ConditionalOnProperty("feign.hystrix.enabled")
        public static class HystrixFeignConfiguration {
            @Bean
            @Scope("prototype")
            public Feign.Builder feignHystrixBuilder() {
                return HystrixFeign.builder();
            }
        }
```

이제 Hystrix 통합 잘 됨.
통합이 잘되는지 안되는지 여부는 `/actuator/hystrix.stream` 에 있는 정보를 바탕으로 디버깅함
여기에서 `HystrixCommand`의 threadPool, group 이름을 보면 설정이 되는지 안되는지 알 수 있음.

설정이 제대로 안된 경우에는 group, threadPool 속성에 "HystrixCircuitBreakerFactory" 인가 하는 값이 들어가 있음.
거기에는 `@FeignClient` 애노테이션의 name(value) 속성이랑 같은 값이 설정되어 있어야함. 
